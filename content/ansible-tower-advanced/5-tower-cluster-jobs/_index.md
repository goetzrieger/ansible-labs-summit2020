+++
title = "Run a Job in a Cluster"
weight = 5
+++

After boot-strapping the Tower configuration from bottom up you are ready to start a job in your Tower cluster. In one of your Tower nodes web UI’s:

- Open the **Templates** view

- Look for the **Install Apache** Template you created with the script

- Run it by clicking the rocket icon.

At first this is not different from a standard Tower setup. But as this is a cluster of active Tower instances every instance could have run the job.

## So, which Instance did actually run the Job?

There are a couple of ways to find the node that executed the job.

### From the Job Output

The most obvious way is to look up the **EXECUTION NODE** in the details of the job output. If you closed it already or want to look it up later, go to **VIEWS->Jobs** and look up the job in question.

### From the instance groups

In one of the Tower instances web UI under **ADMINISTRATION** go to the **Instance Groups** view. For the `tower` instance group, the **TOTAL JOBS** counter shows the number of finished jobs. If you click **TOTAL JOBS** you’ll get a detailed list of jobs.

To see on what instance a job actually run go back to the **Instance Groups** view. If you click **INSTANCES** under the **tower** group, you will get an overview of the **TOTAL JOBS** each Tower instance in this group executed. Clicking **TOTAL JOBS** for an instance leads to a detailed job list for this instance.

### Using the API

- First find the job ID: In the web UI access **VIEWS→Jobs**

- The jobs names are prefixed with the job ID, example **3 - Install Apache**

{{% notice note %}}
Make sure you choose a job with type "Playbook run".
{{% /notice %}}

- With the ID you can query the API for the instance/node the job was executed on

Bring up the terminal in your VSCode session and run:

{{% notice warning %}}
Replace **\<ID>** with the job ID you want to query!
{{% /notice %}}

```bash
[{{< param "control_prompt" >}} ~]$ curl -s -k -u admin:{{< param "secret_password" >}} https://{{< param "internal_tower1" >}}/api/v2/jobs/<ID>/ | python3 -m json.tool | grep execution_node

    "execution_node": "{{< param "internal_tower1" >}}",
```

{{% notice tip %}}
You can use any method you want to access the API and to display the result, of course. The usage of curl and python was just an example.
{{% /notice %}}

### Via API in the browser

Another way to query the Tower API is using a browser. For example to have a look at the job details (basically what you did above using curl and friends):

{{% notice tip %}}
Note you used the internal hostname above, when using your browser, you have to use the external hostname, of course.
{{% /notice %}}

- Find the job ID

- Now get the job details via the API interface:

  - Login to the API with user `admin` and password `{{< param "secret_password" >}}`: `https://{{< param "external_tower" >}}/api/`

  - Open the URL `https://{{< param "external_tower" >}}/api/v2/jobs/<ID>/` where `<ID>` is the number of the job you just looked up in the UI.

  - Search the page for the string you are interested in, e.g. `execution_node`

{{% notice tip %}}
You can of course query any Tower node.
{{% /notice %}}
