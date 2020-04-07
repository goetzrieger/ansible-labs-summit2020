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

We’ll install it on your Tower node 1 using the official repository RPM
packages. Using the VSCode terminal window you opened before install **AWX CLI** as `root`:

    # yum-config-manager --add-repo https://releases.ansible.com/ansible-tower/cli/ansible-tower-cli-el7.repo
    # yum install ansible-tower-cli -y

After installing the tool, you have to configure authentication. The preferred way is to create a token and export it into an environment variable. After this you can seemlessly use **awx** commands in this shell. First set a number of env variables to define your connection:

    [root@ansible ~]# export TOWER_HOST=https://student<N>.ansible.<LABID>.rhdemo.io
    [root@ansible ~]# export TOWER_USERNAME=admin
    [root@ansible ~]# export TOWER_PASSWORD='r3dh4t1!'
    [root@ansible ~]# export TOWER_VERIFY_SSL=false

Then use **awx** to login and print out the access token:

    [root@ansible ~]# awx login -f human
    export TOWER_TOKEN=<YOUR_TOKEN>

Finally set the environment variable with the token:

    [root@ansible ~]# export TOWER_TOKEN=<YOUR_TOKEN>

Now test **awx** is working. First run it without arguments to get a
list of resources you can manage with it:

    [root@ansible ~]# awx --help

And then test something, e.g. (leave out **-f human** if you're a JSON fan...):

    [root@ansible ~]# awx -f human user list

> **Tip**
>
> When trying to find a **awx** command line for something you want
> to do, just move one by one.

Example:

    [root@ansible ~]# awx --help

Okay, there is an **inventory** resource. Let’s see…

    [root@ansible ~]# awx inventory --help

Well, **create** sounds like what I had in mind. But what arguments do I
need? Just run:

    [root@ansible ~]# awx -f human inventory create

Bingo! Take note of the **REQUIRED** information at the end.

## Challenge Lab: awx

To practice your **awx** skills, here is a challenge:

  - Try to change the **idle time out** of the Tower web UI, it’s 1800
    seconds by default. Set it to, say, 7200. Using **awx**, of
    course.

  - Start by looking for a resource type **awx** provides using
    **--help** that sounds like it has something to do with changing
    settings.

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
>     [root@ansible ~]# awx setting list | grep SESSION
>     [root@ansible ~]# awx setting modify SESSION_COOKIE_AGE 7200
>     [root@ansible ~]# awx setting list | grep SESSION
>
> </p>
> </details>

If you want to, go to the web UI of any node (not just the one you
connected **awx** to) and check the setting under
**ADMINISTRATION→Settings→System**.
