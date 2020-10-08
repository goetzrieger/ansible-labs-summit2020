+++
title = "Creating Collections"
weight = 5
+++

In this exercise you will learn how to create your own custom collection.

## Preparing the exercise environment

To keep the content of this exercise separated from the other exercises you've done, first create a directory in your lab named `exercise-05` and cd into it. This directory will be used during the
whole exercise. In your VSCode terminal, run the following commands:

```bash
[{{< param "control_prompt" >}} ~]$ mkdir exercise-05
[{{< param "control_prompt" >}} ~]$ cd exercise-05
[{{< param "control_prompt" >}} exercise-05]$
```

This was covered already in this lab, but it's important enough to state again: Ansible Collections have two default lookup paths that are searched.

- User scoped path `/home/<username>/.ansible/collections`

- System scoped path `/usr/share/ansible/collections`

{{% notice tip %}}
Users can customize the collections path by modifying the `collections_path` key in the `ansible.cfg` file or by setting the environment variable `ANSIBLE_COLLECTIONS_PATHS` with the desired search path.
{{% /notice %}}

## Inspecting the structure of a collection

Ansible Collections have a standard directory and file structure that can hold modules, plugins, roles and playbooks.

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
└── tests/
```

Here is a short description of the collection structure:

- The `plugins` folder holds plugins, modules, and module_utils that can be reused in playbooks and roles.

- The `roles` folder hosts custom roles, while all collection playbooks must be stored in the `playbooks` folder.

- The `docs` folder can be used for the collections documentation, as well as the main `README.md` file that is used to describe the collection and its content.

- The `tests` folder holds tests written for the collection.

- The `galaxy.yml` file is a YAML text file that contains all the metadata used in the Ansible Galaxy hub to index the collection.  It is also used to list collection dependencies, if there are any.

## Creating a Collection

Okay, with the introduction out of the way, let's create a custom collection and populate it with roles, playbook, plugins and modules. A scaffold for a user defined custom collection can be created manually or with the `ansible-galaxy collection init` command.

### Create the Structure

Let's get started, in your VSCode terminal create the initial structure for your new collection:

```bash
[{{< param "control_prompt" >}} exercise-05]$ ansible-galaxy collection init --init-path ansible_collections redhat.workshop_demo_collection
- Collection redhat.workshop_demo_collection was created successfully
```

The `--init-path` flag is used to define a custom path in which the skeleton will be initialized.
The collection name always follows the pattern `<namespace>.<collection>`. The above example creates
the `workshop_demo_collection` in the `redhat` namespace.

Have a look for yourself:

```bash
[{{< param "control_prompt" >}} exercise-05]$ tree
.
└── ansible_collections
    └── redhat
        └── workshop_demo_collection
            ├── docs
            ├── galaxy.yml
            ├── plugins
            │   └── README.md
            ├── README.md
            └── roles
```

You can see the namespace directory was created together with a top-level `ansible_collections` directory. The scaffold is pretty minimal, note the template README files and a template `galaxy.yml` file is created to define Galaxy metadata.

### Adding Content: Custom Modules and Plugins

Now it's time to add content to our collection scaffold. Collections can include different kinds of plugins and modules. For a complete list of types please refer to the `README.md` file in the `plugins` folder.

In this lab we are going to create a minimal **Hello World** module and install it in the `plugins/modules` directory.

First, create the `plugins/modules` directory:

```bash
[{{< param "control_prompt" >}} exercise-05]$ cd ansible_collections/redhat/workshop_demo_collection
[{{< param "control_prompt" >}} workshop_demo_collection}} ]$ mkdir plugins/modules
```

Using the VSCode editor, create the file `demo_hello.py` with the content below in the `modules` folder. The `demo_hello` module says, well, "Hello" in different languages to a user defined through a parameter. This is not a lab about writing plugins, but take the time to look at the module code and understand its behavior.

{{% notice tip %}}
When doing copy/paste from your browser into the VSCode editor you might have to use `Shift-Ctrl-V`.
{{% /notice %}}

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

An Ansible module is basically an implementation of the AnsibleModule class created and executed in a minimal function called `run_module()`. As you can see, a module has a `main()` function, like a plain Python executable. Anyway, it is not meant to be executed independently.

## Adding Content: Adding a Custom Role

Ansible Collections are about bundling content that belongs together. So in the last step of this exercise we'll create a role inside the custom collection that utilizes the new module. To make things easy it will simply write the greeting into the Message of the Day file `/etc/motd`.

Generate the new role `hello_motd` using the `ansible-galaxy init` command:

{{% notice tip %}}
The `ansible-galaxy` command can be used to create initial directory structures for collections and roles. Make sure you are in the root directory of your ansible collection before executing this command.
{{% /notice %}}

```bash
[{{< param "control_prompt" >}} workshop_demo_collection ]$ ansible-galaxy init --init-path roles hello_motd
- Role hello_motd was created successfully
```

In the next step add the following role tasks in the `roles/hello_motd/tasks/main.yml` file:

```yaml
---
# tasks file for hello_motd
- name: Generate greeting and store result
  demo_hello:
    name: "{{ friend_name }}"
  register: demo_greeting

