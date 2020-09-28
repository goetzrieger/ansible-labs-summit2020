+++
title = "Introduction"
weight = 1
+++

This chapter will explain the theoretical knowledge to understand the importance of Ansible Collections, what they are and where existing collections can be found and consumed.

## Your Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                 | Inventory name |
| ---------------------| ---------------|
| Ansible Control Host | ansible        |
| Managed Host 1       | node1          |
| Managed Host 2       | node2          |
| Managed Host 2       | node3          |

{{% notice warning %}}
The lab environments in this session have a **\<LABID>** and are separated by numbered **student\<N>** accounts. Follow the instructions given by the lab facilitators to receive the values for **student\<N>** and **\<LABID>**!
{{% /notice %}}

{{% notice tip %}}
On the lab landing page you'll find the URLs you need to access complete with student number and lab ID already filled in.
{{% /notice %}}

## Accessing your Lab Environment

Your main points of contact with the lab is **code-server**, providing a VSCode-experience in your browser.

Now open code-server using the **VS Code access** link from the lab landing page or use this link in your browser by replacing **\<N\>** by your student number and the **\<LABID\>**:

    https://student<N>-code.<LABID>.events.opentlc.com

![code-server login](../../images/vscode-pwd.png)

Use the password **provided on the lab landing page** to login into the code-server web UI, you can close the **Welcome** tab. Now open a new terminal by heading to the menu item **Terminal** at the top of the page and select **New Terminal**. A new section will appear in the lower half of the screen and you will be greeted with a prompt:

![code-server terminal](../../images/vscode-terminal.png)

If unsure how to use code-server, read the [Visual Studio Code Server introduction](../../vscode-intro/), to learn more about how to create and edit files, and to work with the Terminal.

Congrats, you now have a shell terminal on your Ansible control node. From here you run commands or access the other hosts in your lab environment if the lab task requires it.

Now in the terminal become root:

    [student<N>@ansible-1 ~]$ sudo -i

Most prerequisite tasks have already been done for you:

- Ansible software is installed

- `sudo` has been configured on the managed hosts to run commands that require root privileges.

Check Ansible has been installed correctly (your actual Ansible version might differ):

    [root@ansible-1 ~]# ansible --version
    ansible 2.9.6
    [...]

Log out of the root account again:

    [root@ansible-1 ~]# exit
    logout

{{% notice warning %}}
In all subsequent exercises you should work as the student\<N\> user on the control node if not explicitly told differently.
{{% /notice %}}>

# Guide

## Introduction: what are collections and why should I care?

## Understand collections lookup

Ansible Collections use a simple method to define collection namespaces. Anyway, if
your playbook loads collections using the `collections` key and one or more roles, then
the roles will not inherit the collections set by the playbook.

This leads to the main topic of this exercise: roles have an independent collection
loading method based on the role's metadata.
To control collections search for the tasks inside the role, users can choose between
two approaches:

- **Approach 1**: Pass a list of collections in the `collections` field inside the `meta/main.yml` file within the role.
  This will ensure that the collections list searched by the role will have higher priority than
  the collections list in the playbook. Ansible will use the collections list defined inside the role
  even if the playbook that calls the role defines different collections in a separate `collections` keyword entry.

  ```yaml
  # myrole/meta/main.yml
  collections:
    - my_namespace.first_collection
    - my_namespace.second_collection
    - other_namespace.other_collection
  ```

- **Approach 2**: Use the collection fully qualified collection name (FQCN) directly from a task in the role.
  In this way the collection will always be called with its unique FQCN, and override any other
  lookup in the playbook

  ```yaml
  - name: Create an EC2 instance using collection by FQCN
    amazon.aws.ec2:
      key_name: mykey
      instance_type: t2.micro
      image: ami-123456
      wait: yes
      group: webserver
      count: 3
      vpc_subnet_id: subnet-29e63245
      assign_public_ip: yes
  ```

Roles defined **within** a collection always implicitly search their own collection
first, so there is no need to use the `collections` keyword in the role metadata to access modules, plugins, or other roles.

## Automation Hub vs Galaxy

Explain the concept of content repositories.

### Red Hat Automation Hub

It is a service that is provided as part of the Red Hat SaaS Offering. It consists of the location where to discover and download only supported and certified Ansible Content Collections by Red Hat and its Partners. These content collections contain ways to consume automation, how-to-guides to implement them in your infrastructure. The support for Automation Hub is included with Red Hat Automation Platform subscription.

{{% notice tip %}}
Red Hat Automation Hub resides on [https://cloud.redhat.com/ansible/automation-hub](https://cloud.redhat.com/ansible/automation-hub): requires Red Hat customer portal credentials and a valid Red Hat Automation Platform subscription.
{{% /notice %}}

### Certified Content

In the portal of Automation Hub, users have direct access to trusted content collections from Red Hat and Certified Partners. Certified collections are developed, tested, built, delivered, and supported by Red Hat and its Partners. To find more details about the scope of support, check the [Ansible Certified Content FAQ](https://access.redhat.com/articles/4916901),

### Supported Automation

Automation Hub is a one-stop-shop for Ansible content that is backed by support from Red Hat to deliver additional reassurance for customers. Additional supportability claims for these collections may be provided under the "Maintained and Supported By" one of Red Hat Partners.

## Ansible Galaxy

It is the location for wider Ansible community that initially started to provide pre-packaged units of work known as Ansible roles. Roles can be dropped into Ansible Playbooks and immediately put to work. in a recent version of Galaxy started to provide Ansible content collections as well.

Ansible Galaxy resides on [https://galaxy.ansible.com/](https://galaxy.ansible.com/)

----
**Navigation**
<br>
[Next Exercise](../2-using-collections-from-playbooks/)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
