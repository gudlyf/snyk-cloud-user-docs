# Snyk Broker

{% hint style="info" %}
**Feature availability**\
Snyk Broker is available with Enterprise plans. See [pricing plans](https://snyk.io/plans/) for more details.
{% endhint %}

{% hint style="warning" %}
**Multi-tenant settings for EU and AU**\
When you set up Broker, Code Agent, or both for use in EU or AU Multi-tenant environments, additional environment variables with the specific URLs are required.\
Example:  `-e BROKER_SERVER_URL=https://broker.eu.snyk.io`\
For the URLs, see [EU and AU account datacenter creation](https://docs.snyk.io/snyk-processes/data-residency-at-snyk#eu-and-au-datacenter-account-creation).
{% endhint %}

## Introduction to Snyk Broker

Snyk Broker is an open-source tool that can act as a proxy between Snyk and special integrations including:

* Your Source Code Management (SCM) system on-premise platforms
* Your publicly-accessible Git-based repositories, allowing you to view and control Snyk activity in those repositories for increased data security
* Your on-premise Jira installation or JFrog Artifactory installation
* Network-restricted [container registries](snyk-broker-container-registry-agent/)
* [Infrastructure as code (IaC) configuration](snyk-broker-infrastructure-as-code-detection/) files using Snyk IaC located on private Git-based repositories

Snyk Broker is designed to connect Snyk products to self-hosted integrations that are not publicly accessible from the internet. Snyk Broker also allows you to do the following:

* Control Snyk access to your network, by limiting the files to which Snyk has access and the actions that Snyk can perform.
* Manage a fixed private IP for your integration, targeting the Broker.

The Snyk Broker project is hosted at [GitHub](https://github.com/snyk/broker) and published as a set of Docker images for specific integrations.

## Components of Snyk Broker

Snyk Broker has a Server and a Client, basic components that are the same across all integrations:

* **Broker Server** - running on the Snyk SaaS backend\
  The Broker Server is provided by Snyk, and no installation is required on your part.
* **Broker Client** - a [Docker image](https://hub.docker.com/r/snyk/broker/) deployed in your infrastructure

For information about components that are required for using Snyk Code Agent, Snyk Container Registry Agent, and Snyk Broker for Infrastructure as Code, see [Define your Broker deployment components](https://docs.snyk.io/integrations/snyk-broker/set-up-snyk-broker/prepare-snyk-broker-for-deployment#define-your-broker-deployment-components).

The diagram that follows illustrates how the basic components operate.

* All data, both in transit and at rest, is encrypted.
* Communication between the Client and the Server takes place over a secure WebSocket connection.
* There is no need to open incoming ports since the communication is initiated outbound.
* Once the connection is initiated, the WebSocket connection is bi-directional.

<figure><img src="../../.gitbook/assets/Snyk Broker diagram.png" alt="Snyk Broker WebSocket initiated by Client over HTTPS"><figcaption><p>Snyk Broker WebSocket initiated by Client over HTTPS</p></figcaption></figure>

## Using inbound and outbound connections with Snyk Broker

The Broker client runs within your internal network, keeping sensitive data such as Git tokens within the network perimeter.

### Inbound connection from Snyk to the Broker Client

There is no direct inbound connection from Snyk to the Broker Client.

The Broker Client makes an outbound connection to [https://broker.snyk.io](https://broker.snyk.io), which establishes a WebSocket connection. This allows communication with the Broker Server.

Thus there is no need to allowlist a Snyk IP address. Instead you can allow the Broker Client IP/port.

### Outbound connection from the Broker Client

The Broker Client initiates the outbound connection to establish the WebSocket.

After the WebSocket connection initiated by the Broker Client is established, Snyk can send inbound requests to the Broker Client via the WebSocket.

Thus you do not need to allow inbound connections to the Broker Client from Snyk-specific IP addresses or other external IP addresses.

## **Approved data list for Snyk Broker**

The Broker Client maintains an approved data list for inbound and outbound data requests. Only data on this approved list may be requested. This narrows the access permissions to the absolute minimum required for Snyk to actively monitor a repository.

### Inbound requests allowed

For Snyk Open Source:

* Snyk.io is allowed to fetch and view only dependency manifest files and the `.snyk` policy file.
* No other source code is viewed, extracted, or modified.
* You may check in additional `.snyk` files to support the Snyk patch mechanism and for any ignore instructions included in your vulnerability policy.

Snyk Code and Snyk IaC need access to the entire repository.

See [How Snyk handles your data](../../more-info/how-snyk-handles-your-data.md) for more details. For information about using Snyk Broker with Snyk Container and requirements for open source, code analysis, and IaC, see [Using Snyk Broker to scan your code](./#using-snyk-broker-to-scan-your-code).

### Outbound requests allowed

When you configure your Broker Client setup, Git repository webhooks are set to enable automatic Snyk scans that are triggered when your developers submit new pull requests or merge events.&#x20;

Webhook notifications are delivered to Snyk via the Broker Client only for events relevant to Snyk actions: push to branch and open pull request _**and only when**_ the event data also includes a dependency manifest file or a .`snyk` policy file.

### Default approved data list and `accept.json` file

Because of the limitations of the default approved data list, if you want to scan Infrastructure as Code files with Snyk Broker, you must [add and configure an `accept.json`](snyk-broker-infrastructure-as-code-detection/) file in your Broker deployment.

To learn more about the approved data list and the `accept.json` file, see [Install and configure the Snyk Broker Client](set-up-snyk-broker/how-to-install-and-configure-your-snyk-broker-client.md), _Custom approved-listing filter_ section.

## **Using Snyk Broker to scan your code**

To use **Snyk Open Source** with Snyk Broker, you need only the Broker Server and the Broker Client components.

To scan other types of code with Snyk Broker, you must add a component or configurations and add parameters to the Broker Client setup:

* **Snyk Code** – add the [**Code Agent** ](snyk-broker-code-agent/)component to enable Snyk Code analysis of repositories in SCMs that are integrated through Snyk Broker.
* **Snyk Container** – add the [**Container Registry Agent**](snyk-broker-container-registry-agent/) to enable the connection to network-restricted container registries container registries and the analysis of container images.
* **Snyk Infrastructure as Code** – configure the [**`accept.json`** file with additional parameters](snyk-broker-infrastructure-as-code-detection/) to detect and analyze Terraform, CloudFormation, and Kubernetes configuration files through Snyk Broker.

## **Integrations with Snyk Broker**

Snyk Broker currently integrates with the following Git repository systems:

* [GitHub](../git-repository-scm-integrations/github-integration.md) and [GitHub Enterprise](../git-repository-scm-integrations/github-enterprise-integration.md) (Cloud and On-prem)
* [GitLab](../git-repository-scm-integrations/gitlab-integration.md) (Cloud and On-prem)
* [Bitbucket Server / Data Center](../git-repository-scm-integrations/bitbucket-data-center-server-integration.md) (On-prem)
* [Azure Repos](../git-repository-scm-integrations/azure-repos-integration.md) (Cloud and On-prem)

In addition, Snyk Broker integrates with [Jira Server/Jira Data Center](../notifications-ticketing-system-integrations/jira.md), [JFrog Artifactory](../private-registry-integrations/artifactory-registry-setup.md), and [Nexus Repository Manager](../private-registry-integrations/nexus-repo-manager-setup.md).

With the Container Registry Agent, Snyk Broker also connects to all [Snyk-supported container registries](snyk-broker-container-registry-agent/).

## Common questions about Snyk Broker

**How often is Snyk Broker updated?**\
Snyk Broker is updated any time there are new features available and when there are fixes.

**How often is Snyk Broker checked for vulnerabilities?**\
The Snyk Broker application and images are being tested on a daily basis for vulnerabilities.

**What is the SLA to fix vulnerabilities?**\
There is a 14-day SLA for fixing high vulnerabilities and five-day SLA for fixing critical vulnerabilities in public images.

