# Exercise 2.1 - Introduction to Tower

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

## Why Ansible Tower?

Ansible Tower is a web-based UI that provides an enterprise solution for IT automation. It

  - has a user-friendly dashboard

  - complements Ansible, adding automation, visual management, and monitoring capabilities.

  - provides user access control to administrators.

  - graphically manages or synchronizes inventories with a wide variety of sources.

  - has a RESTful API

  - And much more...

## Your Ansible Tower Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                         | Inventory name |
| -----------------------------| ---------------|
| Ansible Control Host & Tower | ansible        |
| Managed Host 1               | node1          |
| Managed Host 2               | node2          |
| Managed Host 2               | node3          |

The Ansible Tower provided in this lab is individually setup for you. Make sure to access the right machine whenever you work with it. Ansible Tower has already been installed and licensed for you, the web UI will be reachable over HTTP/HTTPS.

## Dashboard

Let's have a first look at the Tower: Point your browser to the URL you were given, similar to `https://student<X>.workshopname.rhdemo.io` (replace `<X>` with your student number and `workshopname` with the name of your current workshop) and log in as `admin`. The password will be provided by the instructor.

The web UI of Ansible Tower greets you with a dashboard with a graph showing:

  - recent job activity

  - the number of managed hosts

  - quick pointers to lists of hosts with problems.

The dashboard also displays real time data about the execution of tasks completed in playbooks.

![Ansible Tower Dashboard](../images/ydashboard.png)

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

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
# Exercise 2.2 - Inventories, credentials and ad hoc commands

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

## Create an Inventory

Let’s get started with: The first thing we need is an inventory of your managed hosts. This is the equivalent of an inventory file in Ansible Engine. There is a lot more to it (like dynamic inventories) but let’s start with the basics.

  - You should already have the web UI open, if not: Point your browser to the URL you were given, similar to **https://student\<X\>.workshopname.rhdemo.io** (replace "\<X\>" with your student number and "workshopname" with the name of your current workshop) and log in as `admin`. The password will be provided by the instructor.

Create the inventory:

  - In the web UI menu on the left side, go to **RESOURCES** → **Inventories**, click the ![plus](../images/ygreen_plus.png) button on the right side and choose **Inventory**.

  - **NAME:** Workshop Inventory

  - **ORGANIZATION:** Default

  - Click **SAVE**

Now there will be two inventories, the **Demo Inventory** and the **Workshop Inventory**. In the **Workshop Inventory** click the **Hosts** button, it will be empty since we have not added any hosts there.

So let's add some hosts. First we need to have the list of all hosts which are accessible to you within this lab. These can be found in an inventory on the ansible control node on which Tower is installed. You'll find the password for the SSH connection there as well.

Login to your Tower control host via SSH:

> **Warning**
>
> Replace **workshopname** by the workshop name provided to you, and the **X** in student**X** by the student number provided to you.

```bash
ssh student<X>@student<X>.workshopname.rhdemo.io
```

You can find the inventory information at `~/lab_inventory/hosts`. Output them with `cat`, they should look like:

```bash
$ cat ~/lab_inventory/hosts
[all:vars]
ansible_user=student<X>
ansible_ssh_pass=PASSWORD
ansible_port=22

[web]
node1 ansible_host=22.33.44.55
node2 ansible_host=33.44.55.66
node3 ansible_host=44.55.66.77

[control]
ansible ansible_host=11.22.33.44
```
> **Warning**
>
> In your inventory the IP addresses will be different.

Note the names for the nodes and the IP addresses, we will use them to fill the inventory in Tower now:

  - In the inventory view of Tower click on your **Workshop Inventory**

  - Click on  the **HOSTS** button

  - To the right click the ![plus](../images/ygreen_plus.png) button.

  - **HOST NAME:** `node1`

  - **Variables:** Under the three dashes `---`, enter `ansible_host: 22.33.44.55` in a new line. Make sure to enter your specific IP address for your `node1` from the inventory looked up above, and note that the variable definition has a colon **:** and a space between the values, not an equal sign **=** like in the inventory file.

  - Click **SAVE**

  - Go back to **HOSTS** and repeat to add `node2` as a second host and `node3` as a third node. Make sure that for each node you enter the right IP addresses.

