+++
title = "Ansible Tower Advanced"
weight = 1
+++

## About this Lab

You have already used Ansible Automation quite a bit and have started to
look into Tower? Or you are already using Tower? Cool. We prepared this
lab to give a hands-on introduction to some of the more advanced
features of Tower. You’ll learn about:

- Using command line tools to manage Ansible Tower

- Ansible Tower clustering

- Working with Instance Groups

- Using isolated nodes to manage remote locations

- Ways to provide inventories (importing inventory, dynamic inventory)

- The Smart Inventory feature

- Optional: How to structure Ansible content in Git repos

- Optional: How to work with the Tower API

## So little time and so much to do…

{{% notice warning %}}
To be honest we got carried away slightly while trying to press all these cool features into a two-hours lab session. We decided to flag the last two chapters as "optional" instead of taking them out. If you find the time to run them, cool\! If not, the lab guide will stay where it is, feel free to go through these lab tasks later (you don’t  need a Tower cluster for this).
{{% /notice %}}

## Want to Use this Lab after Summit?

Definitely, the Markdown sources are available here:

**[https://github.com/goetzrieger/ansible-labs-summit2020/tree/master/content/ansible-tower-advanced](https://github.com/goetzrieger/ansible-labs-summit2020/tree/master/content/ansible-tower-advanced)**

## Your Ansible Tower Lab Environment

In this lab you work in a pre-configured lab environment. You will have
access to the following hosts:

| Role                         | URL for External Access (if applicable)  | Hostname Internal                   |
| ---------------------------- | ---------------------------------- | ----------------------------------- |
| Ansible Tower Node 1         | {{< param "external_tower1" >}}    | {{< param "internal_tower1" >}}     |
| Ansible Tower Node 2         |                                    | {{< param "internal_tower2" >}}     |
| Ansible Tower Node 3         |                                    | {{< param "internal_tower3" >}}     |
| Visual Code Web UI           | {{< param "external_code" >}}      |                                     |
| Ansible Tower Database Host  |                                    | {{< param "internal_towerdb" >}}    |
| Managed RHEL8 Host 1         |                                    | {{< param "internal_host1" >}}      |
| Managed RHEL8 Host 2         |                                    | {{< param "internal_host2" >}}      |
| Ansible Tower Isolated Node  |                                    | {{< param "internal_toweriso" >}}   |
| Managed Remote Host 1        |                                    | {{< param "internal_hostremote" >}} |

{{% notice tip %}}
The lab environments in this session have a **{{< param "labid" >}}** and are separated by numbered **student{{< param "student" >}}** accounts. You will be able to access the hosts using the external hostnames. Internally the hosts have different names as shown above. Follow the instructions given by the lab facilitators to receive the values for **student\<N>** and **\<LABID>**!
{{% /notice %}}

{{% notice tip %}}
Ansible Tower has already been installed and licensed for you, the web UI will be reachable over HTTP/HTTPS.
{{% /notice %}}

{{% notice info %}}
Wherever you see the placeholder **{{< param "secret_password" >}}** in the following pages, use instead the specific password provided to you on the lab page. In general, whenever you need a password, even without the placeholder explicitly written, it's the same one.
{{% /notice %}}

As you can see the lab environment is pretty extensive. You basically
have:

- A three-node Tower cluster with a separate DB host, accessed via SSH or web UI

- Two managed RHEL 8 hosts

And to mimic a remote site with isolated nodes:

- One RHEL 8 host that acts as an isolated Tower node that can be reached via SSH from the Tower cluster nodes.

- One RHEL 8 host which acts as a remote managed node that can only be reached from/through the isolated node.

<!-- A diagram says more then a thousand words:

![tower diagram.png](../../images/tower_diagram.png) -->

{{% notice tip %}}
Access to the isolated node and the managed hosts is actually not restricted in the lab environment. Just imagine filtered, DMZ-like access rules for educational purposes… ;-)
{{% /notice %}}

## Working the Lab

Some hints to get you started:

- Don’t type everything manually, use copy & paste from the browser when appropriate. But don’t stop to think and understand… ;-)

- To **edit files** or **open a terminal window**, we provide **code-server**, basically the great VSCode Editor running in your browser. It's running on the first Tower node and can be accessed through the URL **https://{{< param "external_code" >}}**

{{% notice tip %}}
Commands you are supposed to run are shown with or without the expected output, whatever makes more sense in the context.
{{% /notice %}}

{{% notice tip %}}
The command line can wrap on the HTML page from time to time. Therefore the output is often separated from the command line for better readability by an empty line. **Anyway, the line you should actually run should be recognizable by the prompt.** :-)
{{% /notice %}}

## Accessing your Lab Environment

You'll get the access information for your lab (URL's, password) from a landing page. Getting access to this page depends on how you are consuming the lab:

- If you deployed from RHPDS, you'll receive an email with the landing page URL

- If you attend this lab at an event, your lab facilitator will lead you to the landing page

Either way you'll get an URL similar to this: `http://{{< param "external_domain" >}}`

Your main points of contact with the lab are the Ansible Tower web UI's and **code-server**, providing a VSCode-experience in your browser. You'll use **code-server** to:

- open virtual terminals

- edit files

Now open code-server using the link from the lab landing page or this link in your browser by replacing **\<N\>** by your student number and the **\<LABID\>**:

```bash
     https://{{< param "external_code" >}}
```

![code-server login](../../images/vscode-pwd.png)

Use the password provided on the landing page to login into the code-server web UI, you can close the **Welcome** tab. Now open a new terminal by heading to the menu item **Terminal** at the top of the page and select **New Terminal**. A new section will appear in the lower half of the screen and you will be greeted with a prompt:

![code-server terminal](../../images/vscode-terminal.png)

If unsure about the usage, read the [Visual Studio Code Server introduction](../../vscode-intro/), to learn more about how to create and edit files, and to work with the Terminal.

Congrats, you now have a shell terminal on your Ansible Tower node 1. From here you run commands or access the other hosts in your lab environment if the lab task requires it.
