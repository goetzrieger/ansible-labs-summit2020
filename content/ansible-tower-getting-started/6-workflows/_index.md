+++
title = "Workflows"
weight = 6
+++

Workflows were introduced as a major new feature in Ansible Tower 3.1. The basic idea of a workflow is to link multiple Job Templates together. They may or may not share inventory, Playbooks or even permissions. The links can be conditional:

  - if job template A succeeds, job template B is automatically executed afterwards

  - but in case of failure, job template C will be run.

And the workflows are not even limited to Job Templates, but can also include project or inventory updates.

This enables new applications for Tower: different Job Templates can build upon each other. E.g. the networking team creates playbooks with their own content, in their own Git repository and even targeting their own inventory, while the operations team also has their own repos, playbooks and inventory.

In this lab you’ll learn how to setup a workflow.

## Lab Scenario

You have two departements in your organization:

  - The web operations team that is developing Playbooks for deploying web infrastructures in their own Git repository.

  - The web applications team, that develops JavaScript web applications for NodeJS in their Git repository.

When there is a new NodeJS-based website to deploy, two main steps need to happen:

The web operations team has to:

  - Install and configure NodeJS to run as a service.
  - An Apache instance needs to be installed and configured as proxy to pass requests for the NodeJS content to the NodeJS backend. And a lot of other steps might be needed, too. Like SELinux, firewall... you know the drill.

The web developer team has to:

  - Deploy the most recent version of the JavaScript web application.
  - Make sure stuff like making sure the directory structure is fine and service restarts happen.

To make things somewhat easier for you, everything needed already exists in a Github repository: Playbooks, JavaScript files etc. You just need to glue it together.

{{% notice tip %}}
In this example we use two different tags (each on its specific branch) of the same repository for the content of the separate teams. In reality the structure of your SCM repositories depends on a lot of factors and could be different.
{{% /notice %}}

## Set up Projects

First you have to set up the Git repo as Projects like you normally would. You have done this before, try to do this on your own. Detailed instructions can be found below.

{{% notice warning %}}
If you are still logged in as user **wweb**, log out of and log in as user **admin** again.
{{% /notice %}}

- Create the project for web operations:

  - It should be named **Webops Git Repo**

  - The URL to access the repo is **https://github.com/ansible/workshop-examples.git**

  - The **SCM BRANCH/TAG/COMMIT** is **webops_summit_2020**

  - Do not allow branch overrides

- Create the project for the application developers:

  - It should be named **Webdev Git Repo**

  - The URL to access the repo is **https://github.com/ansible/workshop-examples.git**

  - The **SCM BRANCH/TAG/COMMIT** is **webdev_summit_2020**

  - Do not allow branch overrides

<details><summary>**>>Click here for Solution<<**</summary>
<p>

- Create the project for web operations. In the **Projects** view click the green us button and fill in:
    - **NAME:** Webops Git Repo
    - **ORGANIZATION:** Default
    - **SCM TYPE:** Git
    - **SCM URL:** https://github.com/ansible/workshop-examples.git
    - **SCM BRANCH/TAG/COMMIT:** `webops_summit_2020`
    - **SCM UPDATE OPTIONS:** Tick the first three boxes.
- Click **SAVE**
- Create the project for the application developers. In the **Projects** view click the green plus button and fill in:
    - **NAME:** Webdev Git Repo
    - **ORGANIZATION:** Default
    - **SCM TYPE:** Git
    - **SCM URL:** https://github.com/ansible/workshop-examples.git
    - **SCM BRANCH/TAG/COMMIT:** `webdev_summit_2020`
    - **SCM UPDATE OPTIONS:** Tick the first three boxes.
- Click **SAVE**
</p>
</details>

## Set up Job Templates

Now you have to create Job Templates like you would for "normal" Jobs.

{{% notice tip %}}
We want to install the **NodeJS app on node3 only**. The Inventory **Workshop Inventory** contains all nodes, but we can limit the nodes using the **LIMIT** field!
{{% /notice %}}

  - Go to the **Templates** view, click the green plus button and choose **Job Template**:

      - **NAME:** Web Infra Deploy

      - **JOB TYPE:** Run

      - **INVENTORY:** Workshop Inventory

      - **PROJECT:** Webops Git Repo

      - **PLAYBOOK:** `rhel/webops/web_infrastructure.yml`

      - **CREDENTIAL:** Workshop Credentials

      - **LIMIT:** node3

      - **OPTIONS:** Enable privilege escalation

  - Click **SAVE**

  - Go to the **Templates** view, click the green plus button and choose **Job Template**:

      - **NAME:** Web App Deploy

      - **JOB TYPE:** Run

      - **INVENTORY:** Workshop Inventory

      - **PROJECT:** Webdev Git Repo

      - **PLAYBOOK:** `rhel/webdev/install_node_app.yml`

      - **CREDENTIALS:** Workshop Credentials

      - **LIMIT:** node3

      - **OPTIONS:** Enable privilege escalation

  - Click **SAVE**

{{% notice tip %}}
If you want to know what the Playbooks look like, check out the Github URL and switch to the appropriate branches.
{{% /notice %}}

## Set up the Workflow

And now you finally set up the Workflow. Workflows are configured in the **Templates** view, you might have noticed you can choose between **Job Template** and **Workflow Template** when adding a template so this is finally making sense.

  - Go to the **Templates** view and click the the green plus button. This time choose **Workflow Template**

      - **NAME:** Deploy Webapplication

      - **ORGANIZATION:** Default

  - Click **SAVE**

  - After saving the template the **Workflow Visualizer** opens to allow you to build a workflow. You can later open the **Workflow Visualizer** again by using the button on the template details page.

  - Click on the **START** button, a new node opens. To the right you can assign an action to the node, you can choose between **JOBS**, **PROJECT SYNC** and **INVENTORY SYNC**.

  - In this lab we’ll link Jobs together, so select the **Web Infra Deploy** Template and click **SELECT**.

  - The node gets annotated with the name of the job. Hover the mouse pointer over the node, you’ll see a red **x**, a green **+** and a blue **chain**-symbol appear.

{{% notice tip %}}
Using the red "x" allows you to remove the node, the green plus lets you add the next node and the chain-symbol links to another node.
{{% /notice %}}

  - Click the green **+** sign

  - Choose **Web App Deploy** as the next Job (you might have to switch to the next page)

  - Leave **Run** set to **On Success**

{{% notice tip %}}
The type allows for more complex workflows. You could lay out different execution paths for successful and for failed Playbook runs.
{{% /notice %}}

  - Click **SELECT**

  - Click **SAVE** in the **WORKFLOW VISUALIZER** view

  - Click **SAVE** in the **Workflow Template** view

{{% notice tip %}}
The **Workflow Visualizer** has options for setting up more advanced workflows, please refer to the [documentation](https://docs.ansible.com/ansible-tower/latest/html/userguide/workflows.html).
{{% /notice %}}

## And Action

Your **Deploy Webapplication** workflow is ready to go, launch it.

  - Click the blue **LAUNCH** button directly or go to the the **Templates** view and launch the **Deploy Webapplication** workflow by clicking the rocket icon.

![jobs view of workflow](../../images/job_workflow.png)

Note how the workflow run is shown in the job view. In contrast to a normal job template job execution this time there is no Playbook output on the right, but a visual representation of the different workflow steps. If you want to look at the actual Jobs behind that, click **DETAILS** in each step. If you want to get back from a details view to the corresponding workflow, just hit your browsers back button.

After the job has finished, check if everything worked fine. In your code-server terminal, run:

```bash
$ curl http://node3/nodejs
```

You should be greeted with a friendly `Hello World`