You have now created an inventory with three managed hosts.

## Machine Credentials

One of the great features of Ansible Tower is to make credentials usable to users without making them visible. To allow Tower to execute jobs on remote hosts, you must configure connection credentials.

> **Note**
>
> This is one of the most important features of Tower: **Credential Separation**\! Credentials are defined separately and not with the hosts or inventory settings.

As this is an important part of your Tower setup, why not make sure that connecting to the managed nodes from Tower is working?

 To access the Tower host via SSH do the following:

- Login to your Tower control host via SSH: `ssh student<X>@student<X>.workshopname.rhdemo.io`
- Replace **workshopname** by the workshop name provided to you, and the `<X>` in `student<X>` by the student number provided to you.
- From Tower SSH into `node1` or one of the other nodes (look up the IP addresses from the inventory) and execute `sudo -i`.
- For the SSH connection use the node password from the inventory file, `sudo -i` works without password.

```bash
[student<X>@ansible ~]$ ssh student<X>@22.33.44.55
student<X>@22.33.44.55's password:
Last login: Thu Jul  4 14:47:04 2019 from 11.22.33.44
[student<X>@node1 ~]$ sudo -i
[root@node1 ~]#
```

What does this mean?

  - Tower user **student\<X>** can connect to the managed hosts with password based SSH

  - User **student\<X>** can execute commands on the managed hosts as **root** with `sudo`

## Configure Machine Credentials

Now we will configure the credentials to access our managed hosts from Tower. In the **RESOURCES** menu choose **Credentials**. Now:

Click the ![plus](../images/ygreen_plus.png) button to add new credentials

  - **NAME:** Workshop Credentials

  - **ORGANIZATION:** Default

  - **CREDENTIAL TYPE:** Click on the magnifying glass, pick **Machine** and click ![plus](../images/yselect.png)

  - **USERNAME:** student\<X\> - make sure to replace the **\<X\>** with your actual student number!

  - **PASSWORD:** Enter the password from the inventory file.

  - **PRIVILEGE ESCALATION METHOD:** sudo

  - Click **SAVE**

  - Go back to the **RESOURCES** → **Credentials** → **Workshop Credentials** and note that the password is not visible.

> **Tip**
>
> Whenever you see a magnifiying glass icon next to an input field, clicking it will open a list to choose from.

You have now setup credentials to use later for your inventory hosts.

## Run Ad Hoc Commands

As you’ve probably done with Ansible before you can run ad hoc commands from Tower as well.

  - In the web UI go to **RESOURCES → Inventories → Workshop Inventory**

  - Click the **HOSTS** button to change into the hosts view and select the three hosts by ticking the boxes to the left of the host entries.

  - Click **RUN COMMANDS**. In the next screen you have to specify the ad hoc command:

      - As **MODULE** choose **ping**

      - For **MACHINE CREDENTIAL** click the magnifying glass icon and select **Workshop Credentials**.

      - Click **LAUNCH**, and watch the output.

The simple **ping** module doesn’t need options. For other modules you need to supply the command to run as an argument. Try the **command** module to find the userid of the executing user using an ad hoc command.

- **MODULE:** command

- **ARGUMENTS:** id

> **Tip**
>
> After choosing the module to run, Tower will provide a link to the docs page for the module when clicking the question mark next to "Arguments". This is handy, give it a try.

How about trying to get some secret information from the system? Try to print out */etc/shadow*.

- **MODULE:** command

- **ARGUMENTS:** cat /etc/shadow

> **Warning**
>
> **Expect an error\!**

Oops, the last one didn’t went well, all red.

Re-run the last ad hoc command but this time tick the **ENABLE PRIVILEGE ESCALATION** box.

As you see, this time it worked. For tasks that have to run as root you need to escalate the privileges. This is the same as the **become: yes** you’ve probably used often in your Ansible Playbooks.

## Challenge Lab: Ad Hoc Commands

Okay, a small challenge: Run an ad hoc to make sure the package "tmux" is installed on all hosts. If unsure, consult the documentation either via the web UI as shown above or by running `[ansible@tower ~]$ ansible-doc yum` on your Tower control host.

