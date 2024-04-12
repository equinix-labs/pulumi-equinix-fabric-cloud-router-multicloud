<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 2: Fabric connection to AWS

In this section, we'll start by configuring the connection from Equinix Fabric to AWS. fFor now, we'll add the code in the `__main__.py` file, although later on, we'll separate some logic into other files for better readability and maintenance.

## Steps

### 1. Defining required config parameters

We'll start by adding some configuration variables that we'll need later. Look for the `#Configuration` section at the beginning of the file and add them:

```python
aws_account_id = config.require("awsAccountId")
aws_region = config.get("awsRegion") or "eu-central-1"
```

### 2. Configuring Connection to AWS

Then, we'll move to the end of the file to add the `equinix.fabric.Connection` resource and the `equinix.fabric.get_service_profiles` function that we'll use to obtain the ID of the service profile for AWS Direct Connect.

```python

# Service Profile lookup
aws_service_profile = equinix.fabric.get_service_profiles(
    filter=equinix.fabric.GetServiceProfilesFilterArgs(
        property="/name",
        operator="=",
        values=["AWS Direct Connect"],
    ),
)

# Fabric Connection creation
aws_fabric_connection = equinix.fabric.Connection("connectionToAWS",
    name="pulumi-demo-fcr-aws",
    type="IP_VC",
    notifications=[equinix.fabric.ConnectionNotificationArgs(
        type="ALL", emails=notification_emails)],
    order=equinix.fabric.ConnectionOrderArgs(
        purchase_order_number=purchase_order_num,
    ),
    bandwidth=speed_in_mbps,
    redundancy=equinix.fabric.ConnectionRedundancyArgs(priority="PRIMARY"),
    a_side=equinix.fabric.ConnectionASideArgs(
        access_point=equinix.fabric.ConnectionASideAccessPointArgs(
            type="CLOUD_ROUTER",
            router=equinix.fabric.ConnectionASideAccessPointRouterArgs(
                uuid=fabric_cloud_router.id
            )
        )
    ),
    z_side=equinix.fabric.ConnectionZSideArgs(
        access_point=equinix.fabric.ConnectionZSideAccessPointArgs(
            type="SP",
            authentication_key=aws_account_id,
            seller_region=aws_region,
            profile=equinix.fabric.ConnectionZSideAccessPointProfileArgs(
                type=aws_service_profile.data[0].type,
                uuid=aws_service_profile.data[0].uuid,
            ),
            location=equinix.fabric.ConnectionZSideAccessPointLocationArgs(
                metro_code=equinix_metro,
            ),
        ),
    ),
    opts=pulumi.ResourceOptions(
        depends_on=[fabric_cloud_router],
        ignore_changes=["redundancy", "order"]
    ),
)
```

It's important to note that in this case, we'll need to specify the AWS account ID in the `authentication_key`. Equinix will use it to create the request in that account, and it must be approved later from the AWS side
