+++
title = "Collections in Tower"
weight = 4
+++

Red Hat Ansible Tower supports Ansible Collections starting with version 3.5 - earlier version will not automatically install and configure them for you. To make sure Ansible Collections are recognized by Red Hat Ansible Tower a requirements file is needed and has to be stored in the proper directory.

Ansible Galaxy is already configured by default, however if you want your Red Hat Ansible Tower to prefer and fetch content from the Red Hat Automation Hub, additional configuration changes are required. They are addressed in a the chapter [Use Automation Hub](../7-use-automation-hub/) in this lab.

In this exercise you will learn how to define an Ansible Collection as a requirement in a format recognized by Red Hat Ansible Tower. We'll be using the **posix** collection again from Ansible Galaxy, but as you probably know for Tower to access the needed bits & pieces a version control system is needed. For the sake of keeping this lab setup easy, we'll set up a local Git server for this.

## Set up a Local Git Server

So, let’s get started. We have to create a simplistic Git server on our control host. In a more typical environment, you would work with GitLab, Gitea, or any other commercial Git server.

{{% notice tip %}}
Make sure to run these steps from your users home directory!
{{% /notice %}}

```
[{{< param "control_prompt" >}} ~]$ wget https://raw.githubusercontent.com/ansible-labs-summit-crew/structured-content/master/simple_git.yml
[{{< param "control_prompt" >}} ~]$ ansible-playbook simple_git.yml -e "git_project=tower_collections"
```

Next we will clone the repository on the control host. To enable you to work with git on the command line the SSH key for user *ec2-user* was already added to the Git user *git*. Next, clone the repository on the control machine:

```bash
[{{< param "control_prompt" >}} ~]$ git clone git@ansible-1:projects/tower_collections
# Message "warning: You appear to have cloned an empty repository." is OK and can be ignored
[{{< param "control_prompt" >}} ~]$ git config --global push.default simple
[{{< param "control_prompt" >}} ~]$ git config --global user.name "Your Name"
[{{< param "control_prompt" >}} ~]$ git config --global user.email you@example.com
[{{< param "control_prompt" >}} ~]$ cd tower_collections/
```

{{% notice tip %}}
The repository is currently empty. The three config commands are just there to avoid useless warnings from Git.
{{% /notice %}}

You now have a local Git server that can be accessed via SSH from Tower.

## Create Content for the Tower Project

Red Hat Ansible Tower can download and install Ansible Collections automatically before executing a Job Template. If a `collections/requirements.yml` exists in your project, it will be parsed and the Ansible Collections specified in this file will be automatically installed.

{{% notice tip %}}
Starting with Red Hat Ansible Tower 3.6 the working directory for the Job Template is in `/tmp`. Since the Ansible Collection is downloaded into this directory before the Job Template is executed, you will not find temporary files of your Ansible Collection in `/var/lib/awx/projects/`.
{{% /notice %}}

The format of the `requirements.yml` for Ansible Collections is very similar to the one for roles, however it is very important to store in the folder `collections`.

Let's create the files needed to see how you can use collections in Tower. This is of course just a simple example. First create the `collections` directory in your Git repo (you should have changed into `tower_collections` already above):

```bash
[{{< param "control_prompt" >}} tower_collections]$ mkdir collections
```

Then create the `requirements.yml` file listing the collection(s) you need:

```bash
[{{< param "control_prompt" >}} tower_collections]$ cat > collections/requirements.yml << EOF
---
collections:
- ansible.posix
EOF
```

As this is a simple example we'll just add a Playbook to the Git repo now. Normally you would have a lot more content in your project repository. So what could we do as an example using the **posix** collection again? Let's create a Playbook to configure an `at` job:

```bash
[{{< param "control_prompt" >}} tower_collections]$ cat > configure_at_job.yml << EOF
---
- name: Install AT Job
  hosts: all

  tasks:
  - name: at command needs to be installed
    yum:
      name: at
      state: present

  - name: Schedule a command to execute in 20 minutes as root
    ansible.posix.at:
      command: ls -d / >/dev/null
      count: 20
      units: minutes
EOF
```

{{% notice tip %}}
Note the usage of the Fully Qualified Collection Name in the Playbook
{{% /notice %}}


Make sure everything looks fine:

```bash
[{{< param "control_prompt" >}} tower_collections]$ tree
.
├── collections
│   └── requirements.yml
└── configure_at_job.yml
```

So far you created the code only locally on the control host, now you are ready to add it to the repository and push it:

```bash
[{{< param "control_prompt" >}} tower_collections]$ git add collections configure_at_job.yml
[{{< param "control_prompt" >}} tower_collections]$ git commit -m "Adding requirements.yml and playbook"
[{{< param "control_prompt" >}} tower_collections]$ git push
```

## Create the Project and Job Template

Not it's time to access your Tower web UI if you haven't done so out of curiosity already. Point your browser to the URL you were given on the lab landing page, similar to `https://{{< param "external_tower1" >}}` (replace `<N>` with your student number and `<LABID>` with the ID of this lab) and log in as `admin`. You can find the password again on the lab landing page.

