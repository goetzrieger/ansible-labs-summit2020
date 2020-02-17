- SHOULD BE REMOVED -

# Exercise 1.1 - Check the Prerequisites

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png)[日本語](README.ja.md).

## Your Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                 | Inventory name |
| ---------------------| ---------------|
| Ansible Control Host | ansible        |
| Managed Host 1       | node1          |
| Managed Host 2       | node2          |
| Managed Host 2       | node3          |

## Step 1.1 - Access the Environment

Login to your control host via SSH:

> **Warning**
>
> Replace **11.22.33.44** by the **IP** provided to you, and the **X** in student**X** by the student number provided to you.

    ssh studentX@11.22.33.44

> **Tip**
>
> Use the password provided on the workshops landing page.

Then become root:

    [student<X>@ansible ~]$ sudo -i

Most prerequisite tasks have already been done for you:

  - Ansible software is installed

  - `sudo` has been configured on the managed hosts to run commands that require root privileges.

Check Ansible has been installed correctly (your actual Ansible version might differ):

    [root@ansible ~]# ansible --version
    ansible 2.7.0
    [...]

> **Note**
>
> Ansible is keeping configuration management simple. Ansible requires no database or running daemons and can run easily on a laptop. On the managed hosts it needs no running agent.

Log out of the root account again:

    [root@ansible ~]# exit
    logout

> **Note**
>
> In all subsequent exercises you should work as the student\<X\> user on the control node if not explicitly told differently.

## Step 1.2 - Working the Labs

You might have guessed by now this lab is pretty commandline-centric…​ :-)

  - Don’t type everything manually, use copy & paste from the browser when appropriate. But stop to think and understand.

  - All labs were prepared using **Vim**, but we understand not everybody loves it. Feel free to use alternative editors. In the lab environment we provide **Midnight Commander** (just run **mc**, function keys can be reached via Esc-\<n\> or simply clicked with the mouse) or **Nano** (run **nano**). Here is a short [editor intro](../0.0-support-docs/editor_intro.md).

> **Tip**
>
> In the lab guide commands you are supposed to run are shown with or without the expected output, whatever makes more sense in the context.

## Step 1.3 - Challenge Labs

You will soon discover that many chapters in this lab guide come with a "Challenge Lab" section. These labs are meant to give you a small task to solve using what you have learned so far. The solution of the task is shown underneath a warning sign.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
# Exercise 1.2 - Running Ad-hoc commands

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

For our first exercise, we are going to run some ad-hoc commands to help you get a feel for how Ansible works.  Ansible Ad-Hoc commands enable you to perform tasks on remote nodes without having to write a playbook.  They are very useful when you simply need to do one or two things quickly and often, to many remote nodes.

## Step 2.1 - Work with your Inventory

To use the ansible command for host management, you need to provide an inventory file which defines a list of hosts to be managed from the control node. In this lab the inventory is provided by your instructor. The inventory is an ini formatted file listing your hosts, sorted in groups, additionally providing some variables. It looks like:

```bash
[all:vars]
ansible_user=student1
ansible_ssh_pass=PASSWORD
ansible_port=22

[web]
node1 ansible_host=<X.X.X.X>
node2 ansible_host=<Y.Y.Y.Y>
node3 ansible_host=<Z.Z.Z.Z>

[control]
ansible ansible_host=44.55.66.77
```

> **Tip**
>
> The environment for this lab uses SSH with password authentication to login to the managed nodes. For the sake of keeping things simple the password is put into the inventory file in clear text. In real world scenarios you would either use SSH key authentication or supply the password in a secure way, e.g. by using Ansible Vault. 

Ansible is already configured to use the inventory specific to your environment. We will show you in the next step how that is done. For now, we will execute some simple commands to work with the inventory.

To reference inventory hosts, you supply a host pattern to the ansible command. Ansible has a `--list-hosts` option which can be useful for clarifying which managed hosts are referenced by the host pattern in an ansible command.

The most basic host pattern is the name for a single managed host listed in the inventory file. This specifies that the host will be the only one in the inventory file that will be acted upon by the ansible command. Run:

```bash
[student<X@>ansible ~]$ ansible node1 --list-hosts
  hosts (1):
    node1
```

An inventory file can contain a lot more information, it can organize your hosts in groups or define variables. In our example, the current inventory has the groups `web` and `control`. Run Ansible with these host patterns and observe the output:

```bash
[student<X@>ansible ~]$ ansible web  --list-hosts
[student<X@>ansible ~]$ ansible web,ansible --list-hosts
[student<X@>ansible ~]$ ansible 'node*' --list-hosts
[student<X@>ansible ~]$ ansible all --list-hosts
```

As you see it is OK to put systems in more than one group. For instance, a server could be both a web server and a database server. Note that in Ansible the groups are not necessarily hierarchical.

> **Tip**
>
> The inventory can contain more data. E.g. if you have hosts that run on non-standard SSH ports you can put the port number after the hostname with a colon. Or you could define names specific to Ansible and have them point to the "real" IP or hostname.

## Step 2.2 - The Ansible Configuration Files

The behavior of Ansible can be customized by modifying settings in Ansible’s ini-style configuration file. Ansible will select its configuration file from one of several possible locations on the control node, please refer to the [documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html).

> **Tip**
>
> The recommended practice is to create an `ansible.cfg` file in the directory from which you run Ansible commands. This directory would also contain any files used by your Ansible project, such as the inventory and playbooks. Another recommended practice is to create a file `.ansible.cfg` in your home directory.

In the lab environment provided to you an `.ansible.cfg` file has already been created and filled with the necessary details in the home directory of your `student<X>` user on the control node:

```bash
[student<X>@ansible ~]$ ls -la .ansible.cfg
-rw-r--r--. 1 student<X> student<X> 231 14. Mai 17:17 .ansible.cfg
```

Output the content of the file:

```bash
[student<X>@ansible ~]$ cat .ansible.cfg
[defaults]
stdout_callback = yaml
connection = smart
timeout = 60
deprecation_warnings = False
host_key_checking = False
retry_files_enabled = False
inventory = /home/student<X>/lab_inventory/hosts
```

There are multiple configuration flags provided. Most of them are not of interest here, but make sure to note the last line: there the location of the inventory is provided. That is the way Ansible knew in the previous commands what machines to connect to.

