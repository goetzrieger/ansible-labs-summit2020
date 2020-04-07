# Exercise 6 - Tower Instance Groups

Ansible Tower clustering allows you to easily add capacity to your Tower
infrastructure by adding instances. What it doesn’t allow is to dedicate
capacity or nodes to a purpose, be it a group of people, a department or
a location.

In a single-group Tower cluster where all instances are within the
`tower` group there is no way to influence what node will run a job, the
cluster will take care of scheduling Jobs as it sees fit.

To enable more control over what node is running a job, Tower 3.2 saw
the introduction of the instance groups feature. Instance groups allow
you to organize your cluster nodes into groups. In turn Jobs can be
assigned to Instance Groups by configuring the Groups in Organizations,
Inventories or Job Templates.

> **Tip**
>
> The order of priority is **Job Template &gt; Inventory &gt;
> Organization**. So Instance Groups configured in Job Templates take
> precedence over those configured in Inventories, which take precedence
> over Organizations

Some things to keep in mind about Instance Groups:

  - Nodes in an Instance Group share a job queue

  - You can have as many Instance Groups as you like as long as there is
    at least one node in the `tower` group

  - Nodes can be in one or more Instance Groups

  - Groups can not be named `instance_group_tower`\!

  - Tower instances can’t have the same name as a group

Instance Groups allows for some pretty cool setups, e.g. you could have
some nodes shared over the whole cluster (by putting them into all
groups) but then have other nodes that are dedicated to one group to
reserve some capacity.

> **Warning**
>
> The base `tower` group does house keeping like processing events from
> jobs for all groups so the node count of this group has to scale with
> your overall cluster load, even if these nodes are not used to run
> Jobs.

Talking about the `tower` group: As you have learned this group is
crucial for the operations of a Tower cluster. Apart from the house
keeping tasks, if a resource is not associated with an Instance Group,
one of the nodes from the `tower` group will run the Job. So if there
are no operational nodes in the base group, the cluster will not be able
to run Jobs.

> **Warning**
>
> It is important to have enough nodes in the `tower` group

> **Tip**
>
> Here is a great blog post going into Instance Groups with a lot more
> depth:
> <https://www.ansible.com/blog/ansible-tower-feature-spotlight-instance-groups-and-isolated-nodes>.

## Instance Group Setup

Having the introduction out of the way, let’s get back to our lab and
give Instance Groups a try.

In a basic cluster setup you would just have the `[tower]` base group.
So let’s go and setup two instance groups:

  - In the **Instance Groups** add a new group by clicking the green
    **+** icon and then **CREATE INSTANCE GROUP**

  - Name the new group **dev**

  - **SAVE**

  - Click the **INSTANCES** button and add node
    **tower2.ewl05.internal** again using the **+** icon

Do the same again for the new group **prod** with instance
**tower3.ewl05.internal**

Going back to the **Instance Groups** view you should now have the
following setup:

  - All instances are in the **tower** base group

  - Two more groups (**prod** and **dev**) with one instances each where
    setup

> **Tip**
>
> We’re using the internal names of the Tower nodes here.

> **Warning**
>
> This is not best practice, it’s just for the sake of this lab\! Any
> jobs that are launched targeting a group without active nodes will be
> stuck in a waiting state until instances become available. So
> one-instance groups are never a good idea.

## Verify Instance Groups

You can check your instance groups in a number of ways.

### Via cli

    [root@bastion 0 ~]#awx -k instance_group list -f human 
    id name  
    == ===== 
    1  tower 
    2  dev   
    3  prod 

### Via API

You can again query the API to get this information. Either use the
browser to access the URL
`https://tower.ewl05.sandbox309.opentlc.com/api/v2/instance_groups/` or
use curl to access the API from the command line:

> **Tip**
>
> The curl command has to be on one line.

    [root@ansible ~]# curl -s -k -u admin:r3dh4t1! https://tower2.example.com/api/v2/instance_groups/| python -m json.tool

### Via the Web UI

Open the URL
`https://tower1-<GUID>.rhpds.opentlc.com/#/instance_groups` in your
browser.

In the **INSTANCE GROUPS** overview all instance groups are listed with
details of the group itself like number of instances in the group,
running jobs and finished jobs. Like you’ve seen before for the
**tower** global group the current capacity of the instance groups is
shown in a live view, thus providing a quick insight if there are
capacity problems.

## Deactivating Tower Instances

While in the **INSTANCES GROUPS** overview click the **INSTANCES** link
for, say, the **dev** group. In the next view you’ll see a slide button
next to each Tower instance (only one in this case).

  - The button should be set to "ON" ![on/off](../../images/on_off.png).
    Clicking it would deactivate the corresponding instance and would
    prevent that further jobs are assigned to it.

  - Running jobs on an instance which is set to "OFF" are finished in a
    normal way.

  - Further to the right a slider can change the amount of forks
    scheduled on an instance. This way it is possible to influence in
    which ratio the jobs are assigned.
