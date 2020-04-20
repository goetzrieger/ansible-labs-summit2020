+++
title = "Introduction to Ansible Tower Clustering"
weight = 2
+++

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
three nodes web UIs as:

  - user **admin**

  - password **r3dh4t1\!**

> **Warning**
>
> Replace **\<LABID>** with the session ID and **\<N>** with your student number!

**`https://student<N>-ansible.<LABID>.rhdemo.io`**

**`https://student<N>-towernode2.<LABID>.rhdemo.io`**

**`https://student<N>-towernode3.<LABID>.rhdemo.io`**

Just from the web UI you wouldn’t know you’ve got a Tower cluster at
your hands here. To learn more about your cluster and its state, in one
of the instances web UI under **ADMINISTRATION** choose **Instance
Groups**. Here you will get an overview of the cluster by instance
groups. Explore the information provided, of course there is no capacity
used yet and no Jobs have run.

Right now we have only one instance group named **tower**. When you get more groups, from this view you will see how the instance are distributed over the groups.

To dig deeper click on **INSTANCES** to get more information about the instances allocated to a group. In the instances view you can toggle nodes off/online and adjust the number of forks (don't do this now).

> You’ll learn more about this later.

## Access your Tower Cluster via Commandline

You can also get information about your cluster on the command line. Log in to your **code-server** again if you closed it by opening this URL in your browser:

**`https://student<N>-code.<LABID>.rhdemo.io`**

Your VSCode session is running on your Tower node 1. Again if not still open, open a terminal by clicking **Terminal->New Terminal** in the menu bar.

A terminal window opens at the bottom, become root:

    [student1@ansible ~]$ sudo -i
    [root@ansible ~]#

In the terminal run the following command:

> Your student number and labid will differ, of course!

    [root@ansible ~]# awx-manage list_instances
    [tower capacity=51]
        student1-ansible.ad34.internal capacity=17 version=3.6.3 heartbeat="2020-04-08 14:10:27"
        student1-towernode2.ad34.internal capacity=17 version=3.6.3 heartbeat="2020-04-08 14:10:36"
        student1-towernode3.ad34.internal capacity=17 version=3.6.3 heartbeat="2020-04-08 14:10:38"

So what we’ve got is a three-node Tower cluster, no surprises here. In addition the command tells us the capacity (maximum number of forks/concurrent jobs) per node and for the instance groups. Here the capacity value of 17 is allocated to any of our three nodes.

> **Tip**
>
> The **awx-manage** (formerly tower-manage) utility can be used to
> administer a lot of the more internal aspects of Tower. You can e.g.
> use it to clean up old data, for token and session management and for
> cluster management.
