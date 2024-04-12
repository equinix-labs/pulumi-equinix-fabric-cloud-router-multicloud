<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 7: Configuration and Provisioning

## Steps

### 1. Create a YAML file with the required configuration parameters for our Pulumi template

Before deploying our infrastructure, we need to define the necessary configuration parameters. These parameters might include authentication details, resource names, networking settings, and any other custom configurations required by our Pulumi template.

You can use both the CLI and the programming model for your Pulumi configuration. The key-value pairs for any given stack are stored in your project’s stack settings file, which is automatically named `Pulumi.<stack-name>.yaml`. This YAML file acts as the input for our deployment process, ensuring that our infrastructure is provisioned with the correct specifications.

There are several parameters that must can be configured to use this template:

- `accountNum` - (String) your Fabric account number
- `projectId` - (String) your Fabric project UUID
- `gcpProject` - (String) your Google Cloud project name
- `notification_emails` - (Object) List of contact emails
- `awsAccountId` - (String) your Fabric account number
- `awsRegion` - (String) your AWS region. Used by the Equinix provider to create the Fabric connection to AWS
- `aws:region` - (String) your AWS region. Used as a default value by the AWS provider so you don't need to define the region in each resource

The command needed is `pulumi config set <key> [value]`

```shell
$ pulumi config set accountNum 1234
$ pulumi config set projectId 5678
$ pulumi config set gcpProject myGCPProject
$ pulumi config set --path 'notification_emails[0]' example@equinix.com
$ pulumi config set awsAccountId 000111222
$ pulumi config set awsRegion eu-central-1
$ pulumi config set aws:region eu-central-1
```

After that you will have a new file `Pulumi.<stack-name>.yaml` in your project. If for example your stack is called `equinixstack` and the project `workshop`, it will look like this:

```shell
$ cat Pulumi.equinixstack.yaml

config:
  workshop:accountNum: "1234"
  workshop:projectId: "5678"
  workshop:gcpProject: myGCPProject
  workshop:notification_emails:
    - example@equinix.com
  workshop:awsAccountId: "000111222"
  workshop:awsRegion: eu-central-1
  aws:region: eu-central-1
```

### 2. Execute pulumi up to deploy our infrastructure

Now that our project is configured and dependencies are installed, we're ready to deploy our infrastructure using Pulumi. We'll execute the `pulumi up` command, which initiates the deployment process based on the configurations provided in our YAML file. Pulumi will orchestrate the creation of Equinix resources, establish connections with GCP, and configure the networking settings according to our specifications.

```shell
$ pulumi up
```

### 4. Verify that everything is configured correctly upon deployment

Upon completion of the deployment process, we'll validate that our infrastructure has been provisioned correctly. We'll use the Equinix portal to verify that all Equinix resources are created as expected, the connection to GCP and to AWS are established, and the Fabric Cloud Router configurations, including the BGP sessions, are properly configured. This validation step ensures that our infrastructure is ready for use and meets our desired specifications.
