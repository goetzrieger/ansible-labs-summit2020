+++
title = "Check the Prerequisites"
weight = 1
+++

## Your Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                 | Inventory name |
| ---------------------| ---------------|
| Ansible Control Host | ansible        |
| Managed Host 1       | node1          |
| Managed Host 2       | node2          |
| Managed Host 2       | node3          |

## Access the Environment

Your Ansible control host runs **code-server**, the server part of an VSCode-like editor running in your browser. It provides access to a virtual terminal/shell and file editing capabilities:

    https://student<X>-code.<LABID>.rhdemo.io

Use the above link in your browser by replacing **\<X\>** in student**\<X\>** by the student number and **\<LABID\>** by the workshop name provided to you.

![code-server login](../../images/vscode-pwd.png)

Use the provided password to login into the code-server and open a new terminal by heading to the menu item "Terminal" at the top of the page and select "New Terminal". A new section will appear in the lower half of the screen and you will be presented a prompt:

![code-server terminal](../../images/vscode-terminal.png)

Now become root:

    [student<X>@ansible ~]$ sudo -i

Most prerequisite tasks have already been done for you:

  - Ansible software is installed

  - `sudo` has been configured on the managed hosts to run commands that require root privileges.

Check Ansible has been installed correctly (your actual Ansible version might differ):

    [root@ansible ~]# ansible --version
    ansible 2.9.5
    [...]

{{% notice tip %}}
Ansible is keeping configuration management simple. Ansible requires no database or running daemons and can run easily on a laptop. On the managed hosts it needs no running agent.
{{% /notice %}}

Log out of the root account again:

    [root@ansible ~]# exit
    logout

{{% notice tip %}}
In all subsequent exercises you should work as the student\<X\> user on the control node if not explicitly told differently.
{{% /notice %}}

## Working the Labs

You might have guessed by now this lab is pretty commandline-centric…​ :-)

  - Don’t type everything manually, use copy & paste from the browser when appropriate. But do still take time to think and understand.

{{% notice tip %}}
In the lab guide commands you are supposed to run are shown with or without the expected output, whatever makes more sense in the context.
{{% /notice %}}

## Challenge Labs

You will soon discover that many chapters in this lab guide come with a "Challenge Lab" section. These labs are meant to give you a small task to solve using what you have learned so far. The solution of a challenge task is shown beneath the task in a fold-out.
