<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 1: Setup

To run this workshop you will need access to an Equinix Fabric Account or create a new one following steps below.

> **_Note:_**  You are responsible for the cost of resources created in your Equinix Fabric account while running this workshop.

## Pre-requisites

The following tools will be needed on your local development environment where you will be running most of the commands in this guide.

* [pulumi](https://www.pulumi.com/docs/install/)
* [python](https://www.python.org/downloads/)
* Optional (but recommended): [Google Cloud CLI](https://cloud.google.com/sdk/docs/install?hl=es-419)

## Steps

### 1. Complete previous workshop setup steps 

To get started with this workshop, please ensure you have completed the setup steps outlined in the previous workshop, including the installation of the template, as we'll be working on it here. You can find detailed instructions in Part 1: [Setup](https://equinix-labs.github.io/pulumi-equinix-fabric-cloud-router-workshop/parts/setup/).

### 2. Configure AWS credentials

In addition to the setup steps from the previous workshop, you'll also need to configure your AWS credentials. Please follow the instructions provided in the [AWS Provider Installation & Configuration Guide to set up your AWS credentials](https://www.pulumi.com/registry/packages/aws/installation-configuration/).

### 3. Installing AWS dependency

Furthermore, ensure you have the AWS dependency installed by adding it to the `requirements.txt` file:

```
pulumi_aws>=6.29.0
```

Then, run the following command to install the dependencies:

```shell
pip install -r requirements.txt
```

> **_Note:_** In this workshop, we utilize the classic provider for AWS instead of the native provider unlike GCP. This is because the Direct Connect resources are not yet available in the native provider.

## Discussion

Before proceeding to the next part let's take a few minutes to discuss what we did. Here are some questions to start the discussion.

* How might the setup process differ when integrating additional cloud providers, such as Azure, into the existing infrastructure?