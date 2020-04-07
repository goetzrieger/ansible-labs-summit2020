# Exercise 5 - Run a Job in a Cluster

After boot-strapping the Tower configuration from bottom up you are
ready to start a job in your Tower cluster. In one of the Tower web
UI’s:

  - Open the **Templates** view

  - Look for the **Install Apache** Template you created with the script

  - Run it by clicking the rocket icon.

At first this is not different from a standard Tower setup. But as this
is a cluster of active Tower instances every instance could have run the
job. And the Job output in Tower’s web UI doesn’t tell you where it run,
just the instance group.

## So what Instance run the Job?

In one of the Tower instances web UI under **ADMINISTRATION** go to the
**Instance Groups** view. For the `tower` instance group, the **TOTAL
JOBS** counter shows the number of finished jobs. If you click **TOTAL
JOBS** you’ll get a detailed list of jobs.

To see on what instance a job actually run go back to the **Instance
Groups** view. If you click **INSTANCES** under the **tower** group, you
will get an overview of the **TOTAL JOBS** each Tower instance in this
group executed. Clicking **TOTAL JOBS** for an instance leads to a
detailed job list.

## But I just want to know on which Instance my Job Run!

But it would still be nice to see where a job run (not the other way
round) and to get an idea how jobs are distributed to the available
instances. For this we can utilize the API:

  - First find the job ID: In the web UI access **VIEWS→Jobs**

  - The jobs names are prefixed with the job ID, example **3 - Install
    Apache**

  - With the ID you can query the API for the instance/node the job was
    executed on

Bring up the SSH session on your control host and run:

> **Warning**
>
> Replace &lt;ID&gt; with the job ID you want to query\!

    [root@ansible ~]# curl -s -k -u admin:r3dh4t1! https://tower2.ewl05.internal/api/v2/jobs/*<ID>*/ | python -m json.tool | grep execution_node

        "execution_node": "tower1.ewl05.internal",

> **Tip**
>
> You can use any method you want to access the API and to display the
> result, of course. The usage of curl and python-tool was just for
> example.

## Via API in the browser

Another way to query the Tower API is using a browser. For example to
have a look at the job details (basically what you did above using curl
and friends):

  - Find the job ID

  - Now get the job details via the API interface:

      - Open the URL
        **https://tower.ewl05.sandbox309.opentlc.com/api/v2/jobs/ID/**
        where `<ID>` is the number of the job you just looked up in the
        UI.

      - Search the page for the string you are interested in, e.g.
        `execution_node`

> **Tip**
>
> You can of course query any Tower node.
