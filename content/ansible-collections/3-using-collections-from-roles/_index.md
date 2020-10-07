+++
title = "Collections from Roles"
weight = 3
+++

In this exercise you will learn how collections are used from within Ansible roles. As promised we'll explain the collection name resolution logic in more depth first. Then you'll see how to call a collection from an Ansible role using collection fully qualified collection name (FQCN).

## Step 2: Running collections from an Ansible role

First and to keep you home directory tidy, create an exercise folder:

```bash
[{{< param "control_prompt" >}} ~]$ mkdir exercise-03
[{{< param "control_prompt" >}} ~]$ cd exercise-03
```

For this lab, we will use the `ansible.posix` collection again, which contains a series of POSIX
oriented modules and plugins for systems management. You should have installed the collection in the first exercise, if not, just do it now:

```bash
[{{< param "control_prompt" >}} ~]$ ansible-galaxy collection install ansible.posix
Process install dependency map
Starting collection install process
Skipping 'ansible.posix' as it is already installed
```

{{% notice tip %}}
See how the command didn't fail or complain if the collection was installed already but just let's you know it was already there.
{{% /notice %}}

### Approach 1: Collections loaded as metadata

There are two approaches to use Ansible Collections in your role. The first approach is to specify the collection you want to use in your roles metadata.

First we'll create a simple role. Start with creating a new role scaffold using the `ansible-galaxy init` command (make sure you changed into your exercise folder):

```bash
[{{< param "control_prompt" >}} exercise-03]$ ansible-galaxy init --init-path roles selinux_manage_meta
- Role selinux_manage_meta was created successfully
```

Now you have to edit a couple of files. First edit the role metadata in `roles/selinux_manage_meta/meta/main.yml` and append the following lines at the end of the file, don't change anything else:

```yaml
# Collections list
collections:
  - ansible.posix
```

A role without some tasks is not too interesting, go and edit the `roles/selinux_manage_meta/tasks/main.yml` file and add the following tasks (make sure to keep the YAML start `---` in place):

```yaml
---
# tasks file for selinux_manage_meta
- name: Enable SELinux enforcing mode
  selinux:
    policy: targeted
    state: "{{ selinux_mode }}"

- name: Enable booleans
  seboolean:
    name: "{{ item }}"
    state: true
    persistent: true
  loop: "{{ sebooleans_enable }}"

- name: Disable booleans
  seboolean:
    name: "{{ item }}"
    state: false
    persistent: true
  loop: "{{ sebooleans_disable }}"
```

{{% notice tip %}}
We're using the simple module name. Ansible uses the information from the `collections` list in the metadata file to locate the collection(s) used.
{{% /notice %}}

Every well-written role should come with sensible defaults, edit the `roles/selinux_manage_meta/defaults/main.yml` to define default values for role variables:

```yaml
---
# defaults file for selinux_manage_meta
selinux_mode: enforcing
sebooleans_enable: []
sebooleans_disable: []
```

As a last step clean up the unused folders of the role:

```bash
[{{< param "control_prompt" >}} exercise-03]$ rm -rf roles/selinux_manage_meta/{tests,vars,handlers,files,templates}
```

And you are done with the role! Run `tree` to check everything looks good:

```bash
[{{< param "control_prompt" >}} exercise-03]$ tree
.
└── roles
    └── selinux_manage_meta
        ├── defaults
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        └── tasks
            └── main.yml
```

We can now test the new role with a basic playbook. Create the `playbook.yml` file in the exercise folder with the following content:

```yaml
---
- hosts: localhost
  become: true
  vars:
    sebooleans_enable:
      - httpd_can_network_connect
      - httpd_mod_auth_pam
    sebooleans_disable:
      - httpd_enable_cgi
  roles:
    - selinux_manage_meta
```

Run the playbook and check the results:

```bash
[{{< param "control_prompt" >}} exercise-03]$ ansible-playbook playbook.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ******************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************
ok: [localhost]

TASK [selinux_manage_meta : Enable SELinux enforcing mode] ************************************************************
ok: [localhost]

TASK [selinux_manage_meta : Enable booleans] **************************************************************************
changed: [localhost] => (item=httpd_can_network_connect)
changed: [localhost] => (item=httpd_mod_auth_pam)

TASK [selinux_manage_meta : Disable booleans] *************************************************************************
changed: [localhost] => (item=httpd_can_network_connect)

PLAY RECAP ************************************************************************************************************
localhost                  : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Approach 2: Collections loaded with FQCN

The second approach to use collections in roles uses the collection FQCN to call the related modules and plugins. To better demonstrate the differences, we will implement a new version of the previous role with the FQCN approach without changing the inner logic.

To make this easy we'll just copy and edit the role we created above:

```bash
[{{< param "control_prompt" >}} exercise-03]$ cp -r roles/selinux_manage_meta/ roles/selinux_manage_fqcn
```

Now change the file `roles/selinux_manage_fqcn/tasks/main.yml` so it uses the FQCN for the modules. It should look like this:

```yaml
---
# tasks file for selinux_manage_fqcn
- name: Enable SELinux enforcing mode
  ansible.posix.selinux:
    policy: targeted
    state: "{{ selinux_mode }}"

- name: Enable booleans
  ansible.posix.seboolean:
    name: "{{ item }}"
    state: true
    persistent: true
  loop: "{{ sebooleans_enable }}"

- name: Disable booleans
  ansible.posix.seboolean:
    name: "{{ item }}"
    state: false
    persistent: true
  loop: "{{ sebooleans_disable }}"
```

{{% notice tip %}}
Notice the usage of the modules FQCN in the role tasks. Ansible will directly look for the installed collection from within the role task, no matter if the `collections` keyword is defined at playbook level.
{{% /notice %}}

The remaining files can stay unchanged. To test the FQCN role modify the previously created `playbook.yml` file to use the new role:

```yaml
---
- hosts: localhost
  become: true
  vars:
    sebooleans_enable:
      - httpd_can_network_connect
      - httpd_mod_auth_pam
    sebooleans_disable:
      - httpd_enable_cgi
  roles:
    - selinux_manage_fqcn
```

Run the playbook again and check the results:

```bash
[{{< param "control_prompt" >}} exercise-03]$ ansible-playbook playbook.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ******************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************
ok: [localhost]

TASK [selinux_manage_meta : Enable SELinux enforcing mode] ************************************************************
ok: [localhost]

TASK [selinux_manage_meta : Enable booleans] **************************************************************************
changed: [localhost] => (item=httpd_can_network_connect)
ok: [localhost] => (item=httpd_mod_auth_pam)

TASK [selinux_manage_meta : Disable booleans] *************************************************************************
changed: [localhost] => (item=httpd_can_network_connect)

PLAY RECAP ************************************************************************************************************
localhost                  : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

The Playbook should run in the same way with the same output like above, where you loaded the collection using the role metadata.

## Takeaways

- Collections can be called from roles using the `collections` list defined in `meta/main.yml`.

- Collections can be called from roles using their FQCN directly from the role tasks.
