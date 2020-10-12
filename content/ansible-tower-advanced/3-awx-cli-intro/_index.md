+++
title = "There is more to Tower than the Web UI"
weight = 3
+++

This is an advanced Tower lab so we don’t really want you to use the web UI for everything. Tower’s web UI is well done and helps with a lot of tasks, but same as in system administration it’s often handy to be able to use the command line or scripts for certain tasks.

We’ve incorporated different ways to work with Tower in this lab guide and hope you’ll find it helpful. The first step we do is install the **AWX CLI** utility.

{{% notice tip %}}
**AWX CLI** is the official command-line client for AWX and Red Hat Ansible Tower. It uses naming and structure consistent with the AWX HTTP API, provides consistent output formats with optional machine-parsable formats.
{{% /notice %}}

We’ll install it on your Tower node1 using the official repository RPM packages. Use the VSCode terminal window you opened before to install **AWX CLI** as `root`:

```bash
[{{< param "control_prompt" >}} ~]$ sudo -i
[{{< param "internal_control" >}} ~]# yum-config-manager --add-repo https://releases.ansible.com/ansible-tower/cli/ansible-tower-cli-el8.repo
[{{< param "internal_control" >}} ~]# yum install ansible-tower-cli -y
[{{< param "internal_control" >}} ~]# exit
```

{{% notice warning %}}
Please make sure to leave the root shell after installation of the package!
{{% /notice %}}

After installing the tool, you have to configure authentication. The preferred way is to create a token and export it into an environment variable. After this you can seamlessly use **awx** commands in this shell. First set a number of environment variables to define your connection:

{{% notice tip %}}
Replace student number and LABID!
{{% /notice %}}

```bash
[{{< param "control_prompt" >}} ~]$ export TOWER_HOST=https://{{< param "external_tower" >}}
[{{< param "control_prompt" >}} ~]$ export TOWER_USERNAME=admin
[{{< param "control_prompt" >}} ~]$ export TOWER_PASSWORD='{{< param "secret_password" >}}'
[{{< param "control_prompt" >}} ~]$ export TOWER_VERIFY_SSL=false
```

{{% notice tip %}}
Feel free to write this into a new file using the code-server editor and then to use **source \<filename\>** to set the environment variables. This way if you loose connection to code-server you can easily re-set the vars.
{{% /notice %}}

Then use **awx** to login and print out the access token and to save it to a file at the same time:

```bash
[{{< param "control_prompt" >}} ~]$ awx login -f human | tee token
```

{{% notice tip %}}
We are saving the **export TOWER_OAUTH_TOKEN=\<YOUR_TOKEN\>** command line output to the file **token** using **tee** here to be able to set the environment variable more easily.
{{% /notice %}}

Finally set the environment variable with the token using the line the command printed out:

```bash
[{{< param "control_prompt" >}} ~]$ source token
```

Now that the access token is available in your shell, test **awx** is working. First run it without arguments to get a
list of resources you can manage with it:

    [{{< param "control_prompt" >}} ~]$ awx --help

And then test something, e.g. (leave out **-f human** if you're a JSON fan...) ;)

```bash
[{{< param "control_prompt" >}} ~]$ awx -f human user list
```

{{% notice tip %}}
When trying to find a **awx** command line for something you want to do, just move one by one.
{{% /notice %}}

Example: Need to create an inventory...

    [{{< param "control_prompt" >}} ~]$ awx --help

Okay, there is an **inventory** resource. Let’s see…

    [{{< param "control_prompt" >}} ~]$ awx inventory

Well, the **create** action sounds like what I had in mind. But what arguments do I
need? Just run:

    [{{< param "control_prompt" >}} ~]$ awx inventory create

And you'll get the required and optional arguments for the **create** action!

## Challenge Lab: awx

To practice your **awx** skills, here is a challenge:

- Try to change the **idle time out** of the Tower web UI, it’s 1800 seconds by default. Set it to, say, 7200. Using **awx**, of course.

- Start by looking for a resource type **awx** provides using **--help** that sounds like it has something to do with changing settings.

- Look at the available **awx** commands for this resource type.

- Use the commands to have a look at the parameters settings and change it.

{{% notice tip %}}
The configuration parameter is called **SESSION\_COOKIE\_AGE**
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```bash
[{{< param "control_prompt" >}} ~]$ awx setting list | grep SESSION
[{{< param "control_prompt" >}} ~]$ awx setting modify SESSION_COOKIE_AGE 7200
[{{< param "control_prompt" >}} ~]$ awx setting list | grep SESSION
```

</p>
<hr/>
</details>

If you want to, go to the web UI of any node (not just the one you connected **awx** to) and check the setting under **ADMINISTRATION→Settings→System**.