> **Warning**
>
> <details><summary>Solution below\!</summary>
> <p>
> - **MODULE:** yum
>
> - **ARGUMENTS:** name=tmux
>
> - Tick **ENABLE PRIVILEGE ESCALATION**
> </p>
> </details>

> **Tip**
>
> The yellow output of the command indicates Ansible has actually done something (here it needed to install the package). If you run the ad hoc command a second time, the output will be green and inform you that the package was already installed. So yellow in Ansible doesn’t mean "be careful"…​ ;-).

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
# Exercise 2.3 - Projects & job templates

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

A Tower **Project** is a logical collection of Ansible Playbooks. You can manage your playbooks by placing them into a source code management (SCM) system supported by Tower, including Git, Subversion, and Mercurial.

You should definitely keep your Playbooks under version control. In this lab we’ll use Playbooks kept in a Git repository.

## Setup Git Repository

For this demonstration we will use playbooks stored in a Git repository:

**https://github.com/ansible/workshop-examples**


A Playbook to install the Apache webserver has already been commited to the directory **rhel/apache**, `apache_install.yml`:

```yaml
---
- name: Apache server installed
  hosts: all

  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest

  - name: latest firewalld version installed
    yum:
      name: firewalld
      state: latest

  - name: firewalld enabled and running
    service:
      name: firewalld
      enabled: true
      state: started

  - name: firewalld permits http service
    firewalld:
      service: http
      permanent: true
      state: enabled
      immediate: yes

  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
```

> **Tip**
>
> Note the difference to other Playbooks you might have written\! Most importantly there is no `become` and `hosts` is set to `all`.

To configure and use this repository as a **Source Control Management (SCM)** system in Tower you have to create a **Project** that uses the repository

## Create the Project

  - Go to **RESOURCES → Projects** in the side menu view click the ![plus](../images/ygreen_plus.png) button. Fill in the form:

  - **NAME:** Ansible Workshop Examples

  - **ORGANIZATION:** Default

  - **SCM TYPE:** Git

Now you need the URL to access the repo. Go to the Github repository mentioned above, choose the green **Clone or download** button on the right, click on **Use https** and copy the HTTPS URL.

> **Note**
>
> If there is no **Use https** to click on, but a **Use SSH**, you are fine: just copy the URL. The important thing is that you copy the URL starting with **https**.

 Enter the URL into the Project configuration:

- **SCM URL:** `https://github.com/ansible/workshop-examples.git`

- **SCM UPDATE OPTIONS:** Tick all three boxes to always get a fresh copy of the repository and to update the repository when launching a job.

- Click **SAVE**

The new Project will be synced automatically after creation. But you can also do this automatically: Sync the Project again with the Git repository by going to the **Projects** view and clicking the circular arrow **Get latest SCM revision** icon to the right of the Project.

After starting the sync job, go to the **Jobs** view: there is a new job for the update of the Git repository.

## Create a Job Template and Run a Job

A job template is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many times. So before running an Ansible **Job** from Tower you must create a **Job Template** that pulls together:

- **Inventory**: On what hosts should the job run?

- **Credentials** What credentials are needed to log into the hosts?

- **Project**: Where is the Playbook?

- **What** Playbook to use?

Okay, let’s just do that: Go to the **Templates** view, click the ![plus](../images/ygreen_plus.png) button and choose **Job Template**.

> **Tip**
>
> Remember that you can often click on magnfying glasses to get an overview of options to pick to fill in fields.

- **NAME:** Install Apache

- **JOB TYPE:** Run

- **INVENTORY:** Workshop Inventory

- **PROJECT:** Ansible Workshop Examples

- **PLAYBOOK:** `rhel/apache/apache_install.yml`

- **CREDENTIAL:** Workshop Credentials

- We need to run the tasks as root so check **Enable privilege escalation**

- Click **SAVE**

You can start the job by directly clicking the blue **LAUNCH** button, or by clicking on the rocket in the Job Templates overview. After launching the Job Template, you are automatically brought to the job overview where you can follow the playbook execution in real time:

![job exection](../images/yjob_overview.png)

Since this might take some time, have a closer look at all the details provided:

- All details of the job template like inventory, project, credentials and playbook are shown.

- Additionally, the actual revision of the playbook is recorded here - this makes it easier to analyse job runs later on.

- Also the time of execution with start and end time is recorded, giving you an idea of how long a job execution actually was.

