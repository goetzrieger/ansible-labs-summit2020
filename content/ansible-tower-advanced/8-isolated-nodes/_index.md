+++
title = "Isolated Nodes"
weight = 8
+++

Ansible is used to manage complex infrastructures with machines and networks living in multiple separate datacenters, servers behind firewalls or in cloud VPCs and remote locations only reachable over unstable links which may not survive the length of a job run. In cases like these it’s often better to run automation local to the nodes.

To solve this, Tower provides **Isolated Nodes**:

  - Isolated Nodes **don’t have a full installation of Tower**, but a minimal set of utilities used to run jobs.

  - Isolated Nodes can be deployed behind a firewall/VPC or in a remote datacenter, only **ingress SSH traffic** from a **controller** instance to the **isolated** instances is required.

  - When a job is run that targets things managed by an isolated node, the **job** and its **environment** will be **pushed to the isolated node** over SSH

  - Periodically, the **master Ansible Tower cluster will poll the isolated node** for status on the job.

  - When the **job finishes**, the job status will be **updated in Ansible Tower**

## Setting Up Isolated Nodes

Isolated nodes are defined in the installer inventory file and setup by the Ansible Tower installer. Isolated nodes make up their own instance groups that are specified in the inventory file prefixed with **isolated\_group\_**. In the isolated instance group model, **controller** instances interact with **isolated** instances via a series of Ansible playbooks over SSH.

**So for the fun of it, let’s set one up.**

First have a look at the Tower installer inventory file that was used for lab setup. In your VSCode terminal on your Tower node 1 change into the Ansible installer directory and do the following:

    [student@ansible ~]$ cd /tmp/tower_install/
    [student@ansible tower_install]$ cat inventory

    [tower]
    ansible ansible_host=student<X>-ansible.<LABID>.internal
    towernode2 ansible_host=student<X>-towernode2.<LABID>.internal
    towernode3 ansible_host=student<X>-towernode3.<LABID>.internal

    [database]
    towerdb ansible_host=student<X>-towerdb.<LABID>.internal

    [...]

{{% notice tip %}}
The inventory has been adapted for readability by leaving out connection variables.
{{% /notice %}}

You can see we have the tower base group and one for the database node. For the isolated node we will define a new **isolated_group_** named **dmz** with one entirely new node, called **student\<N>-isonode.\<LABID>.internal** which we’ll use to manage other hosts in the remote location.

{{% notice warning %}}
The Ansible installer files in `/tmp/tower_install/` are owned by root, but your code-server/VSCode instance is running as your student\<N> user. To be able to edit the inventory file, you have to change the file permissions.
{{% /notice %}}

To edit the inventory file in VSCode editor change the permissions (don't do 777 in real life... ;)):

    [student@ansible ~]$ sudo -i
    [root@ansible ~]# chmod 777 /tmp/tower_install/inventory

Then do **File -> Open File** in VSCode, navigate to `/tmp/tower_install/inventory` file and open it. Add the isolated node to the inventory to look like this:

    [tower]
    ansible ansible_host=student<X>-ansible.<LABID>.internal
    towernode2 ansible_host=student<X>-towernode2.<LABID>.internal
    towernode3 ansible_host=student<X>-towernode3.<LABID>.internal

    [isolated_group_dmz]
    isonode ansible_host=student<X>-isonode.<LABID>.internal ansible_user="student1" ansible_password='r3dh4t1!' ansible_become=true

    [isolated_group_dmz:vars]
    controller=tower

    [database]
    towerdb ansible_host=student<X>-towerdb.<LABID>.internal

    [...]

{{% notice warning %}}
Only add the isolated_group settings, don't change the other groups and settings!
{{% /notice %}}

{{% notice tip %}}
Each isolated group must have a controller variable set. This variable points to the instance group that manages tasks that are sent to the isolated node. That instance group will be responsible for starting and monitoring jobs on the isolated node. In this case, we’re using the main tower instance group to manage this isolated group.
{{% /notice %}}

After editing the inventory, start the installer in the VSCode terminal to make the desired changes:

    [student@ansible ~]$ sudo -i
    [root@ansible ~]# cd /tmp/tower_install/
    [root@ansible tower_install]# ./setup.sh

## Verify Isolated Nodes

After the installer has finished isolated groups can be listed in the same way like instance groups and Ansible Tower cluster configuration. So the methods listed above discussing instance groups also apply to isolated nodes. For example, using `awx`:

    [student1@ansible ~]$ awx -f human instance_group list
    id name
    == =====
    1  tower
    2  dev
    3  prod
    4  dmz

You can see your **dmz** isolated group has been setup. Like other instance groups, isolated node groups can be assigned at the level of an organization, an inventory, or an individual job template.

## Create Isolated Node specific Inventory

To actually do something in the remote location served by the isolated node we need a managed host. Let’s assume we have a setup with one host in our DMZ, and we want to manage it siloed off from the rest of the infrastructure. The isolated node we configured above is located in the same location and is able to connect to the managed host(s).

For this you have to create a new inventory in your Tower cluster. You can do this with `awx` like we did in the beginning, or you use the web UI. Why not use the web UI for a change?

In the Tower web UI under **RESOURCES**, click **Inventories**:

  - Click the ![plus](../../images/green_plus.png?classes=inline) button to add a new
    inventory

  - **NAME:** Remote Inventory

  - **ORGANIZATION:** Default

  - **INSTANCE GROUPS:** Pick the instance group you created in the last
    step, `dmz`

  - Click **SAVE**

Now you can add managed hosts,the **HOSTS** button is active now. Click it to access the hosts overview. There are no hosts right now, so let’s add one:

  - Click the ![plus](../../images/green_plus.png?classes=inline) button to add a new host

  - **NAME:** `student<X>-remote.<LABID>.internal`

  - Click **SAVE**

## Create Template for Isolated Node

Next we need to assign a job template to the node. Since the node is in a DMZ, we certainly have to ensure their compliance. Thus we are going to make sure that they are following our CIS guidelines - and will set up a template executing the CIS playbook on them.

Go to **Templates** in the **RESOURCES** section of the menu, click the ![plus](../../images/green_plus.png?classes=inline) button and choose **Job Template**.

  - **NAME:** Remote CIS Compliance

  - **JOB TYPE:** Run

  - **INVENTORY:** Remote Inventory

  - **PROJECT:** Apache

  - **PLAYBOOK:** `cis.yml`

  - **CREDENTIAL:** Example Credentials

  - **INSTANCE GROUPS:** `dmz`

  - We need to run the tasks as root so check **Enable privilege escalation**

  - Click **SAVE**

Next, launch the template:

  - In the **Templates** view launch the **Remote CIS Compliance** job by clicking the rocket icon.

  - Wait until the job has finished.

## Verify Results

Last but not least, let’s check that the job was indeed executed by the
isolated node `student<X>-isonode.<LABID>.internal`:

  - Go to **Instance Groups** in the **ADMINISTRATION** section of the
    web UI

  - Click on the **dmz** group.

  - Click on the jobs button at the top to see the executed job.