Output the content of your dedicated inventory:

```bash
[student<X>@ansible ~]$ cat /home/student<X>/lab_inventory/hosts
[all:vars]
ansible_user=student<X>
ansible_ssh_pass=ansible
ansible_port=22

[web]
node1 ansible_host=11.22.33.44
node2 ansible_host=22.33.44.55
node3 ansible_host=33.44.55.66

[control]
ansible ansible_host=44.55.66.77
```

> **Tip**
>
> Note that each student has an individual lab environment. The IP addresses shown above are only an example and the IP addresses of your individual environments are different. As with the other cases, replace **\<X\>** with your actual student number.

## Step 2.3 - Ping a host

Let's start with something really basic - pinging a host. To do that we use the Ansible `ping` module. The `ping` module makes sure our target hosts are responsive. Basically, it connects to the managed host, executes a small script there and collects the results. This ensures that the managed host is reachable and that Ansible is able to execute commands properly on it.

> **Tip**
>
> Think of a module as a tool which is designed to accomplish a specific task.

Ansible needs to know that it should use the `ping` module: The `-m` option defines which Ansible module to use. Options can be passed to the specified modul using the `-a` option. In addition to the module Ansible needs to know what hosts it should run the task on, here you supply the group `web`.

```bash
[student<X>@ansible ~]$ ansible web -m ping
node2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
[...]
```

As you see each node in the `web` group reports the successful execution and the actual result - here "pong".

## Step 2.4 - Listing Modules and Getting Help

Ansible comes with a lot of modules by default. To list all modules run:

```bash
[student<X>@ansible ~]$ ansible-doc -l
```

> **Tip**
>
> In `ansible-doc` leave by pressing the button `q`. Use the `up`/`down` arrows to scroll through the content.

To find a module try e.g.:

```bash
[student<X>@ansible ~]$ ansible-doc -l | grep -i user
```

Get help for a specific module including usage examples:

```bash
[student<X>@ansible ~]$ ansible-doc user
```

> **Tip**
>
> Mandatory options are marked by a "=" in `ansible-doc`.

## Step 2.5 - Use the command module:

Now let's see how we can run a good ol' fashioned Linux command and format the output using the `command` module. It simply executes the specified command on a managed host (note this time not a group but a hostname is used as host pattern):

```bash
[student<X>@ansible ~]$ ansible node1 -m command -a "id"
node1 | CHANGED | rc=0 >>
uid=1001(student1) gid=1001(student1) Gruppen=1001(student1) Kontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
In this case the module is called `command` and the option passed with `-a` is the actual command to run. Try to run this ad hoc command on all managed hosts using the `all` host pattern.

Another example: Have a quick look at the kernel versions your hosts are running:

```bash
[student<X>@ansible ~]$ ansible all -m command -a 'uname -r'
```

Sometimes it’s desirable to have the output for a host on one line:

```bash
[student<X>@ansible ~]$ ansible all -m command -a 'uname -r' -o
```

> **Tip**
>
> Like many Linux commands, `ansible` allows for long-form options as well as short-form.  For example `ansible web --module-name ping` is the same as running `ansible web -m ping`.  We are going to be using the short-form options throughout this workshop.

## Step 2.6 - The copy module and permissions

Using the `copy` module, execute an ad hoc command on `node1` to change the contents of the `/etc/motd` file. **The content is handed to the module through an option in this case**.

Run the following, but **expect an error**:

```bash
[student<X>@ansible ~]$ ansible node1 -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd'
```

As mentioned this produces an **error**:

```bash
    node1 | FAILED! => {
        "changed": false,
        "checksum": "a314620457effe3a1db7e02eacd2b3fe8a8badca",
        "failed": true,
        "msg": "Destination /etc not writable"
    }
