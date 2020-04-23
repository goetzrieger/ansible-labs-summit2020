+++
title = "Introduction to Tower"
weight = 1
+++

## Why Ansible Tower?

Ansible Tower is a web-based UI that provides an enterprise solution for IT automation. It

  - has a user-friendly dashboard.

  - complements Ansible, adding automation, visual management, and monitoring capabilities.

  - provides user access control to administrators.

  - graphically manages or synchronizes inventories with a wide variety of sources.

  - has a RESTful API.

  - And much more...

## Your Ansible Tower Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                         | Inventory name |
| -----------------------------| ---------------|
| Ansible Control Host & Tower | ansible        |
| Managed Host 1               | node1          |
| Managed Host 2               | node2          |
| Managed Host 2               | node3          |

{{% notice tip %}}
The lab environments in this session have a **\<LABID>** and are separated by numbered **student\<N>** accounts. You only need to access your Tower node from the Internet.
{{% /notice %}}

{{% notice tip %}}
Ansible Tower has already been installed and licensed for you, the web UI will be reachable over HTTP/HTTPS.
{{% /notice %}}

## Working the Lab

Some hints to get you started:

  - Don’t type everything manually, use copy & paste from the browser when appropriate. But don’t stop to think and understand… ;-)

  - To **edit files** or **open a terminal window**, we provide **code-server**, basically the great VSCode Editor running in your browser. It's running on the Tower node and can be accessed through the URL **https://student\<N>-code.\<LABID>.events.opentlc.com**

{{% notice tip %}}
Commands you are supposed to run are shown with or without the expected output, whatever makes more sense in the context.
{{% /notice %}}

{{% notice tip %}}
The command line can wrap on the HTML page from time to time. Therefore the output is often separated from the command line for better readability by an empty line. **Anyway, the line you should actually run should be recognizable by the prompt.** :-)
{{% /notice %}}

## Accessing your Lab Environment

Your main points of contact with the lab is the Ansible Tower web UI and **code-server**, providing a VSCode-experience in your browser.

Now open code-server using this link in your browser by replacing **\<N\>** by your student number and the **\<LABID\>**:


     	https://student<N>-code.<LABID>.events.opentlc.com


![code-server login](../../images/vscode-pwd.png)

Use the provided password to login into the code-server web UI, you can close the **Welcome** tab. Now open a new terminal by heading to the menu item **Terminal** at the top of the page and select **New Terminal**. A new section will appear in the lower half of the screen and you will be greeted with a prompt:

![code-server terminal](../../images/vscode-terminal.png)

Read the [Visual Studio Code Server introduction](../../vscode-intro/), to learn more about how to create and edit files, and to work with the Terminal.

Congrats, you now have a shell terminal on your Ansible Tower. From here you run commands or access the other hosts in your lab environment if the lab task requires it.

## Dashboard

Let's have a first look at Tower: Point your browser to the URL you were given, similar to `https://student<N>.<LABID>.events.opentlc.com` (replace `<N>` with your student number and `<LABID>` with the ID of your lab) and log in as `admin`. The password will be provided by the instructor.

The web UI of Ansible Tower greets you with a dashboard with a graph showing:

  - recent job activity

  - the number of managed hosts

  - quick pointers to lists of hosts with problems.

The dashboard also displays real time data about the execution of tasks completed in playbooks.

![Ansible Tower Dashboard](../../images/dashboard.png)

## Concepts

Before we dive further into using Ansible Tower for your automation, you should get familiar with some concepts and naming conventions.

**Projects**

Projects are logical collections of Ansible playbooks in Ansible Tower. These playbooks either reside on the Ansible Tower instance, or in a source code version control system supported by Tower.

**Inventories**

An Inventory is a collection of hosts against which jobs may be launched, the same as an Ansible inventory file. Inventories are divided into groups and these groups contain the actual hosts. Groups may be populated manually, by entering host names into Tower, from one of Ansible Tower’s supported cloud providers or through dynamic inventory scripts.

**Credentials**

Credentials are utilized by Tower for authentication when launching Jobs against machines, synchronizing with inventory sources, and importing project content from a version control system. Credential configuration can be found in the Settings.

Tower credentials are imported and stored encrypted in Tower, and are not retrievable in plain text on the command line by any user. You can grant users and teams the ability to use these credentials, without actually exposing the credential to the user.

**Templates**

A job template is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many times. Job templates also encourage the reuse of Ansible playbook content and collaboration between teams. To execute a job, Tower requires that you first create a job template.

**Jobs**

A job is basically an instance of Tower launching an Ansible playbook against an inventory of hosts.
