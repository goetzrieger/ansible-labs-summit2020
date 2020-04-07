# Exercise 7 - Start Parallel Jobs across Instances

The real power of instance groups is revealed when multiple jobs are
started, and they are assigned to different Tower nodes. To launch
parallel jobs we will set up a workflow with multiple concurrent jobs.

## Lab Scenario

During this lab we’ll focus on security compliance according to STIG,
CIS and so on. Often these compliance rules are enforced by executing an
Ansible task per each requirement. This makes documentation and audit
easier.

Compliance requirements are often grouped into independent categories.
The tasks can often be executed in parallel because they do not conflict
with each other.

In our demo case we use three playbooks which:

  - ensure the absence of a few packages (STIG)

  - ensure configuration of PAM and login cryptography (STIG)

  - ensure absence of services and kernel modules (CIS).

The Playbooks can be found in the Github repository you already setup.

## Prepare the Compliance Lab

### First Step: Create three Templates

As mentioned the Github repository contains three Playbooks to enforce
different compliance requirements. First create these three templates
via `awx`:

    [root@ansible ~]# awx job_template create --name "Compliance STIG packages" \
                          --job-type run \
                          --inventory "Example Inventory" \
                          --project "Apache"  \
                          --playbook "stig-packages.yml" \
                          --become_enabled 1 


    [root@ansible ~]# awx job_template create --name "Compliance STIG config" \
                          --job_type run \
                          --inventory "Example Inventory" \
                          --project "Apache" \
                          --playbook "stig-config.yml" \
                          --become_enabled 1

    [root@ansible ~]# awx job_template create --name "Compliance CIS" \
                          --job-type run  \
                          --inventory "Example Inventory" \
                          --project "Apache" \
                          --playbook "cis.yml" \
                          --become_enabled 1


## Create Parallel Workflow

To enable parallel execution of the tasks in these job templates, we
will create a workflow. We’ll use the web UI because using **awx**
for this is a bit too involved for a lab. Workflows are configured in
the **Templates** view, you might have noticed you can choose between
**Job Template** and **Workflow Template** when adding a template.

  - Go to the **Templates** view and click the
    ![plus](../../images/green_plus.png) button. This time choose **Workflow
    Template**

      - **NAME:** Compliance Workflow

      - **ORGANIZATION:** Default

  - Click **SAVE**

  - Now the **WORKFLOW VISUALIZER** button becomes active and the
    graphical workflow designer opens.

  - Click on the **START** button, a new node opens. To the right you
    can assign an action to the node, you can choose between **JOBS**,
    **PROJECT SYNC** and **INVENTORY SYNC**.

  - In this lab we’ll link multiple jobs to the **START**, so select the
    **Compliance STIG packages** job and click **SELECT**. The node gets
    annotated with the name of the job.

  - Click on the **START** button again, another new node opens.

  - Select the **Compliance STIG config** job and click **SELECT**. The
    node gets annotated with the name of the job.

  - Click on the **START** button again, another new node opens.

  - Select the **Compliance CIS** job and click **SELECT**. The node
    gets annotated with the name of the job.

  - Click **SAVE**

  - In the workflow overview window, again click **SAVE**

You have configured a Workflow that is not going through templates one
after the other but rather executes three templates in parallel.

## Execute and Watch

Your workflow is ready to go, launch it.

  - In the **Templates** view launch the **Compliance Workflow** by
    clicking the rocket icon.

  - Wait until the job has finished.

Go to the **Instance Groups** view and find out how the jobs where
distributed over the instances:

  - Open the **INSTANCES** view of the tower instance group.

  - Look at the **TOTAL JOBS** view of the three instances

  - Because the Job Templates called in the workflow didn’t specify an
    instance group, they where distributed evenly over the instances.

Now deactivate instance **tower1.ewl05.internal** with the
![on/off](../../images/on_off.png) button and wait until it is shown as
deactivated. Make a (mental) note of the **TOTAL JOBS** counter of the
instance. Go back to the list of templates and launch the workflow
**Compliance Workflow** again.

Go back to the **Instance Groups** view, get back to the instance
overview of instance group **tower** and verify that the three Playbooks
where launched on the remaining instances and the **TOTAL JOBS** counter
of instance **tower1.ewl05.internal** didn’t change.

Activate **tower1.example.com** again by pressing
![on/off](../../images/on_off.png) a second time.

## Using Instance Groups

So we have seen how a Tower cluster is distributing jobs over Tower
instances by default. We have already created instance groups which
allow us to take control over what job is executed on which node, so
let’s use them.

To make it easier to spot where the jobs were run, let’s first empty the
jobs history. This can be done using **awx-manage** on one of the Tower
instances. From your control node SSH into one of the Tower hosts and as
`root` run the command:

    [root@tower1 ~]# awx-manage cleanup_jobs  --days=0

### Assign Jobs to Instance Groups

One way to assign a job to an instance group is in the job template. As
our compliance workflow uses three job templates, do this for all of
them:

  - In the web UI, go to **RESOURCES→Templates**

  - Open one of the three compliance templates

  - In the **Instance Groups** field, choose the **dev** instance group
    and click **SAVE**.

  - Click **SAVE** for the template and do this for the other two
    compliance templates, too.

Now the jobs that make up our **Compliance Workflow** are all configured
to run on the instances of the **dev** instance group.

### Run the Workflow

You have done this a couple of times now, you should get along without
detailed instructions.

  - Run the **Compliance Workflow**

  - What would you expect? On what instance(s) should the workflow jobs
    run?

  - Verify\!

> **Tip**
>
> **Result:** The workflow and the associated jobs will run on
> **tower2.ewl05.internal**. Okay, big surprise, in the **dev** instance
> group is only one instance.

But what’s going to happen if you disable this instance?

  - Disable the **tower2.ewl05.internal** instance in the **Instance
    Groups** view.

  - Run the workflow again.

  - What would you expect? On what instance(s) should the workflow jobs
    run?

  - Verify\!

> **Tip**
>
> **Result:** The workflow and the associated jobs will stay in pending
> state because there are no instance available in the **dev** instance
> group.

What’s going to happen if you enable the instance again?

  - Go to the **Instance Groups** view and enable
    **tower2.ewl05.internal** again.

  - Check in the **Jobs** and **Instance Groups** view what’s happening.

> **Tip**
>
> **Result:** After the instance is enabled again the jobs will pickup
> and run on **tower2.ewl05.internal**.

> **Warning**
>
> At this point make sure the instances you disabled in the previous
> steps are definitely enabled again\! Otherwise subsequent steps might
> fail…