- On the right side, the output of the playbook run is shown. Click on a node underneath a task and see that detailed information are provided for each task of each node.

After the Job has finished go to the main **Jobs** view: All jobs are listed here, you should see directly before the Playbook run an SCM update was started. This is the Git update we configured for the **Project** on launch\!

## Challenge Lab: Check the Result

Time for a little challenge:

  - Use an ad hoc command on both hosts to make sure Apache has been installed and is running.

You have already been through all the steps needed, so try this for yourself.

> **Tip**
>
> What about `systemctl status httpd`?

> **Warning**
>
> **Solution Below**

- Go to **Inventories** → **Workshop Inventory**

- In the **HOSTS** view select all hosts and click **RUN COMMANDS**

- **MODULE:** command

- **ARGUMENTS:** systemctl status httpd

- **MACHINE CREDENTIALS:** Workshop Credentials

- Click **LAUNCH**

## What About Some Practice?

Here is a list of tasks:

> **Warning**
>
> Please make sure to finish these steps as the next chapter depends on it\!

- Create a new inventory called `Webserver` and make only `node1` member of it.

- Copy the `Install Apache` template using the copy icon in the **Templates** view

- Change the name to `Install Apache Ask`

- Change the **INVENTORY** setting of the Project so it will ask for the inventory on launch

- **SAVE**

- Launch the `Install Apache Ask` template.

- It will now ask for the inventory to use, choose the `Webserver` inventory and click **LAUNCH**

- Wait until the Job has finished and make sure it run only on `node1`

> **Tip**
>
> The Job didn’t change anything because Apache was already installed in the latest version.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
# Exercise 2.4 - Surveys

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

You might have noticed the **ADD SURVEY** button in the **Template** configuration view. A survey is a way to create a simple form to ask for parameters that get used as variables when a **Template** is launched as a **Job**.

You have installed Apache on all hosts in the job you just run. Now we’re going to extend on this:

- Use a proper role which has a Jinja2 template to deploy an `index.html` file.

- Create a job **Template** with a survey to collect the values for the `index.html` template.

- Launch the job **Template**

Additionally, the role will also make sure that the Apache configuration is properly set up - in case it got mixed up during the other exercises.

> **Tip**
>
> The survey feature only provides a simple query for data - it does not support four-eye principles, queries based on dynamic data or nested menus.

## The Apache-configuration Role

The Playbook and the role with the Jinja template already exist in the Github repository **https://github.com/ansible/workshop-examples** in the directory `rhel/apache`**`.

 Head over to the Github UI and have a look at the content: the playbook `apache_role_install.yml` merely references the role. The role can be found in the `roles/role_apache` subdirectory.

 - Inside the role, note the two variables in the `templates/index.html.j2` template file marked by `{{…​}}`\.
 - Also, check out the tasks in `tasks/main.yml` that deploy the file from the template.

What is this Playbook doing? It creates a file (**dest**) on the managed hosts from the template (**src**).

The role also deploys a static configuration for Apache. This is to make sure that all changes done in the previous chapters are overwritten and your examples work properly.

Because the Playbook and role is located in the same Github repo as the `apache_install.yml` Playbook you don't have to configure a new project for this exercise.

## Create a Template with a Survey

Now you create a new Template that includes a survey.

### Create Template

- Go to **Templates**, click the ![plus](../images/ygreen_plus.png) button and choose **Job Template**

- **NAME:** Create index.html

- Configure the template to:

    - Use the `Ansible Workshop Examples` **Project**

    - Use the `apache_role_install.yml` **Playbook**

    - To run on `node1`

    - To run in privileged mode

Try for yourself, the solution is below.

> **Warning**
>
> **Solution Below\!**

- **NAME:** Create index.html

- **JOB TYPE:** Run

- **INVENTORY:** Webserver

- **Project:** Ansible Workshop Examples

- **PLAYBOOK:** `rhel/apache/apache_role_install.yml`

- **CREDENTIAL:** Workshop Credentials

- **OPTIONS:** Enable Privilege Escalation

- Click **SAVE**

> **Warning**
>
> **Do not run the template yet!**

### Add the Survey

- In the Template, click the **ADD SURVEY** button

