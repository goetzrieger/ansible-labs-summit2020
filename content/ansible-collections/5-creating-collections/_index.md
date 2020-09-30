+++
title = "Creating Collections"
weight = 5
+++

This exercise will help users understand how collections are installed, created and customized.
Covered topics:

- Demonstrate the steps involved in the installation of an Ansible Collection from the command line using the `ansible-galaxy` utility.

- Demonstrate the steps involved in the creation of a new collection with the `ansible-galaxy` utility.

- Demonstrate the creation of a custom role inside the newly created collection.

- Demonstrate the creation of a new custom plugin (basic Ansible module) in the newly created collection.

## Step 1: Preparing the exercise environment

Ansible Collections can be searched and installed from Ansible Galaxy and Red Hat Automation Hub.
Once installed, a collection can be used locally and its plugins, modules, and roles can be imported
and executed in complex Ansible-based projects.

### Preparing the exercise environment

Create a directory in your lab named `dir_name` and cd into it. This directory will be used during the
whole exercise.

```bash
mkdir exercise-05
cd exercise-05
```

Collection have two default lookup paths that are searched:

- User scoped path `/home/<username>/.ansible/collections`

- System scoped path `/usr/share/ansible/collections`

{{% notice tip %}}
Users can customized the collections path by modifying the `collections_path` key in the `ansible.cfg` file or by setting the environment variable `ANSIBLE_COLLECTIONS_PATHS` with the desired search path.
{{% /notice %}}

### Inspecting the contents of the collection

Collections have a standard structure that can can hold modules, plugins, roles and playbooks.

```bash
collection/
├── docs/
├── galaxy.yml
├── plugins/
│   ├── modules/
│   │   └── module1.py
│   ├── inventory/
│   └── .../
├── README.md
├── roles/
│   ├── role1/
│   ├── role2/
│   └── .../
├── playbooks/
│   ├── files/
│   ├── vars/
│   ├── templates/
│   └── tasks/
└── tests/
```

A short description of the collection structure:

- The `plugins` folder holds plugins, modules, and module_utils that can be reused in playbooks and roles.

- The `roles` folder hosts custom roles, while all collection playbooks must be stored in the `playbooks` folder.

- The `docs` folder can be used for the collections documentation, as well as the main `README.md` file that is used to describe the collection and its content.

- The `tests` folder holds tests written for the collection.

- The `galaxy.yml` file is a YAML text file that contains all the metadata used in the Ansible Galaxy hub to index the collection.  It is also used to list collection dependencies, if there are any.

When a collection is downloaded with the `ansible-galaxy collection install` two more files are installed:

- `MANIFEST.json`, holding additional Galaxy metadata in JSON format.

- `FILES.json`, a JSON object containing all the files SHA256 checksum.

## Step 2: Creating collections from the command line

Users can create their own collections and populate them with roles, playbook, plugins and modules.
User defined collections skeleton can be created manually or it can be authored with the
`ansible-galaxy collection init` command. This will create a standard skeleton that can be
customized lately.

Create the following collection:

```bash
ansible-galaxy collection init --init-path ansible_collections redhat.workshop_demo_collection
```

The `--init-path` flag is used to define a custom path in which the skeleton will be initialized.
The collection name always follows the pattern `<namespace.collection>`. The above example creates
the `workshop_demo_collection` in the `redhat` namespace.

The command created the following skeleton:

```bash
$ tree ansible_collections/redhat/workshop_demo_collection/
ansible_collections/redhat/workshop_demo_collection/
├── docs
├── galaxy.yml
├── plugins
│   └── README.md
├── README.md
└── roles
```

The skeleton is really minimal. Besides the template README files, a template `galaxy.yml` file is created to define Galaxy metadata.

## Step 3: Adding custom modules and plugins to the collection

Collections can be customized with different kinds of plugins and modules. For a complete list please refer to the `README.md` file in the `plugins` folder.

In this workshop we are going to create a minimal *Hello World* module and install it in the `plugins/modules` directory.

First, create the `plugins/modules` directory:

```bash
cd ansible_collections/redhat/workshop_demo_collection
mkdir plugins/modules
```

Create the `demo_hello.py` module in the the new folder. The `demo_hello` module says Hello in different languages to custom defined users. Take your time to look at the module code and understand its behavior.

```bash
#!/usr/bin/python

ANSIBLE_METADATA = {
    'metadata_version': '1.0',
    'status': ['preview'],
    'supported_by': 'community'
}

DOCUMENTATION = '''
---
module: demo_hello
short_description: A module that says hello in many languages
version_added: "2.8"
description:
  - "A module that says hello in many languages."
options:
    name:
        description:
          - Name of the person to salute. If no value is provided the default
            value will be used.
        required: false
        type: str
        default: John Doe
author:
    - Gianni Salinetti (@giannisalinetti)
'''

EXAMPLES = '''
# Pass in a custom name
- name: Say hello to Linus Torvalds
  demo_hello:
    name: "Linus Torvalds"
'''

RETURN = '''
fact:
  description: Hello string
  type: str
  sample: Hello John Doe!
'''

import random
from ansible.module_utils.basic import AnsibleModule


FACTS = [
    "Hello {name}!",
    "Bonjour {name}!",
    "Hola {name}!",
    "Ciao {name}!",
    "Hallo {name}!",
    "Hei {name}!",
]


def run_module():
    module_args = dict(
        name=dict(type='str', default='John Doe'),
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    result = dict(
        changed=False,
        fact=''
    )

    result['fact'] = random.choice(FACTS).format(
        name=module.params['name']
    )

    if module.check_mode:
        return result

    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```