```

The output of the ad hoc command is screaming **FAILED** in red at you. Why? Because user **student\<X\>** is not allowed to write the motd file.

Now this is a case for privilege escalation and the reason `sudo` has to be setup properly. We need to instruct Ansible to use `sudo` to run the command as root by using the parameter `-b` (think "become").

> **Tip**
>
> Ansible will connect to the machines using your current user name (student\<X\> in this case), just like SSH would. To override the remote user name, you could use the `-u` parameter.

For us it’s okay to connect as `student<X>` because `sudo` is set up. Change the command to use the `-b` parameter and run again:

```bash
[student<X>@ansible ~]$ ansible node1 -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -b
```

This time the command is a success:

```bash
node1 | CHANGED => {
    "changed": true,
    "checksum": "4458b979ede3c332f8f2128385df4ba305e58c27",
    "dest": "/etc/motd",
    "gid": 0,
    "group": "root",
    "md5sum": "65a4290ee5559756ad04e558b0e0c4e3",
    "mode": "0644",
    "owner": "root",
    "secontext": "system_u:object_r:etc_t:s0",
    "size": 19,
    "src": "/home/student1/.ansible/tmp/ansible-tmp-1557857641.21-120920996103312/source",
    "state": "file",
    "uid": 0
```

Use Ansible with the generic `command` module to check the content of the motd file:

```bash
[student<X>@ansible ~]$ ansible node1 -m command -a 'cat /etc/motd'
node1 | CHANGED | rc=0 >>
Managed by Ansible
```

Run the `ansible node1 -m copy …​` command from above again. Note:

  - The different output color (proper terminal config provided).
  - The change from `"changed": true,` to `"changed": false,`.
  - The first line says `SUCCESS` instead of `CHANGED`.

> **Tip**
>
> This makes it a lot easier to spot changes and what Ansible actually did.

## Challenge Lab: Modules

  - Using `ansible-doc`

      - Find a module that uses Yum to manage software packages.

      - Look up the help examples for the module to learn how to install a package in the latest version.

  - Run an Ansible ad hoc command to install the package "screen" in the latest version on `node1`.

> **Tip**
>
> Use the copy ad hoc command from above as a template and change the module and options.

> **Warning**
>
> **Solution below\!**

```bash
[student<X>@ansible ~]$ ansible-doc -l | grep -i yum
[student<X>@ansible ~]$ ansible-doc yum
[student<X>@ansible ~]$ ansible node1 -m yum -a 'name=screen state=latest' -b
```

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
# Exercise 1.3 - Writing Your First Playbook

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

While Ansible ad hoc commands are useful for simple operations, they are not suited for complex configuration management or orchestration scenarios. For such use cases *playbooks* are the way to go.

Playbooks are files which describe the desired configurations or steps to implement on managed hosts. Playbooks can change lengthy, complex administrative tasks into easily repeatable routines with predictable and successful outcomes.

A playbook is where you can take some of those ad-hoc commands you just ran and put them into a repeatable set of *plays* and *tasks*.

A playbook can have multiple plays and a play can have one or multiple tasks. In a task a *module* is called, like the modules in the previous chapter. The goal of a *play* is to map a group of hosts.  The goal of a *task* is to implement modules against those hosts.

> **Tip**
>
> Here is a nice analogy: When Ansible modules are the tools in your workshop, the inventory is the materials and the Playbooks are the instructions.

## Step 3.1 - Playbook Basics

Playbooks are text files written in YAML format and therefore need:

  - to start with three dashes (`---`)

  - proper identation using spaces and **not** tabs\!

There are some important concepts:

  - **hosts**: the managed hosts to perform the tasks on

  - **tasks**: the operations to be performed by invoking Ansible modules and passing them the necessary options.

  - **become**: privilege escalation in Playbooks, same as using `-b` in the ad hoc command.

> **Warning**
>
> The ordering of the contents within a Playbook is important, because Ansible executes plays and tasks in the order they are presented.

A Playbook should be **idempotent**, so if a Playbook is run once to put the hosts in the correct state, it should be safe to run it a second time and it should make no further changes to the hosts.

> **Tip**
>
> Most Ansible modules are idempotent, so it is relatively easy to ensure this is true.


## Step 3.2 - Creating a Directory Structure and File for your Playbook

Enough theory, it’s time to create your first Playbook. In this lab you create a Playbook to set up an Apache webserver in three steps:

  - First step: Install httpd package

  - Second step: Enable/start httpd service

  - Third step: Create an index.html file

This Playbook makes sure the package containing the Apache webserver is installed on `node1`.

There is a [best practice](http://docs.ansible.com/ansible/playbooks_best_practices.html) on the preferred directory structures for playbooks.  We strongly encourage you to read and understand these practices as you develop your Ansible ninja skills.  That said, our playbook today is very basic and creating a complex structure will just confuse things.

Instead, we are going to create a very simple directory structure for our playbook, and add just a couple of files to it.

On your control host **ansible**, create a directory called `ansible-files` in your home directory and change directories into it:

```bash
[student<X>@ansible ~]$ mkdir ansible-files
[student<X>@ansible ~]$ cd ansible-files/
```

Add a file called `apache.yml` with the following content. As discussed in the previous exercises, use `vi`/`vim` or, if you are new to editors on the command line, check out the [editor intro](../0.0-support-docs/editor_intro.md) again.

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
```

This shows one of Ansible’s strenghts: The Playbook syntax is easy to read and understand. In this Playbook:

  - A name is given for the play via `name:`.

  - The host to run the playbook against is defined via `hosts:`.

  - We enable user privilege escalation with `become:`.

> **Tip**
>
> You obviously need to use privilege escalation to install a package or run any other task that requires root permissions. This is done in the Playbook by `become: yes`.

Now that we've defined the play, let's add a task to get something done. We will add a task in which yum will ensure that the Apache package is installed in the latest version. Modify the file so that it looks like the following listing:

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
```
> **Tip**
>
> Since playbooks are written in YAML, alignment of the lines and keywords is crucial. Make sure to vertically align the *t* in `task` with the *b* in `become`. Once you are more familiar with Ansible, make sure to take some time and study a bit the [YAML Syntax](http://docs.ansible.com/ansible/YAMLSyntax.html).

In the added lines:

  - We started the tasks part with the keyword `tasks:`.

  - A task is named and the module for the task is referenced. Here it uses the `yum` module.

  - Parameters for the module are added: 
  
    - `name:` to identify the package name
    - `state:` to define the wanted state of the package

> **Tip**
>
> The module parameters are individual to each module. If in doubt, look them up again with `ansible-doc`.

Save your playbook and exit your editor.

## Step 3.3 - Running the Playbook

Playbooks are executed using the `ansible-playbook` command on the control node. Before you run a new Playbook it’s a good idea to check for syntax errors:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook --syntax-check apache.yml
```

Now you should be ready to run your Playbook:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook apache.yml
```
The output should not report any errors but provide an overview of the tasks executed and a play recap summarizing what has been done. There is also a task called "Gathering Facts" listed there: this is an built-in task that runs automatically at the beginning of each play. It collects information about the managed nodes. Exercises later on will cover this in more detail.

Use SSH to make sure Apache has been installed on `node1`. The necessary IP address is provided in the inventory. Grep for the IP address there and use it to SSH to the node.

```bash
[student<X>@ansible ansible-files]$ grep node1 ~/lab_inventory/hosts
node1 ansible_host=11.22.33.44
[student<X>@ansible ansible-files]$ ssh 11.22.33.44
student<X>@11.22.33.44's password:
Last login: Wed May 15 14:03:45 2019 from 44.55.66.77
Managed by Ansible
[student<X>@node1 ~]$ rpm -qi httpd
Name        : httpd
Version     : 2.4.6
[...]
```

Log out of `node1` with the command `exit` so that you are back on the control host, and verify the installed package with an Ansible ad hoc command\!

```bash
[student<X>@ansible ansible-files]$ ansible node1 -m command -a 'rpm -qi httpd'
```

Run the Playbook a second time, and compare the output: The output changed from "changed" to "ok", and the color changed from yellow to green. Also the "PLAY RECAP" is different now. This make it easy to spot what Ansible actually did.

## Step 3.4 - Extend your Playbook: Start & Enable Apache

The next part of the Playbook makes sure the Apache webserver is enabled and started on `node1`.

On the control host, as your student user, edit the file `~/ansible-files/apache.yml` to add a second task using the `service` module. The Playbook should now look like this:

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
```

Again: what these lines do is easy to understand:

  - a second task is created and named

  - a module is specified (`service`)

  - parameters for the module are supplied

Thus with the second task we make sure the Apache server is indeed running on the target machine. Run your extended Playbook:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook apache.yml
```

Note the output now: Some tasks are shown as "ok" in green and one is shown as "changed" in yellow.

  - Use an Ansible ad hoc command again to make sure Apache has been enabled and started, e.g. with: `systemctl status httpd`.

  - Run the Playbook a second time to get used to the change in the output.

## Step 3.5 - Extend your Playbook: Create an index.html

Check that the tasks were executed correctly and Apache is accepting connections: Make an HTTP request using Ansible’s `uri` module in an ad hoc command from the control node. Make sure to replace the **\<IP\>** with the IP for the node from the inventory.

> **Warning**
>
> **Expect a lot of red lines and a 403 status\!**

```bash
[student<X>@ansible ansible-files]$ ansible localhost -m uri -a "url=http://<IP>"
```

There are a lot of red lines and an error: As long as there is not at least an `index.html` file to be served by Apache, it will throw an ugly "HTTP Error 403: Forbidden" status and Ansible will report an error.

So why not use Ansible to deploy a simple `index.html` file? Create the file `~/ansible-files/index.html` on the control node:

```html
<body>
<h1>Apache is running fine</h1>
</body>
```

You already used Ansible’s `copy` module to write text supplied on the commandline into a file. Now you’ll use the module in your Playbook to actually copy a file:

On the control node as your student user edit the file `~/ansible-files/apache.yml` and add a new task utilizing the `copy` module. It should now look like this:

```yaml
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
  - name: copy index.html
    copy:
      src: ~/ansible-files/index.html
      dest: /var/www/html/
```

You are getting used to the Playbook syntax, so what happens? The new task uses the `copy` module and defines the source and destination options for the copy operation as parameters.

Run your extended Playbook:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook apache.yml
```

  - Have a good look at the output

  - Run the ad hoc command  using the "uri" module from further above again to test Apache: The command should now return a friendly green "status: 200" line, amongst other information.

## Step 3.6 - Practice: Apply to Multiple Host

This was nice but the real power of Ansible is to apply the same set of tasks reliably to many hosts.

  - So what about changing the apache.yml Playbook to run on `node1` **and** `node2` **and** `node3`?


As you might remember, the inventory lists all nodes as members of the group `web`:

```ini
[web]
node1 ansible_host=11.22.33.44
node2 ansible_host=22.33.44.55
node3 ansible_host=33.44.55.66
```

> **Tip**
>
> The IP addresses shown here are just examples, your nodes will have different IP addresses.

Change the Playbook to point to the group "web":

```yaml
---
- name: Apache server installed
  hosts: web
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
  - name: copy index.html
    copy:
      src: ~/ansible-files/index.html
      dest: /var/www/html/
```

Now run the Playbook:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook apache.yml
```

Finally check if Apache is now running on all servers. Identify the IP addresses of the nodes in your inventory first, and afterwards use them each in the ad hoc command with the uri module as we already did with the `node1` above. All output should be green.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
# Exercise 1.4 - Using Variables

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

Previous exercises showed you the basics of Ansible Engine.  In the next few exercises, we are going
to teach some more advanced Ansible skills that will add flexibility and power to your playbooks.

Ansible exists to make tasks simple and repeatable.  We also know that not all systems are exactly alike and often require
some slight change to the way an Ansible playbook is run.  Enter variables.

Ansible supports variables to store values that can be used in Playbooks. Variables can be defined in a variety of places and have a clear precedence. Ansible substitutes the variable with its value when a task is executed.

Variables are referenced in Playbooks by placing the variable name in double curly braces:

<!-- {% raw %} -->
```yaml
Here comes a variable {{ variable1 }}
```
<!-- {% endraw %} -->

Variables and their values can be defined in various places: the inventory, additional files, on the command line, etc.

The recommended practice to provide variables in the inventory is to define them in files located in two directories named `host_vars` and `group_vars`:

  - To define variables for a group "servers", a YAML file named `group_vars/servers` with the variable definitions is created.

  - To define variables specifically for a host `node1`, the file `host_vars/node1` with the variable definitions is created.

> **Tip**
>
> Host variables take precedence over group variables (more about precedence can be found in the [docs](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)).

## Step 4.1 - Create Variable Files

For understanding and practice let’s do a lab. Following up on the theme "Let’s build a webserver. Or two. Or even more…​", you will change the `index.html` to show the development environment (dev/prod) a server is deployed in.

On the ansible control host, as the `student` user, create the directories to hold the variable definitions in `~/ansible-files/`:

```bash
[student<X>@ansible ansible-files]$ mkdir host_vars group_vars
```

Now create two files containing variable definitions. We’ll define a variable named `stage` which will point to different environments, `dev` or `prod`:

  - Create the file `~/ansible-files/group_vars/web` with this content:

```yaml
---
stage: dev
```

  - Create the file `~/ansible-files/host_vars/node2` with this content:

```yaml
---
stage: prod
```

What is this about?

  - For all servers in the `web` group the variable `stage` with value `dev` is defined. So as default we flag them as members of the dev environment.

  - For server `node2` this is overriden and the host is flagged as a production server.

## Step 4.2 - Create index.html Files

Now create two files in `~/ansible-files/`:

One called `prod_index.html` with the following content:

```html
<body>
<h1>This is a production webserver, take care!</h1>
</body>
```

And the other called `dev_index.html` with the following content:

```html
<body>
<h1>This is a development webserver, have fun!</h1>
</body>
```

## Step 4.3 - Create the Playbook

Now you need a Playbook that copies the prod or dev `index.html` file - according to the "stage" variable.

Create a new Playbook called `deploy_index_html.yml` in the `~/ansible-files/` directory.

> **Tip**
>
> Note how the variable "stage" is used in the name of the file to copy.

<!-- {% raw %} -->
```yaml
---
- name: Copy index.html
  hosts: web
  become: yes
  tasks:
  - name: copy index.html
    copy:
      src: ~/ansible-files/{{ stage }}_index.html
      dest: /var/www/html/index.html
```
<!-- {% endraw %} -->

  - Run the Playbook:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook deploy_index_html.yml
```

## Step 4.4 - Test the Result

The Playbook should copy different files as index.html to the hosts, use `curl` to test it. Check the inventory again if you forgot the IP addresses of your nodes.

```bash
[student<X>@ansible ansible-files]$ grep node ~/lab_inventory/hosts
node1 ansible_host=11.22.33.44
node2 ansible_host=22.33.44.55
node3 ansible_host=33.44.55.66
[student<X>@ansible ansible-files]$ curl http://11.22.33.44
<body>
<h1>This is a development webserver, have fun!</h1>
</body>
[student1@ansible ansible-files]$ curl http://22.33.44.55
<body>
<h1>This is a production webserver, take care!</h1>
</body>
[student1@ansible ansible-files]$ curl http://33.44.55.66
<body>
<h1>This is a development webserver, have fun!</h1>
</body>
```

> **Tip**
>
> If by now you think: There has to be a smarter way to change content in files…​ you are absolutely right. This lab was done to introduce variables, you are about to learn about templates in one of the next chapters.

## Step 4.5 - Ansible Facts

Ansible facts are variables that are automatically discovered by Ansible from a managed host. Remember the "Gathering Facts" task listed in the output of each `ansible-playbook` execution? At that moment the facts are gathered for each managed node. Facts can also be pulled by the `setup` module. They contain useful information stored into variables that administrators can reuse.

To get an idea what facts Ansible collects by default, on your control node as your student user run:

```bash
[student<X>@ansible ansible-files]$ ansible node1 -m setup
```

This might be a bit too much, you can use filters to limit the output to certain facts, the expression is shell-style wildcard:

```bash
[student<X>@ansible ansible-files]$ ansible node1 -m setup -a 'filter=ansible_eth0'
```
Or what about only looking for memory related facts:

```bash
[student<X>@ansible ansible-files]$ ansible node1 -m setup -a 'filter=ansible_*_mb'
```

## Step 4.6 - Challenge Lab: Facts

  - Try to find and print the distribution (Red Hat) of your managed hosts. On one line, please.

> **Tip**
>
> Use grep to find the fact, then apply a filter to only print this fact.

> **Warning**
>
> **Solution below\!**

```bash
[student<X>@ansible ansible-files]$ ansible node1 -m setup|grep distribution
[student<X>@ansible ansible-files]$ ansible node1 -m setup -a 'filter=ansible_distribution' -o
```

## Step 4.7 - Using Facts in Playbooks

Facts can be used in a Playbook like variables, using the proper naming, of course. Create this Playbook as `facts.yml` in the `~/ansible-files/` directory:

<!-- {% raw %} -->
```yaml    
---
- name: Output facts within a playbook
  hosts: all
  tasks:
  - name: Prints Ansible facts
    debug:
      msg: The default IPv4 address of {{ ansible_fqdn }} is {{ ansible_default_ipv4.address }}
```
<!-- {% endraw %} -->

> **Tip**
>
> The "debug" module is handy for e.g. debugging variables or expressions.

Execute it to see how the facts are printed:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook facts.yml

PLAY [Output facts within a playbook] ******************************************

TASK [Gathering Facts] *********************************************************
ok: [node3]
ok: [node2]
ok: [node1]
ok: [ansible]

TASK [Prints Ansible facts] ****************************************************
ok: [node1] =>
  msg: The default IPv4 address of node1 is 172.16.190.143
ok: [node2] =>
  msg: The default IPv4 address of node2 is 172.16.30.170
ok: [node3] =>
  msg: The default IPv4 address of node3 is 172.16.140.196
ok: [ansible] =>
  msg: The default IPv4 address of ansible is 172.16.2.10

PLAY RECAP *********************************************************************
ansible                    : ok=2    changed=0    unreachable=0    failed=0   
node1                      : ok=2    changed=0    unreachable=0    failed=0   
node2                      : ok=2    changed=0    unreachable=0    failed=0   
node3                      : ok=2    changed=0    unreachable=0    failed=0   
```

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
# Exercise 1.5 - Conditionals, Handlers and Loops

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

## Step 5.1 - Conditionals

Ansible can use conditionals to execute tasks or plays when certain conditions are met.

To implement a conditional, the `when` statement must be used, followed by the condition to test. The condition is expressed using one of the available operators like e.g. for comparison:

|      |                                                                        |
| ---- | ---------------------------------------------------------------------- |
| \==  | Compares two objects for equality.                                     |
| \!=  | Compares two objects for inequality.                                   |
| \>   | true if the left hand side is greater than the right hand side.        |
| \>=  | true if the left hand side is greater or equal to the right hand side. |
| \<   | true if the left hand side is lower than the right hand side.          |
| \< = | true if the left hand side is lower or equal to the right hand side.   |

For more on this, please refer to the documentation: <http://jinja.pocoo.org/docs/2.10/templates/>

As an example you would like to install an FTP server, but only on hosts that are in the "ftpserver" inventory group.

To do that, first edit the inventory to add another group, and place `node2` in it. Make sure that that IP address of `node2` is always the same when `node2` is listed. Edit the inventory `~/lab_inventory/hosts` to look like the following listing:

```ini
[all:vars]
ansible_user=student1
ansible_ssh_pass=ansible
ansible_port=22

[web]
node1 ansible_host=11.22.33.44
node2 ansible_host=22.33.44.55
node3 ansible_host=33.44.55.66

[ftpserver]
node2 ansible_host=22.33.44.55

[control]
ansible ansible_host=44.55.66.77
```

Next create the file `ftpserver.yml` on your control host in the `~/ansible-files/` directory:

```yaml
---
- name: Install vsftpd on ftpservers
  hosts: all
  become: yes
  tasks:
    - name: Install FTP server when host in ftpserver group
      yum:
        name: vsftpd
        state: latest
      when: inventory_hostname in groups["ftpserver"]
```

> **Tip**
>
> By now you should know how to run Ansible Playbooks, we’ll start to be less verbose in this guide. Go create and run it. :-)

Run it and examine the output. The expected outcome: The task is skipped on node1, node3 and the ansible host (your control host) because they are not in the ftpserver group in your inventory file.

```bash
TASK [Install FTP server when host in ftpserver group] *******************************************
skipping: [ansible]
skipping: [node1]
skipping: [node3]
changed: [node2]
```

## Step 5.2 - Handlers

Sometimes when a task does make a change to the system, an additional task or tasks may need to be run. For example, a change to a service’s configuration file may then require that the service be restarted so that the changed configuration takes effect.

Here Ansible’s handlers come into play. Handlers can be seen as inactive tasks that only get triggered when explicitly invoked using the "notify" statement. Read more about them in the [Ansible Handlers](http://docs.ansible.com/ansible/latest/playbooks_intro.html#handlers-running-operations-on-change) documentation.

As a an example, let’s write a Playbook that:

  - manages Apache’s configuration file `httpd.conf` on all hosts in the `web` group

  - restarts Apache when the file has changed

First we need the file Ansible will deploy, let’s just take the one from node1. Remember to replace the IP address shown in the listing below with the IP address from your individual `node1`.

```bash
[student<X>@ansible ansible-files]$ scp 11.22.33.44:/etc/httpd/conf/httpd.conf ~/ansible-files/.
student<X>@11.22.33.44's password:
httpd.conf             
```

Next, create the Playbook `httpd_conf.yml`. Make sure that you are in the directory `~/ansible-files`.

```yaml
---
- name: manage httpd.conf
  hosts: web
  become: yes
  tasks:
  - name: Copy Apache configuration file
    copy:
      src: httpd.conf
      dest: /etc/httpd/conf/
    notify:
        - restart_apache
  handlers:
    - name: restart_apache
      service:
        name: httpd
        state: restarted
```

So what’s new here?

  - The "notify" section calls the handler only when the copy task actually changes the file. That way the service is only restarted if needed - and not each time the playbook is run.

  - The "handlers" section defines a task that is only run on notification.

Run the Playbook. We didn’t change anything in the file yet so there should not be any `changed` lines in the output and of course the handler shouldn’t have fired.

  - Now change the `Listen 80` line in httpd.conf to:


```ini
Listen 8080
```

  - Run the Playbook again. Now Ansible’s output should be a lot more interesting:

      - httpd.conf should have been copied over

      - The handler should have restarted Apache

Apache should now listen on port 8080. Easy enough to verify:

```bash
[student1@ansible ansible-files]$ curl http://22.33.44.55
curl: (7) Failed connect to 22.33.44.55:80; Connection refused
[student1@ansible ansible-files]$ curl http://22.33.44.55:8080
<body>
<h1>This is a production webserver, take care!</h1>
</body>
```
Feel free to change the httpd.conf file again and run the Playbook.

## Step 5.3 - Simple Loops

Loops enable us to repeat the same task over and over again. For example, lets say you want to create multiple users. By using an Ansible loop, you can do that in a single task. Loops can also iterate over more than just basic lists. For example, if you have a list of users with their coresponding group, loop can iterate over them as well. Find out more about loops in the [Ansible Loops](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html) documentation.

To show the loops feature we will generate three new users on `node1`. For that, create the Playbook `loop_users.yml` in `~/ansible-files` on your control node as your student user and run it. We will use the `user` module to generate the user accounts.

<!-- {% raw %} -->
```yaml
---
- name: Ensure users
  hosts: node1
  become: yes

  tasks:
    - name: Ensure three users are present
      user:
        name: "{{ item }}"
        state: present
      loop:
         - dev_user
         - qa_user
         - prod_user
```
<!-- {% endraw %} -->

Understand the playbook and the output:

<!-- {% raw %} -->
  - The names are not provided to the user module directly. Instead, there is only a variable called `{{ item }}` for the parameter `name`.

  - The `loop` keyword lists the actual user names. Those replace the `{{ item }}` during the actual execution of the playbook.

  - During execution the task is only listed once, but there are three changes listed underneath it.
<!-- {% endraw %} -->

## Step 5.4 - Loops over hashes

As mentioned loops can also be over lists of hashes. Imagine that the users should be assigned to different additional groups:

```yaml
- username: dev_user
  groups: ftp
- username: qa_user
  groups: ftp
- username: prod_user
  groups: apache
```

The `user` module has the optional parameter `groups` to list additional users. To reference items in a hash, the `{{ item }}` keyword needs to reference the subkey: `{{ item.groups }}` for example.

Let's rewrite the playbook to create the users with additional user rights:

<!-- {% raw %} -->
```yaml
---
- name: Ensure users
  hosts: node1
  become: yes

  tasks:
    - name: Ensure three users are present
      user:
        name: "{{ item.username }}"
        state: present
        groups: "{{ item.groups }}"
      loop:
        - { username: 'dev_user', groups: 'ftp' }
        - { username: 'qa_user', groups: 'ftp' }
        - { username: 'prod_user', groups: 'apache' }

```
<!-- {% endraw %} -->

Check the output:

  - Again the task is listed once, but three changes are listed. Each loop with its content is shown.

Verify that the user `prod_user` was indeed created on `node1`:

```bash
[student<X>@ansible ansible-files]$ ansible node1 -m command -a "id dev_user"
node1 | CHANGED | rc=0 >>
uid=1002(dev_user) gid=1002(dev_user) Gruppen=1002(dev_user),50(ftp)
```

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
# Exercise 1.6 - Templates

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

Ansible uses Jinja2 templating to modify files before they are distributed to managed hosts. Jinja2 is one of the most used template engines for Python (<http://jinja.pocoo.org/>).

## Step 6.1 - Using Templates in Playbooks

When a template for a file has been created, it can be deployed to the managed hosts using the `template` module, which supports the transfer of a local file from the control node to the managed hosts.

As an example of using templates you will change the motd file to contain host-specific data.

First in the `~/ansible-files/` directory create the template file `motd-facts.j2`:

<!-- {% raw %} -->
```html+jinja
Welcome to {{ ansible_hostname }}.
{{ ansible_distribution }} {{ ansible_distribution_version}}
deployed on {{ ansible_architecture }} architecture.
```
<!-- {% endraw %} -->

The template file contains the basic text that will later be copied over. It also contains variables which will be replaced on the target machines individually.

Next we need a playbook to use this template. In the `~/ansible-files/` directory create the Playbook `motd-facts.yml`:

```yaml
---
- name: Fill motd file with host data
  hosts: node1
  become: yes
  tasks:
    - template:
        src: motd-facts.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: 0644
```

You have done this a couple of times by now:

  - Understand what the Playbook does.

  - Execute the Playbook `motd-facts.yml`.

  - Login to node1 via SSH and check the message of the day content.

  - Log out of node1.

You should see how Ansible replaces the variables with the facts it discovered from the system.

## Step 6.2 - Challenge Lab

Add a line to the template to list the current kernel of the managed node.

  - Find a fact that contains the kernel version using the commands you learned in the "Ansible Facts" chapter.

> **Tip**
>
> Do a `grep -i` for kernel

  - Change the template to use the fact you found.

  - Run the Playbook again.

  - Check motd by logging in to node1

> **Warning**
>
> **Solution below\!**


  - Find the fact:

```bash
[student<X>@ansible ansible-files]$ ansible node1 -m setup|grep -i kernel
       "ansible_kernel": "3.10.0-693.el7.x86_64",
```

  - Modify the template `motd-facts.j2`:

<!-- {% raw %} -->
```html+jinja
Welcome to {{ ansible_hostname }}.
{{ ansible_distribution }} {{ ansible_distribution_version}}
deployed on {{ ansible_architecture }} architecture
running kernel {{ ansible_kernel }}.
```
<!-- {% endraw %} -->

  - Run the playbook.
  - Verify the new message via SSH login to `node1`.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
# Exercise 1.7 - Roles: Making your playbooks reusable

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

While it is possible to write a playbook in one file as we've done throughout this workshop, eventually you’ll want to reuse files and start to organize things.

Ansible Roles are the way we do this.  When you create a role, you deconstruct your playbook into parts and those parts sit in a directory structure.  This is explained in more detail in the [best practice](http://docs.ansible.com/ansible/playbooks_best_practices.html) already mentioned in exercise 3.

## Step 7.1 - Understanding the Ansible Role Structure

Roles are basically automation built around *include* directives and really don’t contain much additional magic beyond some improvements to search path handling for referenced files.

Roles follow a defined directory structure; a role is named by the top level directory. Some of the subdirectories contain YAML files, named `main.yml`. The files and templates subdirectories can contain objects referenced by the YAML files.

An example project structure could look like this, the name of the role would be "apache":

```text
apache/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

The various `main.yml` files contain content depending on their location in the directory structure shown above. For instance, `vars/main.yml` references variables, `handlers/main.yaml` describes handlers, and so on. Note that in contrast to playbooks, the `main.yml` files only contain the specific content and not additional playbook information like hosts, `become` or other keywords.

> **Tip**
>
> There are actually two directories for variables: `vars` and `default`: Default variables have the lowest precedence and usually contain default values set by the role authors and are often used when it is intended that their values will be overridden.. Variables can be set in either `vars/main.yml` or `defaults/main.yml`, but not in both places.

Using roles in a Playbook is straight forward:

```yaml
---
- name: launch roles
  hosts: web
  roles:
    - role1
    - role2
```

For each role, the tasks, handlers and variables of that role will be included in the Playbook, in that order. Any copy, script, template, or include tasks in the role can reference the relevant files, templates, or tasks *without absolute or relative path names*. Ansible will look for them in the role's files, templates, or tasks respectively, based on their
use.

## Step 7.2 - Create a Basic Role Directory Structure

Ansible looks for roles in a subdirectory called `roles` in the project directory. This can be overridden in the Ansible configuration. Each role has its own directory. To ease creation of a new role the tool `ansible-galaxy` can be used.

> **Tip**
>
> Ansible Galaxy is your hub for finding, reusing and sharing the best Ansible content. `ansible-galaxy` helps to interact with Ansible Galaxy. For now we'll just using it as a helper to build the directory structure.

Okay, lets start to build a role. We'll build a role that installs and configures Apache to serve a virtual host. Run these commands in your `~/ansible-files` directory:

```bash
[student<X>@ansible ansible-files]$ mkdir roles
[student<X>@ansible ansible-files]$ ansible-galaxy init --offline roles/apache_vhost
```

Have a look at the role directories and their content:

```bash
[student<X>@ansible ansible-files]$ tree roles
```

## Step 7.3 - Create the Tasks File

The `main.yml` file in the tasks subdirectory of the role should do the following:

  - Make sure httpd is installed

  - Make sure httpd is started and enabled

  - Put HTML content into the Apache document root

  - Install the template provided to configure the vhost

> **WARNING**
>
> **The `main.yml` (and other files possibly included by main.yml) can only contain tasks, *not* complete Playbooks!**

Change into the `roles/apache_vhost` directory. Edit the `tasks/main.yml` file:

```yaml
---
- name: install httpd
  yum:
    name: httpd
    state: latest

- name: start and enable httpd service
  service:
    name: httpd
    state: started
    enabled: true
```

Note that here just tasks were added. The details of a playbook are not present.

The tasks added so far do:

  - Install the httpd package using the yum module

  - Use the service module to enable and start httpd

Next we add two more tasks to ensure a vhost directory structure and copy html content:

<!-- {% raw %} -->
```yaml
- name: ensure vhost directory is present
  file:
    path: "/var/www/vhosts/{{ ansible_hostname }}"
    state: directory

- name: deliver html content
  copy:
    src: index.html
    dest: "/var/www/vhosts/{{ ansible_hostname }}"
```
<!-- {% endraw %} -->

Note that the vhost directory is created/ensured using the `file` module.

The last task we add uses the template module to create the vhost configuration file from a j2-template:

```yaml
- name: template vhost file
  template:
    src: vhost.conf.j2
    dest: /etc/httpd/conf.d/vhost.conf
    owner: root
    group: root
    mode: 0644
  notify:
    - restart_httpd
```
Note it is using a handler to restart httpd after an configuration update.

The full `tasks/main.yml` file is:

<!-- {% raw %} -->
```yaml
---
- name: install httpd
  yum:
    name: httpd
    state: latest

- name: start and enable httpd service
  service:
    name: httpd
    state: started
    enabled: true

- name: ensure vhost directory is present
  file:
    path: "/var/www/vhosts/{{ ansible_hostname }}"
    state: directory

- name: deliver html content
  copy:
    src: index.html
    dest: "/var/www/vhosts/{{ ansible_hostname }}"

- name: template vhost file
  template:
    src: vhost.conf.j2
    dest: /etc/httpd/conf.d/vhost.conf
    owner: root
    group: root
    mode: 0644
  notify:
    - restart_httpd
```
<!-- {% endraw %} -->


## Step 7.4 - Create the handler

Create the handler in the file `handlers/main.yml` to restart httpd when notified by the template task:

```yaml
---
# handlers file for roles/apache_vhost
- name: restart_httpd
  service:
    name: httpd
    state: restarted
```

## Step 7.5 - Create the index.html and vhost configuration file template

Create the HTML content that will be served by the webserver.

  - Create an index.html file in the "src" directory of the role, `files`:

```bash
[student<X>@ansible ansible-files]$ echo 'simple vhost index' > ~/ansible-files/roles/apache_vhost/files/index.html
```

  - Create the `vhost.conf.j2` template file in the role's `templates` subdirectory.

<!-- {% raw %} -->
```html
# {{ ansible_managed }}

<VirtualHost *:8080>
    ServerAdmin webmaster@{{ ansible_fqdn }}
    ServerName {{ ansible_fqdn }}
    ErrorLog logs/{{ ansible_hostname }}-error.log
    CustomLog logs/{{ ansible_hostname }}-common.log common
    DocumentRoot /var/www/vhosts/{{ ansible_hostname }}/

    <Directory /var/www/vhosts/{{ ansible_hostname }}/>
  Options +Indexes +FollowSymlinks +Includes
  Order allow,deny
  Allow from all
    </Directory>
</VirtualHost>
```
<!-- {% endraw %} -->

## Step 7.6 - Test the role

You are ready to test the role against `node2`. But since a role cannot be assigned to a node directly, first create a playbook which connects the role and the host. Create the file `test_apache_role.yml` in the directory `~/ansible-files`:

```yaml
---
- name: use apache_vhost role playbook
  hosts: node2
  become: yes

  pre_tasks:
    - debug:
        msg: 'Beginning web server configuration.'

  roles:
    - apache_vhost

  post_tasks:
    - debug:
        msg: 'Web server has been configured.'
```

Note the `pre_tasks` and `post_tasks` keywords. Normally, the tasks of roles execute before the tasks of a playbook. To control order of execution `pre_tasks` are performed before any roles are applied. The `post_tasks` are performed after all the roles have completed. Here we just use them to better highlight when the actual role is executed.

Now you are ready to run your playbook:

```bash
[student<X>@ansible ansible-files]$ ansible-playbook test_apache_role.yml
```

Run a curl command against `node2` to confirm that the role worked (remember Apache is still listening on port 8080):

```bash
[student<X>@ansible ansible-files]$ curl -s http://22.33.44.55:8080
simple vhost index
```

All looking good? Congratulations! You have successfully completed the Ansible Engine Workshop Exercises!

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
# Exercise 1.8 - Bonus Labs

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

You have finished the lab already. But it doesn’t have to end here. We prepared some slightly more advanced bonus labs for you to follow through if you like. So if you are done with the labs and still have some time, here are some more labs for you:

## Step 8.1 - Bonus Lab: Ad Hoc Commands

Create a new user "testuser" on `node1` and `node3` with a comment using an ad hoc command, make sure that it is not created on `node2`!

  - Find the parameters for the appropriate module using `ansible-doc user` (leave with `q`)

  - Use an Ansible ad hoc command to create the user with the comment "Test D User"

  - Use the "command" module with the proper invocation to find the userid

  - Delete the user and check it has been deleted

> **Tip**
>
> Remember privilege escalation…​

> **Warning**
>
> **Solution below\!**

Your commands could look like these:

```bash
[student<X>@ansible ansible-files]$ ansible-doc -l | grep -i user
[student<X>@ansible ansible-files]$ ansible-doc user
[student<X>@ansible ansible-files]$ ansible node1,node3 -m user -a "name=testuser comment='Test D User'" -b
[student<X>@ansible ansible-files]$ ansible node1,node3 -m command -a " id testuser" -b
[student<X>@ansible ansible-files]$ ansible node2 -m command -a " id testuser" -b
[student<X>@ansible ansible-files]$ ansible node1,node3 -m user -a "name=testuser state=absent remove=yes" -b
[student<X>@ansible ansible-files]$ ansible web -m command -a " id testuser" -b
```

## Step 8.2 - Bonus Lab: Templates and Variables

You have learned the basics about Ansible templates, variables and handlers. Let’s combine all of these.

Instead of editing and copying `httpd.conf` why don’t you just define a variable for the listen port and use it in a template? Here is your job:

  - Define a variable `listen_port` for the `web` group with the value `8080` and another for `node2` with the value `80` using the proper files.

  - Copy the `httpd.conf` file into the template `httpd.conf.j2` that uses the `listen_port` variable instead of the hard-coded port number.

  - Write a Playbook that deploys the template and restarts Apache on changes using a handler.

  - Run the Playbook and test the result using `curl`.

> **Tip**
>
> Remember the `group_vars` and `host_vars` directories? If not, refer to the chapter "Ansible Variables".


> **Warning**
>
> **Solution below\!**

### Define the variables:


Add this line to `group_vars/web`:

```ini
listen_port: 8080
```

Add this line to `host_vars/node2`:

```ini
listen_port: 80
```
### Prepare the template:

  - Copy `httpd.conf` to `httpd.conf.j2`

  - Edit the "Listen" directive in `httpd.conf.j2` to make it look like this:

<!-- {% raw %} -->
```ini
[...]
Listen {{ listen_port }}
[]...]
```
<!-- {% endraw %} -->

### Create the Playbook

Create a playbook called `apache_config_tpl.yml`:

```yaml
---
- name: Apache httpd.conf
  hosts: web
  become: yes
  tasks:
  - name: Create Apache configuration file from template
    template:
      src: httpd.conf.j2
      dest: /etc/httpd/conf/httpd.conf
    notify:
        - restart apache
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```

### Run and test

First run the playbook itself, then run curl against `node1` with port `8080` and `node2` with port `80`.

```bash
[student1@ansible ansible-files]$ ansible-playbook apache_config_tpl.yml
[...]
[student1@ansible ansible-files]$ curl http://18.195.235.231:8080
<body>
<h1>This is a development webserver, have fun!</h1>
</body>
[student1@ansible ansible-files]$ curl http://35.156.28.209:80
<body>
<h1>This is a production webserver, take care!</h1>
</body>
```

----