- Under **ADD SURVEY PROMPT** fill in:

    - **PROMPT:** First Line

    - **ANSWER VARIABLE NAME:** `first_line`

    - **ANSWER TYPE:** Text

- Click **+ADD**

- In the same way add a second **Survey Prompt**

    - **PROMPT:** Second Line

    - **ANSWER VARIABLE NAME:** `second_line`

    - **ANSWER TYPE:** Text

- Click **+ADD**

- Click **SAVE** for the Survey

- Click **SAVE** for the Template

## Launch the Template

Now launch **Create index.html** job template.

Before the actual launch the survey will ask for **First Line** and **Second Line**. Fill in some text and click **Next**. The next window shows the values, if all is good run the Job by clicking **Launch**.

> **Tip**
>
> Note how the two survey lines are shown to the left of the Job view as **Extra Variables**.

After the job has completed, check the Apache homepage. In the SSH console on the control host, execute `curl` against the IP address of your `node1`:

```bash
$ curl http://22.33.44.55
<body>
<h1>Apache is running fine</h1>
<h1>This is survey field "First Line": line one</h1>
<h1>This is survey field "Second Line": line two</h1>
</body>
```
Note how the two variables where used by the playbook to create the content of the `index.html` file.

## What About Some Practice?

Here is a list of tasks:

> **Warning**
>
> **Please make sure to finish these steps as the next chapter depends on it\!**

- Take the inventory `Webserver` and add the other nodes, `node2` and `node3` as well.

- Run the **Create index.html** Template again.

- Verify the results on the other two nodes by using `curl` against their IP addresses.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
# Exercise 2.5 - Role-based access control

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

You have already learned how Tower separates credentials from users. Another advantage of Ansible Tower is the user and group rights management.

## Ansible Tower Users

There are three types of Tower Users:

- **Normal User**: Have read and write access limited to the inventory and projects for which that user has been granted the appropriate roles and privileges.

- **System Auditor**: Auditors implicitly inherit the read-only capability for all objects within the Tower environment.

- **System Administrator**: Has admin, read, and write privileges over the entire Tower installation.

Let’s create a user:

- In the Tower menu under **ACCESS** click **Users**

- Click the green plus button

- Fill in the values for the new user:

    - **FIRST NAME:** Werner

    - **LAST NAME:** Web

    - **EMAIL:** wweb@example.com

    - **USERNAME:** wweb

    - **USER TYPE:** Normal User

    - **PASSWORD:** ansible

    - Confirm password

- Click **SAVE**

## Ansible Tower Teams

A Team is a subdivision of an organization with associated users, projects, credentials, and permissions. Teams provide a means to implement role-based access control schemes and delegate responsibilities across organizations. For instance, permissions may be granted to a whole Team rather than each user on the Team.

Create a Team:

- In the menu go to **ACCESS → Teams**

- Click the green plus button and create a team named `Web Content`.

- Click **SAVE**

Now you can add a user to the Team:

- Switch to the Users view of the `Web Content` Team by clicking the **USERS** button.

- Click the green plus button, tick the box next to the `wweb` user and click **SAVE**.

Now click the **PERMISSIONS** button in the **TEAMS** view, you will be greeted with "No Permissions Have Been Granted".

Permissions allow to read, modify, and administer projects, inventories, and other Tower elements. Permissions can be set for different resources.

## Granting Permissions

To allow users or teams to actually do something, you have to set permissions. The user **wweb** should only be allowed to modify content of the assigned webservers.

Add the permission to use the template:

- In the Permissions view of the Team `Web Content` click the green plus button to add permissions.

- A new window opens. You can choose to set permissions for a number of resources.

    - Select the resource type **JOB TEMPLATES**

    - Choose the `Create index.html` Template by ticking the box next to it.

- The second part of the window opens, here you assign roles to the selected resource.

    - Choose **EXECUTE**

- Click **SAVE**

## Test Permissions

Now log out of Tower’s web UI and in again as the **wweb** user.

- Go to the **Templates** view, you should notice for wweb only the `Create
  index.html` template is listed. He is allowed to view and launch, but not to edit the Template. Just open the template and try to change it.

- Run the Job Template by clicking the rocket icon. Enter the survey content to your liking and launch the job.