An Ansible module is basically an implementation of the AnsibleModule class created and executed in a minimal function called `run_module()`. As you can see, a module has a `main()` function, like a plain Python executable. Anyway, it is
not meant to be executed independently.

## Step 4: Adding custom roles to the collection

The last step of this exercise will be focused on a role creation inside the custom collection. We will simply write the greeting into the Message of the Day file `/etc/motd`.

Generate the new role skeleton using the `ansible-galaxy init` command:

{{% notice tip %}}
Make sure you are in the root directory of your ansible collection before executing this command.
{{% /notice %}}

```bash
$ pwd
exercise-05/ansible_collections/redhat/workshop_demo_collection
$ ansible-galaxy init --init-path roles hello_motd
```

Create the following tasks in the `roles/hello_motd/tasks/main.yml` file:

```yaml
---
# tasks file for hello_motd
- name: Generate greeting and store result
  demo_hello:
    name: "{{ friend_name }}"
  register: demo_greeting

- name: store test in /etc/motd
  lineinfile:
    path: /etc/motd
    line: "{{ demo_greeting.fact }}"
  become: yes

```

Notice the usage of the `demo_hello` module, installed in the collection, to generate the greeting string.

{{% notice tip %}}
When a collection role calls a module in the same collection namespace, the module is automatically resolved.
{{% /notice %}}

Create the following variables in the `roles/hello_motd/defaults/main.yml`:

```yaml
---
# defaults file for hello_motd
friend_name: "John Doe"
```

The skeleton generates a complete structure files and folder. We can clean up the unused ones:

```bash
rm -rf roles/demo_image_builder/{handlers,vars,tests}
```

Customize the `roles/hello_motd/meta/main.yml` file to define Galaxy metadata and potential dependencies of the role. Use this sample minimal content:

```yaml
galaxy_info:
  author: Ansible Workshop Team
  description: Hello world demo

  license: GPL-2.0-or-later

  min_ansible_version: 2.9

  galaxy_tags: ["demo"]

dependencies: []
```

## Step 5: Building and installing collections

Once completed the creation task we can build the collection and generate a .tar.gz file that can be installed locally or uploaded to Galaxy.

From the collection folder run the following command:

```bash
ansible-galaxy collection build
```

The above command will create the file `redhat-workshop_demo_collection-1.0.0.tar.gz`. Notice the semantic x.y.z versioning.

Once created the file can be installed in the `COLLECTIONS_PATH` to be tested locally:

```bash
ansible-galaxy collection install redhat-workshop_demo_collection-1.0.0.tar.gz
```

By default the collection will be installed in the `~/.ansible/collections/ansible_collections` folder. Now the collection can be tested locally.

## Step 6: Testing collections locally

Create the `exercise-05/collections_test` folder to execute the local test:

```bash
cd ~/exercise-05
mkdir collections_test
cd collections_test
```

Create a basic `playbook.yml` file with the following contents:

```bash
cat > playbook.yml << EOF
---
- hosts: localhost

  tasks:
  - import_role:
      name: redhat.workshop_demo_collection.hello_motd
EOF
```

### Running the test playbook

Run the test playbook. Since some tasks require privilege escalation use the `-K` option to authenticate via sudo.

TODO: Replace playbook output with correct output!

```bash
$ ansible-playbook playbook.yml

PLAY [localhost] ********************************************************************************************

TASK [Gathering Facts] **************************************************************************************
ok: [localhost]

TASK [redhat.workshop_demo_collection.demo_image_builder : Ensure podman is present in the host] ************
ok: [localhost]

TASK [redhat.workshop_demo_collection.demo_image_builder : Generate greeting and store result] **************
ok: [localhost]

TASK [redhat.workshop_demo_collection.demo_image_builder : Create build directory] **************************
ok: [localhost]

TASK [redhat.workshop_demo_collection.demo_image_builder : Copy Dockerfile] *********************************
ok: [localhost]

TASK [redhat.workshop_demo_collection.demo_image_builder : Copy custom index.html] **************************
changed: [localhost]

TASK [redhat.workshop_demo_collection.demo_image_builder : Build and Push OCI image] ************************
changed: [localhost]

PLAY RECAP **************************************************************************************************
localhost                  : ok=7    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Takeaways

- Collections can be installed from Galaxy of from Red Hat Automation Hub. Default collections search paths or custom paths can be used.

- Collections can be created using the `ansible-galaxy collection init` command. Users can develop collections contents accordingly to their needs and business logic.

- Collections plugins can be either any kind of Ansible plugins or modules. Modules are often developed inside collection to create an autonomous lifecycle from the main Ansible upstream.

- Collection roles can use local collections plugins and modules.

----
**Navigation**
<br>
[Previous Exercise](../4-using-collections-from-tower/) - [Next Exercise](../6-automation-hub-and-galaxy/)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
