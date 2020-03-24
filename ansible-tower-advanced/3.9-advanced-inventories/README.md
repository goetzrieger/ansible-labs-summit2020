# Exercise 9 - Advanced Inventories

In Ansible and Ansible Tower, as you know, everything starts with an
inventory. There are a several methods how inventories can be created,
starting from simple static definitions over importing inventory files
to dynamic and smart inventories.

In real life it’s very common to deal with external dynamic inventory
sources (think cloud…). In this chapter we’ll introduce you to building
dynamic inventories using custom scripts. Another great feature of Tower
to deal with inventories is the Smart Inventory feature which you’ll do
a lab on as well.

## Dynamic Inventories

Quite often just using static inventories will not be enough. You might
be dealing with ever-changing cloud environments or you have to get your
managed systems from a CMDB or other sources of truth.

Tower includes built-in support for syncing dynamic inventory from cloud
sources such as Amazon AWS, Google Compute Engine, among others. Tower
also offers the ability to use custom scripts to pull from your own
inventory source.

In this chapter you’ll get started with dynamic inventories in Tower.
Aside from the build-in sources you can write inventory scripts in any
programming/scripting language that you have installed on the Tower
machine. To keep it easy we’ll use a most simple custom inventory script
using… Bash\! Yes\!

> **Tip**
>
> Don’t get this wrong… we’ve chosen to use Bash to make it as simple as
> possible to show the concepts behind dynamic and custom inventories.
> Usually you’d use Python or some other scripting/programming language.

### The Inventory Source

First you need a source. In **real life** this would be your **cloud
provider, your CMDB or what not**. For the sake of this lab we put a
file into the Github repository you’ve already used to be our source

Use curl to query your external inventory source:

    [root@tower1 ~]# curl https://raw.githubusercontent.com/goetzrieger/ansible-labs-playbooks/master/inventory_list
    {
        "dyngroup":{
            "hosts":[
                "cloud1.cloud.example.com",
                "cloud2.cloud.example.com"
            ],
            "vars":{
                "var1": true
            }
        },
        "_meta":{
            "hostvars":{
                "cloud1.cloud.example.com":{
                    "type":"web"
                },
                "cloud2.cloud.example.com":{
                    "type":"database"
                }
            }
        }
    }

Well, this is handy, the output is already configured as JSON like
Ansible would expect… ;-)

> **Warning**
>
> Okay, seriously, in real life your script would likely get some
> information from your source system, format it as JSON and return the
> data to Tower.

### The Custom Inventory Script

An inventory script has to follow some conventions. It must accept the
**--list** and **--host &lt;hostname&gt;** arguments. When it is called with
**--list**, the script must output a JSON-encoded data containing all
groups and hosts to be managed. When called with **--host &lt;hostname&gt;**
it must return an JSON-formatted hash or dictionary of host variables
(can be empty).

As looping over all hosts and calling the script with **--host** can be
pretty slow, it is possible to return a top level element called
"\_meta" with all of the host variables in one script run. And this is
what we’ll do. So this is our custom inventory script:

    #!/bin/bash

    if [ "$1" == "--list" ] ; then
      curl https://raw.githubusercontent.com/goetzrieger/ansible-labs-playbooks/master/inventory_list
    elif [ "$1" == "--host" ]; then
      echo '{"_meta": {"hostvars": {}}}'
    else
      echo "{ }"
    fi

What it basically does is to return the data collected by curl when
called with **--list** and as the data includes **\_meta** information
about the host variables Ansible will not call it with **--host**. The
curl command is of course the place where your script would get data by
whatever means, format it as proper JSON and return it.

But before we integrate the custom inventory script into our Tower
cluster, it’s a good idea to test it on the command line first:

  - Log in to your bastion host if you don’t still have an SSH session
    open:

  - Create the file dyninv.sh with the content shown above

  - Make the script executable:

<!-- end list -->

    [root@ansible ~]# chmod +x dyninv.sh

  - Execute it:

<!-- end list -->

    [root@ansible ~]# ./dyninv.sh --list
    {
        "dyngroup":{
            "hosts":[
                "cloud1.cloud.example.com",
                "cloud2.cloud.example.com"
            ],
            "vars":{
                "var1": true
            }
        },
        "_meta":{
            "hostvars":{
                "cloud1.cloud.example.com":{
                    "type":"web"
                },
                "cloud2.cloud.example.com":{
                    "type":"database"
                }
            }
        }
    }

The script should output the JSON-formatted output shown above.

As simple as it gets, right? More information can be found
[here](https://docs.ansible.com/ansible/latest/dev_guide/developing_inventory.html)

So now you have a source of (slightly static) dynamic inventory data
(talk about oxymoron…) and a script to fetch and pass it to Tower. Now
you need to get this into Tower.

### Integrate into Tower

The first step is to add the inventory script to Tower:

  - In the web UI, open **RESOURCES→Inventory Scripts**.

  - To create a new custom inventory script, click the
    ![plus](../../images/green_plus.png) button.

  - Fill in the needed data:

      - **NAME:** Cloud Inventory

      - Copy the Bash script from above and paste it into the **CUSTOM
        SCRIPT** field

  - Click **SAVE**

Finally the new inventory script can be used in an actual **Inventory**.

  - Go to **RESOURCES→Inventories**

  - Click the ![plus](../../images/green_plus.png) button and choose
    **Inventory**.

  - **NAME:** Cloud Inventory

  - Click **SAVE**

  - The **SOURCES** button on top becomes active now, click it

  - Click the ![plus](../../images/green_plus.png) to add a new source

  - **NAME:** Cloud Custom Script

  - From the **SOURCE** drop-down choose **Custom Script**

  - Now the dialog for the source opens, your custom script **Cloud
    Inventory** should already be selected in the **CUSTOM INVENTORY
    SCRIPT**.

  - Under **UPDATE OPTIONS** check **Overwrite** and **Overwrite
    Variables**

  - Click **SAVE**

To sync your new source into the inventory:

  - Open the **Cloud Inventory** again

  - Click the **SOURCES** button

  - To the right click the circular arrow to start the sync process for
    your custom source.

  - After the sync has finished click the **HOSTS** button.

You should now see a list of hosts according to what you got from the
curl command above. Click the hosts to make sure the host variables are
there, too.

**What is the take away?**

Using this simple example you have:

  - Created a script to query an inventory source

  - Integrated the script into Tower

  - Populated an inventory using the custom script

## Smart Inventories

You will most likely have inventories from different sources in your
Tower installation. Maybe you have a local CMDB, your virtualization
management and your public cloud provider to query for managed systems.
Imagine you now want to run automation jobs across these inventories on
hosts matching certain search criteria.

This is where Smart Inventory comes in. A Smart Inventory is a
collection of hosts defined by a stored search. Search criteria can be
host attributes (like groups) or facts (such as installed software,
services, hardware or whatever information Ansible pulls). A Smart
Inventory can be viewed like a standard inventory and used with job
runs.

The base rules of a search are:

  - A search typically consists of a field (left-hand side) and a value
    (right-hand side)

  - A colon separates the field that you want to search from the value

  - A search string without a colon is treated as a simple string

### A Simple Smart Inventory

Let’s start with a simple string example. In your Tower web UI, open the
**RESOURCES→Inventories** view. Then click the
![plus](../../images/green_plus.png) button and choose to create a new **Smart
Inventory**. In the next view:

  - **NAME:** Smart Inventory Simple

  - Click the magnifying glass icon next to **SMART HOST FILTER**

  - A window **DYNAMIC HOSTS** opens, here you define the search query

To start with you can just use simple search terms. Try **cloud** or
**internal** as search terms and see what you get after hitting
**ENTER**.

> **Tip**
>
> Search terms are automatically saved so make sure to hit **CLEAR ALL**
> to clear the saved search when testing expressions.

Or what about searching by inventory groups? In the **SEARCH** field
enter **groups.name:dyngroup**. After hitting **ENTER** the hosts from
the dynamic inventory exercise should show up.

When your search returns the expected results, hit **SAVE** for the
**DYNAMIC HOSTS** window and again for the Smart Inventory. Now your
Smart Inventory is usable for executing job templates\!

> **Tip**
>
> You may press the **KEY** button to get a feeling along which fields
> you can search. Browsing through the API becomes necessary to
> understand which related fields have which attributes (e.g. name for
> groups).

### Build Smart Inventories with Facts

As you know Ansible can collect facts from managed hosts to be used in
Playbooks. But before Ansible Tower 3.2 facts where only kept during a
Playbook run. Ansible Tower 3.2 introduced an **integrated fact cache**
to keep host facts for later usage and better performance. This is how
we can use facts in searches for Smart Inventories.

#### Enable Fact Caching

> **Warning**
>
> Fact caching is not enabled by default\!

Fact caching can be enabled for **Templates** and is not enabled by
default. So first we have to enable it. Check
**support1.ewl05.internal** and **support2.ewl05.internal** have no
facts stored:

  - In **RESOURCES→Inventories** open the **Example Inventory** and
    click the **HOSTS** button.

  - Now inspect both hosts by opening the host details and clicking the
    **FACTS** button at the top.

  - For both hosts the **FACTS** field should be empty

Now enable fact caching for the **Install Apache** template:

  - In **RESOURCES→Templates** open the **Install Apache** template.

  - Check the **Use Fact Cache** tick box and click **SAVE**

To gather and save the facts you have to run the job template.

  - In the **Templates** list start **Install Apache** by clicking the
    rocket icon

Now enable fact caching for the **Remote CIS Compliance** template and
run it, too. This way we’ll get cached facts for the isolated hosts.

After you run the templates go back to the host details like you did
above and check the **FACTS** fields for

  - **support1.ewl05.internal** and **support2.ewl05.internal** (from
    the **Example Inventory**)

  - **isosupport1.ewl05.internal** and **isosupport2.ewl05.internal**
    (from the **Remote Inventory**).

The hosts facts should now be populated with a lot of facts.

#### Use Facts in Smart Inventory Searches

Now that we got the facts for the hosts in the facts cache, we can use
facts in our searches.

  - Create a new Smart Inventory named **Smart Inventory Facts**

  - Open the **SMART HOST FILTER** window to enter the search

To search for facts the search field (left side of a search query) has
to start with **ansible\_facts.** followed by the fact. The value is
separated by a colon on the right side.

> **Warning**
>
> No blank between field and value is allowed\!

So what could we search for… start to look at the facts of a host. As
all hosts are Red Hat, searching for the fact
**ansible\_distribution:RedHat** won’t be too exciting. Ah, what the
heck, just try it:

  - Put **ansible\_facts.ansible\_distribution:RedHat** in the search
    field

  - Run the search by hitting **ENTER**

There should be no surprises: All hosts you have run a fact-caching
enabled template on should show up.

#### Nested Facts

A small hint: If a fact is deeper in the structure like this:

    ansible_eth0:
      active: true

The search string would look like this:
**ansible\_facts.ansible\_eth0.active:true**

### Challenge Lab: Facts

So a small challenge: Find out if all hosts have the SElinux mode set to
"enforcing".

  - Find the fact to use by looking at the host facts

  - Create a Smart Inventory

  - Create the proper search string

> **Warning**
>
> <details><summary>Solution below!</summary>
> <p>
> The search string to use is:
> **ansible\_facts.ansible\_selinux.mode:enforcing**
>
>It should return all hosts.
> </p>
> </details>

And to make this a bit more fun:

  - ssh into one of your hosts (say **support1.ewl05.internal**) as
    `root` from your bastion node.

  - Set SELinux to **permissive** (`sudo setenforce 0`)

  - Run the **Install Apache** template again to update the facts.

  - Make sure the host is not showing up in the Smart inventory you just
    created anymore.