- In the following **Jobs** view have a good look around, note that there where changes to the host (of course…​).

Check the result: execute `curl` again on the control host to pull the content of the webserver on the IP address of `node1` (you could of course check `node2` and `node3`, too):

```bash
$ curl http://22.33.44.55
```

Just recall what you have just done: You enabled a restricted user to run an Ansible Playbook

  - Without having access to the credentials

  - Without being able to change the Playbook itself

  - But with the ability to change variables you predefined\!

Effectively you provided the power to execute automation to another user without handing out your credentials or giving the user the ability to change the automation code. And yet, at the same time the user can still modify things based on the surveys you created.

This capability is one of the main strengths of Ansible Tower\!

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
# Exercise 2.6 - Workflows

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

# Ansible Tower Workflows

Workflows were introduced as a major new feature in Ansible Tower 3.1. The basic idea of a workflow is to link multiple Job Templates together. They may or may not share inventory, Playbooks or even permissions. The links can be conditional:

  - if job template A succeeds, job template B is automatically executed afterwards

  - but in case of failure, job template C will be run.

And the workflows are not even limited to Job Templates, but can also include project or inventory updates.

This enables new applications for Tower: different Job Templates can build upon each other. E.g. the networking team creates playbooks with their own content, in their own Git repository and even targeting their own inventory, while the operations team also has their own repos, playbooks and inventory.

In this lab you’ll learn how to setup a workflow.

## Lab Scenario

You have two departements in your organization:

  - The web operations team that is developing Playbooks in their own Git repository.

  - The web applications team, that develops JSP web applications for Tomcat in their Git repository.

When there is a new Tomcat server to deploy, two things need to happen:

  - Tomcat needs to be installed, the firewall needs to be opened and Tomcat should get started.

  - The most recent version of the web application needs to be deployed.

To make things somewhat easier for you, everything needed already exists in a Github repository: Playbooks, JSP-files etc. You just need to glue it together.

> **Note**
>
> In this example we use two different branches of the same repository for the content of the separate teams. In reality the structure of your SCM repositories depends on a lot of factors and could be different.

## Set up Projects

First you have to set up the Git repo as Projects like you normally would. You have done this before, try to do this on your own. Detailed instructions can be found below.

> **Warning**
>
> **If you are still logged in as user **wweb**, log out of and log in as user **admin** again.**

- Create the project for web operations:

  - It should be named **Webops Git Repo**

  - The URL to access the repo is **https://github.com/ansible/workshop-examples.git**

  - The **SCM BRANCH/TAG/COMMIT** is **webops**

- Create the project for the application developers:

  - It should be named **Webdev Git Repo**

  - The URL to access the repo is **https://github.com/ansible/workshop-examples.git**

  - The **SCM BRANCH/TAG/COMMIT** is **webdev**

> **Warning**
>
> **Solution Below**

- Create the project for web operations. In the **Projects** view click the green plus button and fill in:

    - **NAME:** Webops Git Repo

    - **ORGANIZATION:** Default

    - **SCM TYPE:** Git

    - **SCM URL:** https://github.com/ansible/workshop-examples.git

    - **SCM BRANCH/TAG/COMMIT:** webops

    - **SCM UPDATE OPTIONS:** Tick all three boxes.

- Click **SAVE**

- Create the project for the application developers. In the **Projects** view click the green plus button and fill in:

    - **NAME:** Webdev Git Repo

    - **ORGANIZATION:** Default

    - **SCM TYPE:** Git

    - **SCM URL:** https://github.com/ansible/workshop-examples.git

    - **SCM BRANCH/TAG/COMMIT:** webdev

    - **SCM UPDATE OPTIONS:** Tick all three boxes.

- Click **SAVE**

## Set up Job Templates

Now you have to create Job Templates like you would for "normal" Jobs.

  - Go to the **Templates** view, click the green plus button and choose **Job Template**:

      - **NAME:** Tomcat Deploy

      - **JOB TYPE:** Run

      - **INVENTORY:** Workshop Inventory

      - **PROJECT:** Webops Git Repo

      - **PLAYBOOK:** `rhel/webops/tomcat.yml`

      - **CREDENTIAL:** Workshop Credentials

      - **OPTIONS:** Enable privilege escalation

  - Click **SAVE**

  - Go to the **Templates** view, click the green plus button and choose **Job Template**:

      - **NAME:** Web App Deploy

      - **JOB TYPE:** Run

      - **INVENTORY:** Workshop Inventory

      - **PROJECT:** Webdev Git Repo

      - **PLAYBOOK:** `rhel/webdev/create_jsp.yml`

      - **CREDENTIALS:** Workshop Credentials

      - **OPTIONS:** Enable privilege escalation

  - Click **SAVE**