- name: store test in /etc/motd
  copy:
    content: "{{ demo_greeting.fact }}\n"
    dest: /etc/motd
  become: yes
```

Notice the usage of the `demo_hello` module, installed in the collection, to generate the greeting string.

{{% notice tip %}}
When a collection role calls a module in the same collection namespace, the module is automatically resolved.
{{% /notice %}}

Every role should come with sensible defaults, add the following default variable to the `roles/hello_motd/defaults/main.yml` file to make it look like this:

```yaml
---
# defaults file for hello_motd
friend_name: "John Doe"
```

Because `ansible-galaxy` creates a complete structure of directories and files, it's a good idea to clean up unused ones to keep it tidy:

```bash
[{{< param "control_prompt" >}} workshop_demo_collection ]$ rm -rf roles/demo_image_builder/{handlers,vars,tests}
```

And as the final step customize the `roles/hello_motd/meta/main.yml` file to define Galaxy metadata and potential dependencies of the role. Use this sample minimal content:

```yaml
galaxy_info:
  author: Ansible Workshop Team
  description: Hello world demo

  license: GPL-2.0-or-later

  min_ansible_version: 2.9

  galaxy_tags: ["demo"]

dependencies: []
```

## Build and Install your Custom Collection

Okay, you are done with creating your role. Now you'll build the collection and generate a .tar.gz file that can be installed locally or uploaded to Galaxy. From the collection folder run the following command:

```bash
[{{< param "control_prompt" >}} workshop_demo_collection ]$ ansible-galaxy collection build
Created collection for redhat.workshop_demo_collection at /home/student{{< param "student" >}}/exercise-05/ansible_collections/redhat/workshop_demo_collection/redhat-workshop_demo_collection-1.0.0.tar.gz
```

The above command will create the file `redhat-workshop_demo_collection-1.0.0.tar.gz`. Notice the semantic x.y.z versioning. Once created the file can be installed in the `COLLECTIONS_PATH` to be tested locally:

```bash
[{{< param "control_prompt" >}} workshop_demo_collection ]$ ansible-galaxy collection install redhat-workshop_demo_collection-1.0.0.tar.gz
Process install dependency map
Starting collection install process
Installing 'redhat.workshop_demo_collection:1.0.0' to '/home/student{{< param "student" >}}/.ansible/collections/ansible_collections/redhat/workshop_demo_collection'
```

By default the collection will be installed in the `~/.ansible/collections/ansible_collections` folder. Now the collection can be used locally!

## Testing your Collection

Create the `exercise-05/collections_test` folder to hold the local test:

```bash
[{{< param "control_prompt" >}} workshop_demo_collection ]$ cd ~/exercise-05
[{{< param "control_prompt" >}} exercise-05 ]$ mkdir collections_test
[{{< param "control_prompt" >}} exercise-05 ]$ cd collections_test
```

To test the collection you need a basic `playbook.yml` file, create it with the following content:

```yaml
---
- hosts: localhost

  tasks:
  - import_role:
      name: redhat.workshop_demo_collection.hello_motd
    vars:
      friend_name: "Angry Potato"
```

### Running the Test Playbook

Run the test playbook.

```bash
[{{< param "control_prompt" >}} collections_test ]$ ansible-playbook playbook.yml

PLAY [localhost] ******************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************
ok: [localhost]

TASK [redhat.workshop_demo_collection.hello_motd : Generate greeting and store result] ********************************
ok: [localhost]

TASK [redhat.workshop_demo_collection.hello_motd : store test in /etc/motd] *******************************************
changed: [localhost]

PLAY RECAP ************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Verify the result by viewing the content of `/etc/motd`:

```bash
[{{< param "control_prompt" >}} collections_test ]$ cat /etc/motd
Hello Angry Potato!
```

{{% notice note %}}
The Module is creating the text in a random language, so your greeting might differ from the example above.
{{% /notice %}}

## Takeaways

- Collections can be created using the `ansible-galaxy collection init` command. Users can develop collections contents accordingly to their needs and business logic.

- Collections plugins can be either any kind of Ansible plugins or modules. Modules are developed inside collection to create an autonomous lifecycle from the main Ansible upstream.

- Collection roles can use local collections plugins and modules.
