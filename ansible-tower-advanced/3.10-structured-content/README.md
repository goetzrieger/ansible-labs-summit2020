# Exercise 10 - OPTIONAL: Well Structured Content Repositories

It’s a common part of the learning curve for Ansible and Ansible Tower:
At some point you will have written so many playbooks that a need for
structure comes up. Where to put the Playbooks, what about the
Templates, Files and so on.

The main recommendations are:

  - Put your content in a version control system like Git or SVN. This
    comes naturally since Ansible code is usually in text form anyway,
    and thus can be managed easily.

  - Group your code by logical units, called
    "[roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)"
    in Ansible.

      - Example: have all code, config templates and files for the
        apache web server in one role, and all code, configuration and
        sql statements for the database in another role. That way the
        code becomes much better to read and handle, and roles can be
        made re-usable and shared between projects, teams or with the
        global community.

Of course, what structure works best in the end depends on the
individual requirements, but we will highlight some common ground rules
which apply to almost all use cases.

The first recommendation is to separate *specific code* from
*reusable/generic code* from *data*:

  - specific code
    Playbooks and their direct dependencies which are not shared outside
    the realm of the project or team.

  - generic code
    All content that will be used across multiple projects.

  - data
    This is mostly the inventory or the inventory scripts and the
    corresponding variables for hosts and groups. In many use cases it
    is advisable to have a dedicated inventory for each life-cycle
    environment.