> **Tip**
>
> If you want to know what the Playbooks look like, check out the Github URL and switch to the appropriate branches.

## Set up the Workflow

And now you finally set up the workflow. Workflows are configured in the **Templates** view, you might have noticed you can choose between **Job Template** and **Workflow Template** when adding a template so this is finally making sense.

  - Go to the **Templates** view and click the the green plus button. This time choose **Workflow Template**

      - **NAME:** Deploy Webapp Server

      - **ORGANIZATION:** Default

  - Click **SAVE**

  - After saving the template the **Workflow Visualizer** opens to allow you to build a workflow. You can later open the **Workflow Visualizer** again by using the button on the template details page.

  - Click on the **START** button, a new node opens. To the right you can assign an action to the node, you can choose between **JOBS**, **PROJECT SYNC** and **INVENTORY SYNC**.

  - In this lab we’ll link Jobs together, so select the **Tomcat Deploy** job and click **SELECT**.

  - The node gets annotated with the name of the job. Hover the mouse pointer over the node, you’ll see a red **x**, a green **+** and a blue **chain**-symbol appear.

> **Tip**
>
> Using the red "x" allows you to remove the node, the green plus lets you add the next node and the chain-symbol links to another node .

  - Click the green **+** sign

  - Choose **Web App Deploy** as the next Job (you might have to switch to the next page)

  - Leave **Type** set to **On Success**

> **Tip**
>
> The type allows for more complex workflows. You could lay out different execution paths for successful and for failed Playbook runs.

  - Click **SELECT**

  - Click **SAVE** in the **WORKFLOW VISUALIZER** view

  - Click **SAVE** in the **Workflow Template** view

> **Tip**
>
> The **Workflow Visualizer** has options for setting up more advanced workflows, please refer to the documentation.

## And Action

Your workflow is ready to go, launch it.

  - Click the blue **LAUNCH** button directly or go to the the **Templates** view and launch the **Deploy Webapp Server** workflow by clicking the rocket icon.

![jobs view of workflow](../images/yjob_workflow.png)

Note how the workflow run is shown in the job view. In contrast to a normal job template job execution this time there is no playbook output on the right, but a visual representation of the different workflow steps. If you want to look at the actual playbooks behind that, click **DETAILS** in each step. If you want to get back from a details view to the corresponding workflow, click the ![w-button](../images/yw_button.png) in the **JOB TEMPLATE** line in the **DETAILS** part on the left side of the job overview.

After the job was finished, check if everything worked fine: log into `node1`, `node2` or `node3` from your control host and run:

```bash
$ curl http://localhost:8080/coolapp/
```

> **Tip**
>
> You might have to wait a couple of minutes until Tomcat answers requests.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
# Exercise 2.7 - Wrap up

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

# Final Challenge or Putting it all Together

This is the final challenge where we try to put most of what you have learned together.

## Let’s set the stage

Your operations team and your application development team like what they see in Tower. To really use it in their environment they put together these requirements:

- All webservers (`node1`, `node2` and `node3`) should go in one group

- As the webservers can be used for development purposes or in production, there has to be a way to flag them accordingly as "stage dev" or "stage prod".

    - Currently `node1` and `node3` are used as a development system and `node2` is in production.

- Of course the content of the world famous application "index.html" will be different between dev and prod stages.

    - There should be a title on the page stating the environment

    - There should be a content field

- The content writer `wweb` should have access to a survey to change the content for dev and prod servers.

## The Git Repository

All code is already in place - this is a Tower lab after all. Check out the **Ansible Workshop Examples** git repository at **https://github.com/ansible/workshop-examples**. There you will find the playbook `webcontent.yml`, which calls the role `role_webcontent`.

Compared to the previous Apache installation role there is a major difference: there are now two versions of an `index.html` template, and a task deploying the template file which has a variable as part of the source file name:

`dev_index.html.j2`

```html
<body>
<h1>This is a development webserver, have fun!</h1>
{{ dev_content }}
</body>
```

`prod_index.html.j2`

```html
<body>
<h1>This is a production webserver, take care!</h1>
{{ prod_content }}
</body>
```

`main.yml`

```yaml
[...]
- name: Deploy index.html from template
  template:
    src: "{{ stage }}_index.html.j2"
    dest: /var/www/html/index.html
  notify: apache-restart
```

## Prepare Inventory

There is of course more then one way to accomplish this, but here is what you should do:

- Make sure all hosts are in the inventory group `Webserver`.

- Define a variable `stage` with the value `dev` for the `Webserver` inventory:

    - Add `stage: dev` to the inventory `Webserver` by putting it into the **VARIABLES** field beneath the three start-yaml dashes.

- In the same way add a variable `stage: prod` but this time only for `node2` (by clicking the hostname) so that it overrides the inventory variable.

> **Tip**
>
> Make sure to keep the three dashes that mark the YAML start and the `ansible_host` line in place\!

## Create the Template

- Create a new **Job Template** named `Create Web Content` that

    - targets the `Webserver` inventory

    - uses the Playbook `rhel/apache/webcontent.yml` from the **Ansible Workshop Examples** Project

    - Defines two variables: `dev_content: default dev content` and `prod_content: default prod content` in the **EXTRA VARIABLES FIELD**

    - Uses `Workshop Credentials` and runs with privilege escalation

- Save and run the template

## Check the results

This time we use the power of Ansible to check the results: execute curl to get the web content from each node, orchestrated by an ad-hoc command on the command line of your Tower control host:

> **Tip**
>
> We are using the `ansible_host` variable in the URL to access every node in the inventory group.

```bash
[student<X>@ansible ~]$ ansible web -m command -a "curl -s http://{{ ansible_host }}"
 [WARNING]: Consider using the get_url or uri module rather than running 'curl'.  If you need to use command because get_url or uri is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.

node2 | CHANGED | rc=0 >>
<body>
<h1>This is a production webserver, take care!</h1>
prod wweb
</body>

node1 | CHANGED | rc=0 >>
<body>
<h1>This is a development webserver, have fun!</h1>
dev wweb
</body>

node3 | CHANGED | rc=0 >>
<body>
<h1>This is a development webserver, have fun!</h1>
dev wweb
</body>
```

Note the warning in the first line about not to use `curl` via the `command` module since there are better modules right within Ansible. We will come back to that in the next part.

## Add Survey

- Add a survey to the Template to allow changing the variables `dev_content` and `prod_content`

- Add permissions to the Team `Web Content` so the Template **Create Web Content** can be executed by `wweb`.

- Run the survey as user `wweb`

Check the results again from your Tower control host. Since we got a warning last time using `curl` via the `command` module, this time we will use the dedicated `uri` module. As arguments it needs the actual URL and a flag to output the body in the results.

```bash
[student<X>ansible ~]$ ansible web -m command -a "curl -s http://{{ ansible_host }}"
node3 | SUCCESS => {
    "accept_ranges": "bytes",
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "connection": "close",
    "content": "<body>\n<h1>This is a development webserver, have fun!</h1>\nwerners dev content\n</body>\n",
    "content_length": "87",
    "content_type": "text/html; charset=UTF-8",
    "cookies": {},
    "cookies_string": "",
    "date": "Tue, 29 Oct 2019 11:14:24 GMT",
    "elapsed": 0,
    "etag": "\"57-5960ab74fc401\"",
    "last_modified": "Tue, 29 Oct 2019 11:14:12 GMT",
    "msg": "OK (87 bytes)",
    "redirected": false,
    "server": "Apache/2.4.6 (Red Hat Enterprise Linux)",
    "status": 200,
    "url": "http://18.205.236.208"
}
[...]
```

## Solution

> **Warning**
>
> **Solution Not Below**

You have done all the required configuration steps in the lab already. If unsure, just refer back to the respective chapters.

# The End

Congratulations, you finished your labs\! We hope you enjoyed your first encounter with Ansible Tower as much as we enjoyed creating the labs.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
