# Appendix

## Setup Considerations

Here are a number of things to consider when planning a clustered Tower
deployment:

  - The PostgreSQL database is a central component of the cluster.
    Ansible Tower is not taking care of availabilty, redundancy or
    replication of the database, this has to be configured "outside" of
    Tower.

  - The number of instances in a cluster should always be an **odd
    number** and a minimum number of three is strongly recommended with
    a maximum of 20.

  - RabbitMQ is a core component, so a lot of the requirements are
    dictated by it. Like e.g. the odd node count for quorum…

  - Typical cluster considerations apply: All nodes need to be able to
    reliably connect to each other, stable address/hostname resolution,
    geographically co-located with reliable low-latency connections
    between instances.

  - Remember there is no concept of primary/secondary instance, all
    systems are primary.

### Installing an Ansible Tower Cluster

For initial configuration of a Tower cluster and for adding new
instances the default Ansible installer is used, but the inventory file
needs to be extended. Some important basic concepts:

  - There has to be at least an inventory group named `tower`. We’ll
    cover instance groups later, but keep in mind the nodes in this
    group are responsible for housekeeping tasks like where to launch
    jobs or to process playbook events.

  - If all Tower instances in this group fail, jobs might not run and
    playbook events might not get written. So make sure there are enough
    instances in this group.

  - The database can be installed and configured by the installer by
    adding the host to the `database` group. If the database host is
    provisioned separately, leave the group empty.

### The Installer Inventory File

In this lab a three node Ansible Tower cluster is provided ready to go
as installing it would eat too much of your lab time. It’s pretty
straight forward anyway. The inventory file here is just for giving you
an idea what you are using here and for reference.

> **Tip**
>
> Keep in mind when working with clustered Ansible Tower that the
> database will not be clustered or replicated by the installer. This is
> something you have to take care of yourself.

    [tower]
    tower1.example.com
    tower2.example.com
    tower3.example.com

    [database]
    towerdb.example.com

    [all:vars]
    ansible_become=true

    admin_password='r3dh4t1!'

    pg_host='towerdb.example.com'
    pg_port='5432'

    pg_database='tower'
    pg_username='tower'
    pg_password='r3dh4t1!'

    rabbitmq_port=5672
    rabbitmq_vhost=tower
    rabbitmq_username=tower
    rabbitmq_password='r3dh4t1!'
    rabbitmq_cookie=rabbitmqcookie

> **Warning**
>
> In this lab this has been taken care of, but remember all instances
> have to able to resolve all hostnames and to reach each other\!
