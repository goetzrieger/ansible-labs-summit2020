# Exercise 3 - There is more to Tower than the Web UI

This is an advanced Tower lab so we don’t really want you to use the web
UI for everything. Tower’s web UI is well done and helps with a lot of
tasks, but same as in system administration it’s often handy to be able
to use the command line or scripts for certain tasks.

We’ve incorporated different ways to work with Tower in this lab guide
and hope you’ll find it helpful. The first step we do is install the
**AWX CLI** utility.

> **Tip**
>
> **AWX CLI** is the official command-line client for AWX and Red Hat
> Ansible Tower. It uses naming and structure consistent with the AWX
> HTTP API, provides consistent output formats with optional
> machine-parsable formats.

We’ll install it on your control host using the official repository RPM
packages. Exit the SSH session to **tower1.example.com** or open a new
one to the control host, then install **AWX CLI** as `root`:

    yum-config-manager --add-repo https://releases.ansible.com/ansible-tower/cli/ansible-tower-cli-el7.repo
    yum install ansible-awx-cli -y

After installing the tool, you have to do some basic configuration
(Tower node to connect to, user and password):

    [root@bastion ~]# awx-cli config host tower1.ewl04.internal
    [root@bastion ~]# awx-cli config username admin
    [root@bastion ~]# awx-cli config password r3dh4t1!

> **Tip**
>
> It doesn’t really matter what node you connect to.

Now test **awx-cli** is working. First run it without arguments to get a
list of resources you can manage with it:

    [root@control ~]# awx-cli --help

And then test something, e.g.:

    [root@control ~]# awx-cli user list

> **Tip**
>
> When trying to find a **awx-cli** command line for something you want
> to do, just move one by one.

Example:

    [root@control ~]# awx-cli --help

Okay, there is an **inventory** resource. Let’s see…

    [root@control ~]# awx-cli inventory --help

Well, **create** sounds like what I had in mind. But what arguments do I
need? Just run:

    [root@control ~]# awx-cli inventory create

Bingo\! Take note of the **REQUIRED** mark.

## Challenge Lab: awx-cli

To practice your **awx-cli** skills, here is a challenge:

  - Try to change the **idle time out** of the Tower web UI, it’s 1800
    seconds by default. Set it to, say, 7200. Using **awx-cli**, of
    course.

  - Start by looking for a resource type **awx-cli** provides using
    **--help** that sounds like it has something to do with changing
    settings.

  - Look at the available **awx-cli** commands for this resource type.

  - Use the commands to have a look at the parameters settings and
    change it.

> **Tip**
>
> The configuration parameter is called **SESSION\_COOKIE\_AGE**

> **Warning**
>
> **SOLUTION BELOW\!**

**\>\> *Click here for the solution* \<\<**

    [root@control ~]# awx-cli setting
    [root@control ~]# awx-cli setting get SESSION_COOKIE_AGE
    [root@control ~]# awx-cli setting modify SESSION_COOKIE_AGE 7200
    [root@control ~]# awx-cli setting get SESSION_COOKIE_AGE

\</div\>\</details\>

If you want to, go to the web UI of any node (not just the one you
connected **awx-cli** to) and check the setting under
**ADMINISTRATION→Settings→System**.