> **Tip**
>
> Data content files can be in the same Git repository, each in its own
> directory (e.g. dev, test, qa, prod). Alternatively, for example in
> larger environments or with dedicated teams per environment there can
> be one Git repository for each environment. We recommend to put
> special focus on [splitting out host and group
> data](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#splitting-out-host-and-group-specific-data).

> **Caution**
>
> Be careful to *not* have separate code repositories for each
> environment. It would go against the purpose of testing the *same*
> code as you push it through your life-cycle, only varying the data /
> inventory. If you have difficulties to keep the same code throughout
> all your environments we recommend to re-think the structure of cour
> code and what you put into your inventory.

## Example repository

So, let’s get started with an example. The content and repo-structure in
this lab is aligned to the [Ansible best
practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#content-organization)
and is explained in more detail there.

Since we want to store all content in a repository, we will first create
an empty Git repository on our Gitea server. Access the Gitea web UI via

**http://control-&lt;GUID&gt;.rhpds.opentlc.com/gitea**.

Login via the already well known credentials:

  - **User**: git

  - **Password**: r3dh4t1\!

  - In the upper right corner, click on the plus sign, and in the
    opening hover menu click on **New Repository**.

  - Set the **Repository Name**: structured-content

  - Click the button **Create Repository** at the end of the page.

Next we will clone the repository on the control host. To enable you to
work with git on the commandline the SSH key for user **ansible** was
already added to the Gitea user **git**. Next, clone the repository on
the control machine:

    [root@ansible ~]# su - ansible
    [ansible@control ~]$ git clone git@control.example.com:/git/structured-content.git
    [ansible@control ~]$ cd structured-content/

> **Tip**
>
> The repository is currently empty.

you are now going to add some default directories and files:

    [ansible@control structured-content]$ touch {staging,production}

This command creates two inventory files: in this case we have different
stages with different hosts which we keep them in separate inventory
files. Note that those files are right now still empty and need to be
filled with content to work properly.

In the current setup we have two instances. Let’s assume that
`host1.example.com` is part of the staging environment, and
`host2.example.com` is part of the production environment. To reflect
that in the inventory files, edit the two empty inventory files to look
like this:

    [ansible@control structured-content]$ cat staging
    [staging]
    host1.example.com

    [ansible@control structured-content]$ cat production
    [production]
    host2.example.com

Next we add some directories:

  - A directories for host and group variables

  - A **roles** directory where the main part of our automation logic
    will be in.

  - For demonstration purpose we also will add a **library** directory:
    it can contain Ansible code related to a project like custom
    modules, plugins, etc.

<!-- end list -->

    [ansible@control structured-content]$ mkdir -p {group_vars,host_vars,library,roles}

Now to the two roles we’ll use in this example. First we’ll create a
structure where we’ll add content later. This can easily be achieved
with the command `ansible-galaxy`: it creates **role skeletons** with
all appropriate files, directories and so on already in place.

    ansible-galaxy init --offline --init-path=roles security
    ansible-galaxy init --offline --init-path=roles apache

> **Important**
>
> Even if a good role is generally self-explanatory, it still makes
> sense to have proper documentation. The right location to document
> roles is the file **meta/main.yml**.

The roles are empty, so we need to add a few tasks to each. In the last
chapters we set up an Apache webserver and used some security tasks.
Let’s add that code to our roles by editing the two task files:

> **Warning**
>
> If you copy and paste text in VI under a comment (\#) character, Vi
> might (depending on settings) add comment signs to the start of each
> new line. Probably not what you want. Because the role files are being
> created with a comment line after the YAML start (---), make sure to
> delete these lines before pasting the content.

    [ansible@control structured-content]$ cat roles/apache/tasks/main.yml
    ---
    # tasks file for apache
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

    [ansible@control structured-content]$ cat roles/security/tasks/main.yml
    ---
    # tasks file for security
    - name: "HIGH | RHEL-07-010290 | PATCH | The Red Hat Enterprise Linux operating system must not have accounts configured with blank or null passwords."
      replace:
        dest: "{{ item }}"
        follow: true
        regexp: 'nullok ?'
      with_items:
        - /etc/pam.d/system-auth
        - /etc/pam.d/password-auth

    - name: "MEDIUM | RHEL-07-010210 | PATCH | The Red Hat Enterprise Linux operating system must be configured to use the shadow file to store only encrypted representations of passwords."
      lineinfile:
        dest: /etc/login.defs
        regexp: ^#?ENCRYPT_METHOD
        line: "ENCRYPT_METHOD SHA512"

    - name: "SCORED | 1.1.1.2 | PATCH | Remove freevxfs module"
      modprobe:
        name: freevxfs
        state: absent

We also need to create a playbook to call the roles from. This is often
call `site.yml`, since it keeps the main code for the setup of our
environment. Create the file:

    [ansible@control structured-content]$ cat site.yml
    ---
    - name: Execute apache and security roles
      hosts: all

      roles:
        - { role: apache}
        - { role: security }

So we have prepared a basic structure for quite some content - call
`tree` to look at it.

> **Warning**
>
> <details><summary>Click here to see how it should look like!</summary>
> <p>
>
>     [ansible@control structured-content]$ tree
>     .
>     ├── group_vars
>     ├── host_vars
>     ├── library
>     ├── production
>     ├── roles
>     │   ├── apache
>     │   │   ├── defaults
>     │   │   │   └── main.yml
>     │   │   ├── files
>     │   │   ├── handlers
>     │   │   │   └── main.yml
>     │   │   ├── meta
>     │   │   │   └── main.yml
>     │   │   ├── README.md
>     │   │   ├── tasks
>     │   │   │   └── main.yml
>     │   │   ├── templates
>     │   │   ├── tests
>     │   │   │   ├── inventory
>     │   │   │   └── test.yml
>     │   │   └── vars
>     │   │       └── main.yml
>     │   └── security
>     │       ├── defaults
>     │       │   └── main.yml
>     │       ├── files
>     │       ├── handlers
>     │       │   └── main.yml
>     │       ├── meta
>     │       │   └── main.yml
>     │       ├── README.md
>     │       ├── tasks
>     │       │   └── main.yml
>     │       ├── templates
>     │       ├── tests
>     │       │   ├── inventory
>     │       │   └── test.yml
>     │       └── vars
>     │           └── main.yml
>     ├── site.yml
>     └── staging
>
> </p>
> </details>

Since we so far created the code only locally on the control host, we
need to add it to the repository and push it:

    [ansible@control structured-content]$ git add production roles site.yml staging
    [ansible@control structured-content]$ git commit -m "Adding inventories and apache security roles"
    [ansible@control-dff2 structured-content]$ git push

Why not head over to Gitea’s web UI to make sure the files are there?

## Launch it\!

### From the Command Line

The code can now be launched. We start at the command line. Call the
playbook `site.yml` with the appropriate inventory and privilege
escalation:

    [ansible@control structured-content]$ ansible-playbook -i staging site.yml -b

Watch how the changes are done to the target machines. Afterwards,
execute the Playbook against the production stage:

    [ansible@control structured-content]$ ansible-playbook -i production site.yml -b

### From Tower

The new repository needs to be added as project. Feel free to use the
web UI or use **awx** as user **root** like shown below.

    [root@ansible ~]# awx project create -n "Structured Content Repository" \
                        --organization Default \
                        --scm-type git \
                        --scm-url http://control.example.com/gitea/git/structured-content.git \
                        --scm-clean 1 \
                        --scm-update-on-launch 1 \
                        --scm-credential "Gitea Credentials"

Now you’ve created the Project in Tower. Earlier on the commandline
you’ve setup a staged environment by creating and using two different
inventory files. But how can we get the same setup in Tower? We use
another way to define Inventories\! It is possible to use inventory
files provided in a SCM repository as an inventory source. This way we
can use the inventory files we keep in Gitea.

In your Tower web UI, open the **RESOURCES→Inventory** view. Then click
the ![plus(../../images/green_plus.png) button and choose to create a new
**Inventory**. In the next view:

  - **NAME:** Structured Content Inventory

  - Click **SAVE**

  - Click the button **SOURCES** which is now active at the top

  - Click the ![plus](../../images/green_plus.png) button

  - **NAME:** Production

  - **SOURCE:** Pick **Sourced from a Project**

  - Click on **Choose an inventory file**

  - **PROJECT:** Structured Content Repository

  - In the **INVENTORY FILE** drop down menu, pick **production**

  - Click the green **SAVE** button

And now for the staging inventory:

  - Down below in the view, click the ![plus](../../images/green_plus.png)
    button again

  - In the next view, add as **NAME:** Staging

  - **SOURCE:** Pick **Sourced from a Project**

  - Click on **Choose an inventory file**

  - **PROJECT:** Structured Content Repository

  - In the **INVENTORY FILE** drop down menu, pick **staging**

  - Click the green **SAVE** button

  - In the screen below, click the sync button for both sources once so
    that the cloud icon on the left site next to the name of each
    inventory turns green.

To make sure that the project based inventory worked, click on the
**HOSTS** button of the Inventory and make sure the two hosts are listed
and tagged with the respective stages as **RELATED GROUPS**.

Now create a template to execute the `site.yml` against both stages at
the same time.

> **Tip**
>
> Please note that in a real world use case you might want to have
> different templates to address the different stages separatly.

    [root@ansible ~]# awx job_template create -n "Structured Content Execution" \
                        --job-type run -i "Structured Content Inventory" \
                        --project "Structured Content Repository" \
                        --playbook "site.yml" \
                        --credential "Example Credentials" \
                        --become-enabled 1

Now in the Tower web UI go to **RESOURCES→Templates**, launch the
playbook and watch the results.

## Adding External Roles

So far we have only worked with content inside a single repository.
While this drastically reduces complexity already, the largest benefit
is in sharing roles among multiple teams or departments and keeping them
in a central place. In this section we will show how to reference shared
roles in your code and execute them together on your behalf.

In enterprise environments it is common to share roles via internal git
repositories, often one git repository per role. If a role might be
interesting and re-used by the world wide Ansible community, they can be
shared on our central platform [Ansible
Galaxy](https://galaxy.ansible.com/). The advantage of Ansible Galaxy is
that it features basic automatic testing and community ratings to give
the interested users an idea of the quality and reusability of a role.

To use external roles in a project, they need to be referenced in a file
called
[`roles/requirements.yml`](https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#installing-multiple-roles-from-a-file),
for example like this:

    # Import directly from Galaxy
    - src: geerlingguy.nginx
    # Import from a local Git repository
    - src: http://control.example.com/gitea/git/external-role.git
      version: master
      name: external-role_locally

The `requirements.yml` needs to be read - either on the command line by
invoking `ansible-galaxy`, or automatically by Ansible Tower during
project check outs. In both cases the file is read, and the roles are
checked out and stored locally, and the roles can be called in
playbooks. The advantage of Tower here is that it takes care of all that
- including authorization to the Git repo, finding a proper place to
store the role, updating it when needed and so on.

In this example, we will include a role which ships a simple
`index.html` file as template and reloads the apache web server. The
role is already shared in Gitea at
**http://control.example.com/gitea/git/shared-apache-role**.

To include it with the existing structured content, first we have to
create a file called `roles/requirements.yml` and reference the role
there:

> **Warning**
>
> Make sure you work as user **ansible**

    [ansible@control structured-content]$ cat roles/requirements.yml
    - src: http://control.example.com/gitea/git/shared-apache-role.git
      scm: git
      version: master

> **Tip**
>
> In a production environment you may want to change the version to a
> fixed version or tag, to make sure that only tested and verified code
> is checked out and used. But this strongly depends on how you develop
> your code and which branching model you use.

Here we add the source for the role and identify the type of source
control.

Next, we reference the role itself in our playbook. Change the
**site.yml** Playbook to look like this:

    [ansible@control structured-content]$ cat site.yml
    ---
    - name: Execute apache and security roles
      hosts: all

      roles:
        - { role: apache}
        - { role: security }
        - { role: shared-apache-role }

Because Tower uses the Gitea repo, you’ve to add, commit and push the
changes:

    [ansible@control structured-content]$ git add site.yml roles/
    [ansible@control structured-content]$ git commit -m "Add roles/requirements.yml referencing shared role"
    [ansible@control structured-content]$ git push

## Launch in Tower

Just in case, make sure to update the Project in Tower: in the menu at
**RESOURCES**, pick **Projects**, and click on the sync button next to
**Structured Content Repository**.

Afterwards, go to **RESOURCES→Templates** and launch the **Structured
Content Execution** job template. As you will see in the job output, the
external role is called just the way the other roles are called:

    TASK [shared-apache-role : deploy content] *************************************
    changed: [host2.example.com]
    changed: [host1.example.com]

And you are done\! This was quite something to follow through, so let’s
review:

  - You successfully integrated a shared role provided from a central
    source into your automation code.

  - This way, you can limit your automation code to things really
    relevant and individual to the task and your environment, while
    everything generic is consumed from a shared resource.