To run your new Playbook in Tower you have to configure a number of objects:

- An **Inventory** with the managed hosts
- **Machine Credentials** to access the managed hosts
- Git **Credentials** to access your Git repository via SSH
- The Git repo as **Project**
- A **Job Template** to run the Playbook

We'll be a bit verbose for students new to Tower, if you are an Ansible Tower old-hand, just skip through and finish the configuration steps shown.

## Inventory and Machine Credentials

The inventory **Workshop Inventory** and the machine credentials **Workshop Credentials** have already been created in your lab environment.

## Configure SCM Credentials

Now we will configure the credentials to access our the Git repo on your control host via SSH. In the **RESOURCES** menu choose **Credentials**. Now:

Click the ![plus](../../images/green_plus.png?classes=inline) button to add new credentials

- **NAME:** Git Credentials
- **ORGANIZATION:** Click on the magnifying glass, pick **Default** and click **SELECT**
- **CREDENTIAL TYPE:** Click on the magnifying glass, pick **Source Control** as type and click **SELECT** (you will have to use the search or cycle through the types to find it).
- **USERNAME:** ec2-user

As we are using SSH key authentication, you have to provide an SSH private key that can be used to access the host with the Git repo as the user `git`.

{{% notice tip %}}
The Playbook we used to configure Git added the SSH private key to the `authorized_keys` of user `git`
{{% /notice %}}

Bring up your code-server terminal on Tower and use `cat` to get the SSH private key:

```bash
[{{< param "internal_control" >}} ~]$ cat ~/.ssh/aws-private.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA2nnL3m5sKvoSy37OZ8DQCTjTIPVmCJt/M02KgDt53+baYAFu1TIkC3Yk+HK1
[...]
-----END RSA PRIVATE KEY-----
```

- Copy the **complete private key** (including "BEGIN" and "END" lines) from the output and paste it into the **SSH PRIVATE KEY** field in the web UI.

- Click **SAVE**

You have now setup credentials to access the Git repo on your control host.

## Set up the Project

It's time to set up the Tower Project pointing to your Git repository holding your Playbook and collections requirements file.

- Go to **RESOURCES → Projects** in the side menu view click the ![plus](../../images/green_plus.png?classes=inline) button. Fill in the form:
- **NAME:** Collections Repo
- **ORGANIZATION:** Default
- **SCM TYPE:** Git

Now you need the URL to access the repo. You could get the URL in Github as **Clone URL**. Enter the URL into the Project configuration:

- **SCM URL:** {{< param "git_user" >}}@{{< param "internal_control" >}}:projects/tower_collections
- **GIT CREDENTIAL:** Click the maginifying glass and choose `Git Credential`, click **SELECT**
- **SCM UPDATE OPTIONS:** Tick the first three boxes to always get a fresh copy of the repository and to update the repository when launching a job.

- Click **SAVE**

The new Project will be synced automatically after creation. If everything went fine, you should see a green icon to the left of the new Project.

## Create the Job Template and run it

The last step is to create a Job Template to run the Playbook. Go to the **Templates** view, click the ![plus](../../images/green_plus.png?classes=inline) button and choose **Job Template**.

- **NAME:** Install AT Job
- **JOB TYPE:** Run
- **INVENTORY:** Workshop Inventory
- **PROJECT:** Collections Repo
- **PLAYBOOK:** `configure_at_job.yml`
- **CREDENTIAL:** Workshop Credentials

- We need to run the tasks as root so check **Enable privilege escalation**

- Click **SAVE**

You can start the job by directly clicking the blue **LAUNCH** button, or by clicking on the rocket in the Job Templates overview. After launching the Job Template, you are automatically brought to the job overview where you can follow the playbook execution in real time.

After the Job has finished bring up your VSCode terminal, as the job was run on all three managed nodes and the control/Tower node, you can simply check the result here. Run `sudo at -l` and you should see your job was scheduled successfully.

## Troubleshooting

Since Red Hat Ansible Tower does only check for updates in the repository in which you stored your Playbook, it might not do a refresh if there was a change in the Ansible Collection used by your Playbook. This happens particularly if you also combine Roles and Collections.

In this case you should check the option **Delete on Update** (like you did in this lab) which will delete the entire local directory during a refresh.

If there is a problem while parsing your `requirements.yml` it's worth testing it with the `ansible-galaxy` command. As a reminder, Red Hat Ansible Tower basically also just runs the command for you with the appropriate parameters, so testing this works manually makes a lot of sense.

```bash
ansible-galaxy collections install -r collections/requirements.yml -f
```

{{% notice tip %}}
The `-f` switch will forces a fresh installation of the specified Ansible Collections, otherwise `ansible-galaxy` will only install it, if it wasn't already installed. You can also use the `--force-with-deps` switch to make sure Ansible Collections which have dependencies to others are refreshed as well.
{{% /notice %}}

----
**Navigation**
<br>
[Previous Exercise](../3-using-collections-from-roles/) - [Next Exercise](../5-creating-collections/)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
