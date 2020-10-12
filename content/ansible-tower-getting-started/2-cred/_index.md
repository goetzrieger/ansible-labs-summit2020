+++
title = "Inventories, credentials and ad hoc commands"
weight = 2
+++

## Create an Inventory

Let’s get started: The first thing we need is an inventory of your managed hosts. This is the equivalent of an inventory file in Ansible Engine. There is a lot more to it (like dynamic inventories) but let’s start with the basics.

- You should already have the web UI open, if not: Point your browser to the URL you were given, similar to **`https://{{< param "external_tower1" >}}`** (replace "\<N\>" and "\<LABID\>") and log in as `admin` with the password given on the lab landing page.

Create the inventory:

- In the web UI menu on the left side, go to **RESOURCES** → **Inventories**, click the ![plus](../../images/green_plus.png?classes=inline) button on the right side and choose **Inventory**.

- **NAME:** Workshop Inventory

- **ORGANIZATION:** Default

- Click **SAVE**

Go back to the **Inventories** list, your new **Workshop Inventory** should show up. Open the **Workshop Inventory** and click the **HOSTS** button, the list will be empty since we have not added any hosts yet.

So let's add some hosts. As mentioned in the intro you have three managed hosts in your lab environment, the names are resolved through the `/etc/hosts` file. The nodes are named `node1`, `node2` and `node3`.

Now add the hosts to the inventory in Tower:

- In the inventory view of the Tower web UI click on your **Workshop Inventory**

- Click on  the **HOSTS** button (the one at the top, _not_ the bottom one, which is read-only and shows all hosts in _all_ inventories)

- To the right click the ![plus](../../images/green_plus.png?classes=inline) button.

- **HOST NAME:** node1

- Click **SAVE**

- Go back to **HOSTS** or use the frame below the new node frame in the UI and repeat to add **node2** as a second and **node3** as a third node.

You have now created an inventory with three managed hosts.

## Machine Credentials

One of the great features of Ansible Tower is to make credentials usable to users without making them visible. To allow Tower to execute jobs on remote hosts, you must configure connection credentials.

{{% notice tip %}}
This is one of the most important features of Tower: **Credential Separation**\! Credentials are defined separately and not with the hosts or inventory settings.
{{% /notice %}}

As this is an important part of your Tower setup, why not make sure that connecting to the managed nodes from Tower is working in the first place?

To test access to the nodes via SSH do the following:

- In your browser bring up the terminal window in code-server (remember this runs on the Tower node)

- From here as user `ec2-user` SSH into `node1` or one of the other nodes and execute `sudo -i`.

- For the SSH connection use the node password from the inventory file, `sudo -i` works without password.

```bash
[{{< param "internal_control" >}} ~]$ ssh ec2-user@node1
[ec2-user@node1 ~]$
sudo -i
[root@node1 ~]# exit
[ec2-user@node1 ~]$ exit
```

What does this mean?

- Tower user **student{{< param "student" >}}** can connect to the managed hosts with SSH key authentication as user **ec2-user**.

- User **ec2-user** can execute commands on the managed hosts as **root** with `sudo`.

## Configure Machine Credentials

Now we will configure the credentials to access our managed hosts from Tower. In the **RESOURCES** menu choose **Credentials**. Now:

Click the ![plus](../../images/green_plus.png?classes=inline) button to add new credentials

- **NAME:** Workshop Credentials

- **ORGANIZATION:** Click on the magnifying glass, pick **Default** and click **SELECT**

- **CREDENTIAL TYPE:** Click on the magnifying glass, pick **Machine** as type and click **SELECT** (you will have to use the search or cycle through the types to find it).

- **USERNAME:** ec2-user

- **PRIVILEGE ESCALATION METHOD:** sudo

{{% notice tip %}}
Whenever you see a magnifying glass icon next to an input field, clicking it will open a list to choose from.
{{% /notice %}}

As we are using SSH key authentication, you have to provide an SSH private key that can be used to access the hosts. You could also configure password authentication here.

Bring up your code-server terminal on Tower, and `cat` the SSH private key:

```bash
[{{< param "internal_control" >}} ~]$ cat .ssh/aws-private.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA2nnL3m5sKvoSy37OZ8DQCTjTIPVmCJt/M02KgDt53+baYAFu1TIkC3Yk+HK1
[...]
-----END RSA PRIVATE KEY-----
```

- Copy the **complete private key** (including "BEGIN" and "END" lines) and paste it into the **SSH PRIVATE KEY** field in the web UI.

- Click **SAVE**

Go back to the **RESOURCES -> Credentials -> Workshop Credentials** and note
that the SSH key is not visible.

You have now setup credentials for Ansible to access your managed hosts.

## Run Ad Hoc Commands

As you’ve probably done with Ansible before you can run ad hoc commands from Tower as well.

- In the web UI go to **RESOURCES → Inventories → Workshop Inventory**

- Click the **HOSTS** button to change into the hosts view and select the three hosts by checking the boxes to the left of the host entries.

- Click **RUN COMMANDS**. In the next screen you have to specify the ad hoc command:

  - As **MODULE** choose **ping**

  - For **MACHINE CREDENTIAL** click the magnifying glass icon and select **Workshop Credentials**.

  - Click **LAUNCH**, and watch the output.

The simple **ping** module doesn’t need options. For other modules you need to supply the command to run as an argument. Try the **command** module to find the user ID of the executing user using an ad hoc command.

- **MODULE:** command

- **ARGUMENTS:** id

<!-- {{% notice tip %}}
After choosing the module to run, Tower will provide a link to the docs page for the module when clicking the question mark next to "Arguments". This is handy, give it a try.
{{% /notice %}} -->

How about trying to get some more private information from the system? Try to print out **/etc/shadow**.

- **MODULE:** command

- **ARGUMENTS:** cat /etc/shadow

{{% notice warning %}}
Expect an error!
{{% /notice %}}

Oops, the last one didn’t went well, all red.

Re-run the last ad hoc command but this time tick the **ENABLE PRIVILEGE ESCALATION** box.

As you see, this time it worked. For tasks that have to run as root you need to escalate the privileges. This is the same as the **become: yes** you’ve probably used often in your Ansible Playbooks.

## Challenge Lab: Ad Hoc Commands

Okay, a small challenge: Run an ad hoc to make sure the package "tmux" is installed on all hosts. If unsure, consult the documentation either via the web UI as shown above or by running `ansible-doc yum` on your Tower control host.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

- **MODULE:** yum
- **ARGUMENTS:** name=tmux
- Tick **ENABLE PRIVILEGE ESCALATION**

</p>
<hr/>
</details>

{{% notice tip %}}
The yellow output of the command indicates Ansible has actually done something (here it needed to install the package). If you run the ad hoc command a second time, the output will be green and inform you that the package was already installed. So yellow in Ansible doesn’t mean "be careful"…​ ;-).
{{% /notice %}}

{{% notice tip %}}
Try to click one of the output lines in the window showing the job output. A small window with a lot of information about the job like module args will open. Have a look and close it again.
{{% /notice %}}
