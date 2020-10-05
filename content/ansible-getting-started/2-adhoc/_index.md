+++
title = "Running ad hoc commands"
weight = 2
+++

For our first exercise, we are going to run some ad hoc commands to help you get a feel for how Ansible works.  Ansible ad hoc commands enable you to perform tasks on remote nodes without having to write a playbook.  They are very useful when you simply need to do one or two things quickly and often, to many remote nodes.

## Work with your Inventory

To use the ansible command for host management, you need to provide an inventory file which defines a list of hosts to be managed from the control node. In this lab the inventory is provided by your instructor. The inventory is an ini formatted file listing your hosts, sorted in groups, additionally providing some variables. Have a look for yourself, in your **code-server terminal** run:

```bash
[{{< param "internal_control" >}} ~]$ cat lab_inventory/hosts

[all:vars]
ansible_user=student{{< param "student" >}}
ansible_ssh_pass=<password>
ansible_port=22

[web]
node1 ansible_host=<IP address>
node2 ansible_host=<IP address>
node3 ansible_host=<IP address>

[control]
ansible ansible_host=<IP address>
```

{{% notice tip %}}
The environment for this lab uses SSH with password authentication to login to the managed nodes. For the sake of keeping things simple the password is put into the inventory file in clear text. In real world scenarios you would either use SSH key authentication or supply the password in a secure way, e.g. by using Ansible Vault.
{{% /notice %}}

{{% notice tip %}}
Note that each student has an individual lab environment so we left out the actual IPs and other data. As with the other cases, replace **\<N\>** with your actual student number.
{{% /notice %}}

Ansible is already configured to use the inventory specific to your environment, you'll learn how in a minute. For now let's execute some simple commands to work with the inventory.

To reference inventory hosts, you supply a host pattern to the ansible command. Ansible has a `--list-hosts` option which can be useful for clarifying which managed hosts are referenced by the host pattern in an ansible command.

The most basic host pattern is the name for a single managed host listed in the inventory file. This specifies that the host will be the only one in the inventory file that will be acted upon by the ansible command. Run:

```bash
[{{< param "internal_control" >}} ~]$ ansible node1 --list-hosts
  hosts (1):
    node1
```

An inventory file can contain a lot more information, it can organize your hosts in groups or define variables. In our example, the current inventory has the groups `web` and `control`. Run Ansible with these host patterns and observe the output:

```bash
[{{< param "internal_control" >}} ~]$ ansible web --list-hosts
[{{< param "internal_control" >}} ~]$ ansible web,ansible --list-hosts
[{{< param "internal_control" >}} ~]$ ansible 'node*' --list-hosts
[{{< param "internal_control" >}} ~]$ ansible all --list-hosts
```

As you see it is OK to put systems in more than one group. For instance, a server could be both a web server and a database server. Note that in Ansible the groups are not necessarily hierarchical.

{{% notice tip %}}
The inventory can contain more data. E.g. if you have hosts that run on non-standard SSH ports you can put the port number after the hostname with a colon. Or you could define names specific to Ansible and have them point to the "real" IP or hostname.
{{% /notice %}}

## The Ansible Configuration Files

The behavior of Ansible can be customized by modifying settings in Ansible’s ini-style configuration file. Ansible will select its configuration file from one of several possible locations on the control node, please refer to the [documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html).

{{% notice tip %}}
The recommended practice is to create an `ansible.cfg` file in the directory from which you run Ansible commands. This directory would also contain any files used by your Ansible project, such as the inventory and playbooks. Another recommended practice is to create a file `.ansible.cfg` in your home directory.
{{% /notice %}}

In the lab environment provided to you an `.ansible.cfg` file has already been created and filled with the necessary details in the home directory of your `student{{< param "student" >}}` user on the control node:

```bash
[{{< param "internal_control" >}} ~]$ ls -la .ansible.cfg
-rw-r--r--. 1 student{{< param "student" >}} student{{< param "student" >}} 231 14. Mai 17:17 .ansible.cfg
```

Review the content of the file:

```bash
[{{< param "internal_control" >}} ~]$ cat .ansible.cfg
[defaults]
stdout_callback = yaml
connection = smart
timeout = 60
deprecation_warnings = False
host_key_checking = False
retry_files_enabled = False
inventory = /home/student{{student}}/lab_inventory/hosts
```

There are multiple configuration flags provided. Most of them are not of interest here, but make sure to note the last line: there the location of the inventory is provided. That is the way Ansible knew in the previous commands what inventory to use to lookup the nodes.

## Ping a host

Let's start with something really basic - pinging a host. To do that we use the Ansible `ping` module. Basically, the `ping` module connects to the managed host, executes a small script there and collects the results. This ensures that the managed host is reachable and that Ansible is able to execute commands properly on it.

