# Exercise 2 - Introduction to Ansible Tower Clustering

With version 3.1 Ansible Tower introduced clustering, replacing the
redundancy solution configured with active-passive nodes. Clustering
is sharing load between Tower nodes/instances. Each Tower instance is
able to act as an entry point for UI and API access.

> **Tip**
>
> Using a load balancer in front of the Tower nodes is
> possible, but optional because an Ansible Tower cluster can be
> accessed via all Tower instances.

Each instance in a Tower cluster expands the cluster’s capacity to
execute jobs. Jobs can and will run anywhere rather than be directed on
where to run.

> **Tip**
>
> The Appendix contains some installation considerations and an
> installer inventory for reference.

## Access the Tower Web UI

For the first contact to your cluster open your browser and login to all
three nodes web UIs (you’ll have to accept the self-signed certificates)
as:

  - user **admin**

  - password **r3dh4t1\!**

> **Warning**
>
> Replace the **&lt;GUID&gt;** string with your GUID\!

  - **https://student\<N>.ansible.\<LABID>.rhdemo.io**

  - **https://student\<N>.towernode2.\<LABID>.rhdemo.io**

  - **https://student\<N>.towernode3.\<LABID>.rhdemo.io**

Just from the web UI you wouldn’t know you’ve got a Tower cluster at
your hands here. To learn more about your cluster and its state, in one
of the instances web UI under **ADMINISTRATION** choose **Instance
Groups**. Here you will get an overview of the cluster by instance
groups. Explore the information provided, of course there is no capacity
used yet and no Jobs have run.

Right now we have only one instance group named **tower**. From this view you can also see how the instance are distributed over the groups.

To dig deeper click on **INSTANCES** to get more
information about the instances allocated to a group. In the instances
view you can toggle nodes off/online and adjust the number of forks.
You’ll learn more about this later.

## Access you Tower Cluster via Commandline

You can also get information about your cluster on the command line. As mentioned the first Tower node runs **code-server** to let you access an editor and a terminal in your browser. Log in to **code-server** by opening this URL in your browser (replace \<LABID> and the student number \<N> with your values), the password is **r3dh4t1!**:

    https://student<N>-code.<LABID>.rdemo.io

This will give you a VSCode session running in your Tower node 1. Open a terminal by clicking **Terminal->New Terminal** in the menu bar.

A terminal window opens at the bottom, become root:

    [student1@ansible ~]$ sudo -i
    [root@ansible ~]#

In the terminal run the following command:

    [root@ansible ~]# awx-manage list_instances
    [tower capacity=51]
        student1-ansible.89cd.ansibleworkshops.com capacity=17 version=3.6.3 heartbeat="2020-03-24 14:31:36"
        student1-towernode2.89cd.ansibleworkshops.com capacity=17 version=3.6.3 heartbeat="2020-03-24 14:31:07"
        student1-towernode3.89cd.ansibleworkshops.com capacity=17 version=3.6.3 heartbeat="2020-03-24 14:31:08"

So what we’ve got is a three-node Tower cluster, no surprises here. In
addition the command tells us the capacity (maximum number of
forks/concurrent jobs) per node and for the instance groups. Here the
capacity value of 51 is allocated to any of our three nodes.

> **Tip**
>
> The **awx-manage** (formerly tower-manage) utility can be used to
> administer a lot of the more internal aspects of Tower. You can e.g.
> use it to clean up old data, for token and session management and for
> cluster management.
