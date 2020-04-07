# Exercise 4 - Creating Tower Objects Using `awx`

Next we want to configure Tower so that we can run Ansible jobs. For
this we need Inventories, Projects, Credentials and Job Templates. When
you first start with Tower, this is usually done via web UI. But using
Tower more often and especially when you want to boot-strap a configured
Tower from the bottom up it makes sense to do this via **awx** in a
scripted way - especially when Ansible is not yet set up properly.

In the first step you will learn to setup the inventory with **awx**
step by step to get practice using the tool. For the following steps
(Projects, Credentials, Job Templates) we will not go into such detail.
Instead we will just explain the actual **awx** commands and put
them all into a shell script. This shell script will serve as an example
of how to bootstrap a Tower from bottom up.

## Create an Inventory

First we create a static inventory, we’ll get to dynamic inventories
later on. Try to figure out the proper invocation of **awx**
yourself and create an inventory name **Example Inventory**.

> **Tip**
>
> Remember how you used the **awx** help to get down to the needed
> command.

> **Warning**
>
> <details><summary>Solution below!</summary>
> <p>
>
>    [root@ansible ~]# awx -f human inventory create --name "Example Inventory" --organization "Default"
>
> **Tip**
>
> You can work with multiple organizations in Tower. In this lab we’ll
> work in the **Default** organization.
>
> </p>
> </details>

### Add Hosts to the Inventory using **awx**

Now that we have the empty inventory created, add your two managed hosts
**host1.example.com** and **host2.example.com**, again using
**awx**.

> **Warning**
>
> <details><summary>Solution below!</summary>
> <p>
>
>    [root@ansible ~]# awx -f human host create --name "student1-node1.89cd.ansibleworkshops.com" --inventory "Example Inventory"
>
>    [root@ansible ~]# awx -f human host create --name "student1-node2.89cd.ansibleworkshops.com" --inventory "Example Inventory"
>
> </p>
> </details>

## Create script to contain this and all following awx commands

As mentioned one of the puproses of **awx** is to use it to
automatically configure more complex Tower setups. In such cases,
multiple **awx** commands are put together in a script. We follow
that practice in our example here, and create a shell script on the
control host with all commands you have to run to bootstrap Tower. So in
the next few paragraphs we describe the steps to do and describe the
corresponding **awx** commands. But we will not execute them, but
instead write them into a script.

In **code-server** create a new file **File->New File** and save it (**File->Save As**) **setup-tower.sh**. Add the commands executed above:

    #!/bin/bash
    awx -f human inventory create --name "Example Inventory" --organization "Default"
    awx -f human host create --name "student1-node1.89cd.ansibleworkshops.com" \
      --inventory "Example Inventory"
    awx -f human host create --name "student1-node2.89cd.ansibleworkshops.com" \
      --inventory "Example Inventory"

> **Tip**
>
> You have run these commands above already, true. But we want to show
> how to create the full script here.

Next, save the script and make the script executable in the terminal window.
Then launch it:

    [root@ansible ~]# chmod u+x /home/student1/setup-tower.sh
    [root@ansible ~]# /home/student1/setup-tower.sh

> **Tip**
>
> You will see that **awx** is idempotent and recognises the objects as duplicate, so it’s fine that you did
> run the **awx** commands already.

From now on we’ll explain the needed comands for each of the next steps
and add them to the script step-by-step.

## Create Machine Credentials

> **Tip**
>
> SSH keys have already been created and distributed in your lab
> environment and `sudo` has been setup on the managed hosts to allow
> password-less login for user **ansible** on **control.example.com**.

Now we want to configure the credentials to access our managed hosts
from Tower, we are using SSH with password authentication in this lab. Add the following line to to **setup-tower.sh**, but don’t run
the script yet:

    awx -f human credential create --name "Example Credentials" \
                      --organization "Default" \
                      --credential_type "Machine" \
                      --inputs="{\"username\":\"ec2-user\",\"ssh_key_data\":\"$(sed -E ':a;N;$!ba;s/\r{0,1}\n/\\n/g' /home/ec2-user/.ssh/aws-private.pem)\n\",\"become_method\":\"sudo\"}

Don’t run the shell script yet, first got through the following steps to
add all commands to it.

> **Warning**
>
> As the **awx** commands get longer you’ll find we use the
> back-slash for line wraps to make the commands readable. You can copy
> the examples or use them without the \\ on one line, of course.

## Create the Project

The Ansible content used in this lab is hosted on Github. The next step
is to add a project to import the playbooks. Add the appropriate
**awx** line to the script **setup-tower.sh**:

    awx -f human project create --name="Apache" \
                   --scm_type=git \
                   --scm_url="https://github.com/goetzrieger/ansible-labs-playbooks.git" \
                   --organization "Default" \
                   --scm_clean=true --scm_delete_on_update=true --scm_update_on_launch=true \
                   --wait

> **Tip**
>
> Note that the first parameter to **awx** is different here since
> we work on the resource **project**.

## Create a Job Template

Before running an Ansible **Job** from your Tower cluster you must
create a **Job Template**, again business as usual for Tower users. Here
**awx** will work on the resource **job\_template**. Add the
following line to your script **setup-tower.sh**. Don’t run the script
yet.

    awx -f human job_templates create  \
                    --name="Install Apache" \
                    --inventory="Example Inventory" \
                    --project=Apache \
                    --playbook=apache_install.yml \
                    --become_enabled="yes"
                   

## Review the final script and execute it

Verify that your script has all the pieces needed for a properly
configured Tower:

  - inventory with hosts

  - machine credentials and credentials for Git

  - project

  - job template

The final script is also shown here:

    #!/bin/bash
    awx inventory create --name "Example Inventory" --organization "Default"
    awx host create --name "support1.ewl05.internal" --inventory "Example Inventory"
    awx host create --name "support2.ewl05.internal" --inventory "Example Inventory"

    awx -f human credential create --name "Example Credentials" \
                      --organization "Default" \
                      --credential_type "Machine" \
                      --inputs="{\"username\":\"ec2-user\",\"ssh_key_data\":\"$(sed -E ':a;N;$!ba;s/\r{0,1}\n/\\n/g' /home/ec2-user/.ssh/aws-private.pem)\n\",\"become_method\":\"sudo\"}"
                      
    awx -f human project create --name="Apache" \
                   --scm_type=git \
                   --scm_url="https://github.com/goetzrieger/ansible-labs-playbooks.git" \
                   --organization "Default" \
                   --scm_clean=true --scm_delete_on_update=true --scm_update_on_launch=true \
                   --wait

    awx -f human job_template create  \
                    --name="Install Apache" \
                    --inventory="Example Inventory" \
                    --credential="Example Credentials" \
                    --project=Apache \
                    --playbook=apache_install.yml \
                    --become_enabled="yes"


Run the script, and verify that all resources were properly created in
the web UI.

**Take away:**

It’s easy to script Tower’s configuration using **awx**. This way
you can bootstrap a new Tower node or script tasks you have to run on a
regular basis. You will learn more about the Tower API at the end of the
lab.

## It’s a Cluster After All

We are working in a clustered environment. To verify that the resources
were created on all instances properly, login to the other Tower
instances web UIs (the ones you didn’t configured the inventory and
credentials on).

Have a look around, everything we automatically configured on one Tower
instance with our script was **synced automatically** to the other
nodes. Inventory, credentials, projects, templates, all there.
