# Exercise 2 - Introduction to Ansible Tower Clustering

With version 3.1 Ansible Tower introduced clustering, replacing the
redundancy solution configured with the active-passive nodes. Clustering
is sharing load between Tower nodes/instances. Each Tower instance is
able to act as an entry point for UI and API access.

> **Tip**
>
> Using a load balancer in front of the Tower nodes like in this lab is
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

  - **https://tower1.&lt;GUID&gt;.sandbox951.opentlc.com**

  - **https://tower2.&lt;GUID&gt;.sandbox951.opentlc.com**

  - **https://tower3.&lt;GUID&gt;.sandbox951.opentlc.com**

Just from the web UI you wouldn’t know you’ve got a Tower cluster at
your hands here. To learn more about your cluster and it’s state, in one
of the instances web UI under **ADMINISTRATION** choose **Instance
Groups**. Here you will get an overview of the cluster by instance
groups. Explore the information provided, of course there is no capacity
used yet and no Jobs have run.

Right now we have three instance groups named **dev**, **prod** and
**tower**. In a freshly installed Tower install you would only find one,
**tower**. From this view you can also see how the instance are
distributed over the groups.

To dig deeper, for every group, click on **INSTANCES** to get more
information about the instances allocated to a group. In the instances
view you can toggle nodes off/online and adjust the number of forks.
You’ll learn more about this later.

## Access you Tower Cluster via SSH

You can also get information about your cluster on the command line.
Open a terminal window and start an SSH session to your control host
using the external hostname (replace the &lt;GUID&gt; string, key
authentication should work automatically):

    # ssh -i ~/.ssh/toweradvlab.pem ec2-user@bastion.<GUID>.sandbox951.opentlc.com

Then become root:

    [ec2-user@bastion 0 ~]$ sudo -i
    [root@bastion 0 ~]#

> **Tip**
>
> Remember the control host is named
> **bastion.&lt;GUID&gt;.sandbox951.opentlc.com** when accessed from the
> outside, inside the lab environment you use
> **bastion.&lt;GUID&gt;.internal**.

From your control host now jump to one of the Tower instances, e.g.:

    # ssh tower1.<GUID>.internal
    # sudo -i

> **Tip**
>
> SSH keys have been distributed for the root user already.

And run the following command:

    [root@tower1 ~]# awx-manage list_instances
    [tower capacity=51]
            tower1.<GUID>.internal capacity=17 version=3.6.0 heartbeat="2020-02-11 16:06:15"
            tower2.<GUID>.internal capacity=17 version=3.6.0 heartbeat="2020-02-11 16:06:15"
            tower3.<GUID>.internal capacity=17 version=3.6.0 heartbeat="2020-02-11 16:06:15"

    [emea capacity=17 controller=tower]
            worker1.emea.<GUID>.internal capacity=17 version=ansible-runner-1.4.4 last_isolated_check="2020-02-11 16:04:10" heartbeat="2020-02-11 16:04:12"

So what we’ve got is a three-node Tower cluster, no surprises here. In
addition the command tells us the capacity (maximum number of
forks/concurrent jobs) per node and for the instance groups. Here the
capacity value of 17 is allocated to any of our three nodes.

> **Tip**
>
> The **awx-manage** (formerly tower-manage) utility can be used to
> administer a lot of the more internal aspects of Tower. You can e.g.
> use it to clean up old data, for token and session management and for
> cluster management.
