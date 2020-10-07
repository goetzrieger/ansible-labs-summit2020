+++
title = "Use Automation Hub"
weight = 6
+++

## Automation Hub and Ansible Galaxy

## Red Hat Automation Hub

Automation Hub is a service that is provided as part of the Red Hat SaaS offering to subscribers of Ansible Automation Platform. It is a central location where supported and certified Ansible Content Collections by Red Hat and its Partners can be found, downloaded and integrated into your Ansible automation. The support for Automation Hub is included with Red Hat Automation Platform subscription.

{{% notice tip %}}
Red Hat Automation Hub resides on [https://cloud.redhat.com/ansible/automation-hub](https://cloud.redhat.com/ansible/automation-hub) and requires Red Hat customer portal credentials and a valid and active Red Hat Automation Platform subscription.
{{% /notice %}}

### Certified Content

In the portal of Automation Hub, users have direct access to certified content collections from Red Hat and Partners. Certified collections are developed, tested, built, delivered, and supported by Red Hat and its Partners. To find more details about the scope of support, check the [Ansible Certified Content FAQ](https://access.redhat.com/articles/4916901),

### Supported Automation

Automation Hub is a one-stop-shop for Ansible content that is backed by support from Red Hat to deliver additional reassurance for customers. Additional supportability claims for these collections may be provided under the "Maintained and Supported By" one of Red Hat Partners. A list of currently supported content can be found in the [Knowledge base](https://access.redhat.com/articles/3642632).

## Ansible Galaxy

Ansible Galaxy is the upstream location for the Ansible community that initially started to provide pre-packaged units of work known as Ansible roles. Roles can be used from Ansible Playbooks and immediately put to work. in a recent version of Galaxy started to provide Ansible content collections as well.

Ansible Galaxy resides on [https://galaxy.ansible.com/](https://galaxy.ansible.com/)

## Accessing collections from Automation Hub

Ansible collections can be used and downloaded from multiple locations. They can either be downloaded using a requirement file, statically included in the git repository or eventually installed separately in the virtual environment.

{{% notice tip %}}
This is not an exercise you can actually run in this environment because you would need to have an account to Ansible Automation Hub that comes with a subscription of Ansible Automation Platform. It is here for your information.
{{% /notice %}}

## Authenticate Ansible Tower to Automation Hub

### Creating a token

Authenticating Ansible Tower requires a token. It can be achieved using the steps below:

1. Navigate to [https://cloud.redhat.com/ansible/automation-hub/token/](https://cloud.redhat.com/ansible/automation-hub/token/)

   ![Load token|845x550,20%](screenshots/create-token.png)

1. Click **Load Token**.

1. Click **copy icon** to copy the API token to the clipboard.

   ![Copy token|845x550,20%](screenshots/copy-token.png)

### Using authentication token

1. As user admin, navigate to the *Settings l> Jobs*

1. Set **PRIMARY GALAXY SERVER URL** to `https://cloud.redhat.com/api/automation-hub/`

1. Set **PRIMARY GALAXY AUTHENTICATION** URL to `https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token`

1. Set **PRIMARY GALAXY SERVER TOKEN** to <COPIED_TOKEN>

{{% notice tip %}}
It is recommended using Red Hat Automation Hub as primary Galaxy Server URL to ensure using certified and supported content by Red Hat and its partners via Red Hat Ansible Automation subscription.
{{% /notice %}}

  ![test image size](screenshots/token.png)

### Using collections

After authenticating Ansible Tower to access Automation Hub, using a `collections/requirements.yml` file automatically fetches the content collections from Automation Hub as first source.

## Takeaways

- Ansible Galaxy hosts upstream community content.
- The Red Hat Automation Hub provides certified collections that are supported by Red Hat and its Partners. It's a service provided by the Red Hat Ansible Automation Platform Subscription.
- Red Hat Ansible Tower can be configured to authenticate to Red Hat Automation Hub in order to fetch certified and supported content collections that are utilized in a given project.
