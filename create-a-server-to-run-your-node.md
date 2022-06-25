---
description: >-
  In this section you will learn how to run a server on Google Cloud Provider
  using Compute Engine to run your Umee Node.
---

# Create a Server to run your Node

### Hardware Requirements

To run a Umee Node you will first need a Ubuntu server.&#x20;

The recommended hardware to run an Umee node will vary depending on the use case and desired functionalities of the node. For example, a significant amount of disk space can be required if the node will act as an archive node, i.e. `pruning=nothing` or if the node is a state-sync snapshot provider. In general, we recommend at a minimum the following specifications:

* 2+ vCPU
* 4+ GB RAM
* 120+ GB SSD

### Google Cloud Provider

1\. Go to your Google Cloud Provider (GCP) console and search for _'compute engine'_.&#x20;

![](<.gitbook/assets/image (5).png>)

2\. Once on the Compute Engine screen press on 'Create Instance'.&#x20;

&#x20;

![](<.gitbook/assets/image (6).png>)

3\. In this step we will configure the instance to meet our needs. Name your instance. Select the series as 'E2'. For Machine Type the bare minimum is a e2-medium (2 vCPU, 4GB memory). But you may want to change these setting based on what you are using the node for.

![](<.gitbook/assets/image (2).png>)

4\. Next we will make sure to set the correct boot disk options. If you scroll down you should find a section called boot disk and click 'Change'. H

![](.gitbook/assets/Create\_an\_instance\_–\_Compute\_Engine\_–\_Test-sales-data\_–\_Google\_Cloud\_console.png)

Here make sure to set the operating system to Ubuntu, you can leave the version to the default settings. Set the Boot disk type to SSD Persistent Disk and enter your size. Remember the minimum SSD size is 120 GB.&#x20;

Your server is now set up.&#x20;