{{% notice tip %}}
Think of a module as a tool which is designed to accomplish a specific task.
{{% /notice %}}

Ansible needs to know that it should use the `ping` module: The `-m` option defines which Ansible module to use. Options can be passed to the specified module using the `-a` option. In addition to the module Ansible needs to know what hosts it should run the task on, here you supply the group `web`.

```bash
[{{< param "internal_control" >}} ~]$ ansible web -m ping
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

## Listing Modules and Getting Help

Ansible comes with a lot of modules by default. To list all modules run:

```bash
[{{< param "internal_control" >}} ~]$ ansible-doc -l
```

{{% notice tip %}}
In `ansible-doc` leave by pressing the button `q`. Use the `up`/`down` arrows to scroll through the content.
{{% /notice %}}

To find a module try e.g.:

```bash
[{{< param "internal_control" >}} ~]$ ansible-doc -l | grep -i user
```

Get help for a specific module including usage examples:

```bash
[{{< param "internal_control" >}} ~]$ ansible-doc user
```

{{% notice tip %}}
Mandatory options are marked by a "=" in `ansible-doc`, optional ones by a "-".
{{% /notice %}}

## Use the command module

Now let's see how we can run a good ol' fashioned Linux command and format the output using the `command` module. It simply executes the specified command on a managed host (note this time not a group but a hostname is used as host pattern):

```bash
[{{< param "internal_control" >}} ~]$ ansible node1 -m command -a "id"
node1 | CHANGED | rc=0 >>
uid=1001(student{{< param "student" >}}) gid=1001(student{{ student }) Gruppen=1001(student{{< param "student" >}}) Kontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

In this case the module is called `command` and the option passed with `-a` is the actual command to run. Try to run this ad hoc command on all managed hosts using the `all` host pattern.

Another example: Have a quick look at the kernel versions your hosts are running:

```bash
[{{< param "internal_control" >}} ~]$ ansible all -m command -a 'uname -r'
```

Sometimes it’s desirable to have the output for a host on one line:

```bash
[{{< param "internal_control" >}} ~]$ ansible all -m command -a 'uname -r' -o
```

{{% notice tip %}}
Like many Linux commands, `ansible` allows long-form options as well as short-form.  For example `ansible web --module-name ping` is the same as running `ansible web -m ping`.  We are going to be using the short-form options throughout this workshop.
{{% /notice %}}

## The copy module and permissions

Using the `copy` module, execute an ad hoc command on `node1` to change the contents of the `/etc/motd` file. **The content is handed to the module through an option in this case**.

Run the following, but **expect an error**:

```bash
[{{< param "internal_control" >}} ~]$ ansible node1 -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd'
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

The output of the ad hoc command is screaming **FAILED** in red at you. Why? Because user **student{{< param "student" >}}** is not allowed to write the motd file.

Now this is a case for privilege escalation and the reason `sudo` has to be setup properly. We need to instruct Ansible to use `sudo` to run the command as root by using the parameter `-b` (think "become").

{{% notice tip %}}
Ansible will connect to the machines using your current user name (student{{< param "student" >}} in this case), just like SSH would. To override the remote user name, you could use the `-u` parameter.
{{% /notice %}}

For us it’s okay to connect as `student{{< param "student" >}}` because `sudo` is set up. Change the command to use the `-b` parameter and run again:

```bash
[{{< param "internal_control" >}} ~]$ ansible node1 -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -b
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
    "src": "/home/student{{< param "student" >}}/.ansible/tmp/ansible-tmp-1557857641.21-120920996103312/source",
    "state": "file",
    "uid": 0
```

Use Ansible with the generic `command` module to check the content of the motd file:

```bash
[{{< param "internal_control" >}} ~]$ ansible node1 -m command -a 'cat /etc/motd'
node1 | CHANGED | rc=0 >>
Managed by Ansible
```

Run the `ansible node1 -m copy …​` command from above again. Note:

- The different output color (proper terminal config provided).

- The change from `"changed": true,` to `"changed": false,`.

- The first line says `SUCCESS` instead of `CHANGED`.

{{% notice tip %}}
This makes it a lot easier to spot changes and what Ansible actually did.
{{% /notice %}}

## Challenge Lab: Modules

- Using `ansible-doc`

  - Find a module that uses yum to manage software packages.

  - Look up the help examples for the module to learn how to install a package in the latest version.

- Run an Ansible ad hoc command to install the package "vim" in the latest version on all available nodes.

{{% notice tip %}}
Use the copy ad hoc command from above as a template and change the module and options.
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>

<p>
```bash
[{{< param "internal_control" >}} ~]$ ansible-doc -l | grep -i yum
[{{< param "internal_control" >}} ~]$ ansible-doc yum
[{{< param "internal_control" >}} ~]$ ansible all -m yum -a 'name=vim state=latest' -b
```
</p>
</details>
