# Exercise 8 - Isolated Nodes

Ansible is used to manage complex infrastructures with machines and
networks living in multiple separate datacenters, servers behind
firewalls or in cloud VPCs and remote locations only reachable over
unstable links which may not survive the length of a job run. In cases
like these it’s often better to run automation local to the nodes.

To solve this, Tower provides Isolated Nodes:

  - Isolated nodes **don’t have a full installation of Tower**, but a
    minimal set of utilities used to run jobs.

  - It can be deployed behind a firewall/VPC or in a remote datacenter,
    only **ingress SSH traffic** from a **controller** instance to the
    **isolated** instances is required.

  - When a job is run that targets things managed by the isolated node,
    the **job** and its **environment** will be **pushed to the isolated
    node** over SSH

  - Periodically, the **master Ansible Tower cluster will poll the
    isolated node** for status on the job.

  - When the **job finishes**, the job status will be **updated in
    Ansible Tower**

## Setting Up Isolated Nodes

Isolated nodes are defined in the inventory file (same as instance
groups) and setup by the Ansible Tower installer. Isolated nodes make up
their own instance groups that are specified in the inventory file
prefixed with **isolated\_group\_**. In the isolated instance group
model, only specific **controller** Tower instance groups interact with
**isolated** nodes.

For this lab an isolated node named **worker1.emea.ewl05.internal** in
an instance group **emea** has already been setup for you.

## Verify Isolated Nodes

Isolated groups can be listed in the same way like instance groups and
Ansible Tower cluster configuration. So the methods listed above
discussing instance groups also applies to isolated nodes. For example,
using `awx`:

    # awx instance_group list
    == ===== ======== =================
    id name  capacity consumed_capacity
    == ===== ======== =================
     1 tower       51                 0
     2 *emea*      17                 0
     3 dev         17                 0
     4 prod        17                 0
    == ===== ======== =================

Like other instance groups, isolated node groups can be assigned at the
level of an organization, an inventory, or an individual job template.

## Create Isolated Node specific Inventory

Let’s assume we have a setup with two hosts in another region, say EMEA,
and we want to manage these siloed off from the rest of the
infrastructure. The isolated node we configured above is located in the
same region and is able to connect to the managed hosts.

Now create a new inventory in your Tower cluster. You can do this with
`awx` like we did above, or you use the web UI. Why not use the web
UI for a change?

In the Tower web UI under **RESOURCES**, click **Inventories**:

  - Click the ![plus](../../images/green_plus.png) button to add a new
    inventory

  - **NAME:** Remote Inventory

  - **ORGANIZATION:** Default

  - **INSTANCE GROUPS:** Pick the instance group you created in the last
    step, `emea`

  - Click **SAVE**

After you’ve clicked **SAVE**, you can add hosts: the button for the
hosts management ![hosts](../../images/tower_hosts.png) is active now. Click on
it to access the hosts overview. There are no hosts right now, so let’s
add some:

  - Click the ![plus](../../images/green_plus.png) button to add a new host

  - **NAME:** `isosupport1.ewl05.internal`

  - Click **SAVE**

Repeat the steps for a second host called `isosupport2.ewl05.internal`.

## Create Template for Isolated Node

Next we need to assign a template to the nodes. Since those nodes are in
a DMZ, we certainly have to ensure their compliance. Thus we are going
to make sure that they are following our CIS guidelines - and will set
up a template executing the CIS playbook on them.

Go to **Templates** in the **RESOURCES** section of the menu, click the
![plus](../../images/green_plus.png) button and choose **Job Template**.

  - **NAME:** Remote CIS Compliance

  - **JOB TYPE:** Run

  - **INVENTORY:** Remote Inventory

  - **PROJECT:** Compliance Repository

  - **PLAYBOOK:** `cis.yml`

  - **CREDENTIAL:** Example Credentials

  - **INSTANCE GROUPS:** `emea`

  - We need to run the tasks as root so check **Enable privilege
    escalation**

  - Click **SAVE**

Next, launch the template:

  - In the **Templates** view launch the **Remote CIS Compliance** job
    by clicking the rocket icon.

  - Wait until the job is finished.

## Verify Results

Last but not least, let’s check that the job was indeed executed by the
isolated node `worker1.emea.ewl05.internal`:

  - Go to **Instance Groups** in the **ADMINISTRATION** section of the
    web UI

  - Click on the **emea** group.

  - Click on the jobs button at the top to see the executed job.
