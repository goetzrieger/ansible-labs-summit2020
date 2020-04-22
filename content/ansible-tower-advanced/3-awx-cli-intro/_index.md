+++
title = "There is more to Tower than the Web UI"
weight = 3
+++

This is an advanced Tower lab so we don’t really want you to use the web UI for everything. Tower’s web UI is well done and helps with a lot of tasks, but same as in system administration it’s often handy to be able to use the command line or scripts for certain tasks.

We’ve incorporated different ways to work with Tower in this lab guide and hope you’ll find it helpful. The first step we do is install the **AWX CLI** utility.

> **Tip**
>
> **AWX CLI** is the official command-line client for AWX and Red Hat Ansible Tower. It uses naming and structure consistent with the AWX HTTP API, provides consistent output formats with optional machine-parsable formats.

We’ll install it on your Tower node 1 using the official repository RPM packages. Use the VSCode terminal window you opened before to install **AWX CLI** as `root`:

    [student@ansible ~]$ sudo -i
    [root@ansible ~]# yum-config-manager --add-repo https://releases.ansible.com/ansible-tower/cli/ansible-tower-cli-el7.repo
    [root@ansible ~]# yum install ansible-tower-cli -y
    [root@ansible ~] exit
    [student@ansible ~]$

> **WARNING**
>
> Please make sure to leave the root shell after installation of the package!


After installing the tool, you have to configure authentication. The preferred way is to create a token and export it into an environment variable. After this you can seemlessly use **awx** commands in this shell. First set a number of env variables to define your connection:

> **TIP**
>
> Replace student number and labid!

    [student@ansible ~]$ export TOWER_HOST=https://student<X>-ansible.<LABID>. events.opentlc.com
    [student@ansible ~]$ export TOWER_USERNAME=admin
    [student@ansible ~]$ export TOWER_PASSWORD='r3dh4t1!'
    [student@ansible ~]$ export TOWER_VERIFY_SSL=false

Then use **awx** to login and print out the access token:

    [student@ansible ~]$ awx login -f human
    export TOWER_TOKEN=<YOUR_TOKEN>

Finally set the environment variable with the token using the line the command printed out:

    [student@ansible ~]$ export TOWER_TOKEN=<YOUR_TOKEN>

Now that the access token is avalable in your shell, test **awx** is working. First run it without arguments to get a
list of resources you can manage with it:

    [student@ansible ~]$ awx --help

And then test something, e.g. (leave out **-f human** if you're a JSON fan...):

    [student@ansible ~]$ awx -f human user list

> **Tip**
>
> When trying to find a **awx** command line for something you want to do, just move one by one.

Example: Need to create an inventory...

    [student@ansible ~]$ awx --help

Okay, there is an **inventory** resource. Let’s see…

    [student@ansible ~]$ awx inventory

Well, the **create** action sounds like what I had in mind. But what arguments do I
need? Just run:

    [student@ansible ~]$ awx -f human inventory create

And you'll get the required and optional arguments for the **create** action!

## Challenge Lab: awx

To practice your **awx** skills, here is a challenge:

  - Try to change the **idle time out** of the Tower web UI, it’s 1800 seconds by default. Set it to, say, 7200. Using **awx**, of course.

  - Start by looking for a resource type **awx** provides using **--help** that sounds like it has something to do with changing settings.

  - Look at the available **awx** commands for this resource type.

  - Use the commands to have a look at the parameters settings and
    change it.

> **Tip**
>
> The configuration parameter is called **SESSION\_COOKIE\_AGE**

> **Warning**
>
> <details><summary>Solution below!</summary>
> <p>
>
>     [student@ansible ~]$ awx setting list | grep SESSION
>     [student@ansible ~]$ awx setting modify SESSION_COOKIE_AGE 7200
>     [student@ansible ~]$ awx setting list | grep SESSION
>
> </p>
> </details>

If you want to, go to the web UI of any node (not just the one you connected **awx** to) and check the setting under **ADMINISTRATION→Settings→System**.
