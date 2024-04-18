<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 3: Direct Connect configuration

In this part, we will extend our Pulumi template to configure Direct Connect on the AWS side. We'll create a custom Pulumi Component Resource to handle the setup, including accepting the connection, creating a Direct Connect Gateway, and setting up a private virtual interface.

## Steps

### 1. Organizing the Code in a Separate File
To improve readability and facilitate code reuse, we'll create a new file in the root directory named `aws_dx.py` to contain the code for configuring Direct Connect on the AWS side. This will help keep our main Pulumi template file organized and focused on the overall infrastructure setup.

### 2. Implementing a Custom Pulumi Component Resource for AWS

In this step, we'll create a Python class to implement a custom Pulumi Component Resource specifically for configuring Direct Connect on the AWS side (as we did with GCP). This resource will handle accepting the connection, creating a Direct Connect Gateway, and setting up a Private Virtual Interface, required to access an Amazon VPC using private IP addresses.

```python
import pulumi
import pulumi_aws as aws

# Class to set up AWS Direct Connect components
class AwsDirectConnectSetup(pulumi.ComponentResource):
    def __init__(self, name, dx_connection_id, equinix_side_asn, amazon_side_asn, vlan, bgp_authkey=None, opts=None):
        super().__init__('custom:resource:AwsDirectConnectSetup', name, None, opts)

        # Accept Direct Connect connection
        connection_confirmation = aws.directconnect.ConnectionConfirmation(f"{name}-confirmation",
            connection_id=dx_connection_id,
            opts=pulumi.ResourceOptions(parent=self)
        )

        # Create a Direct Connect Gateway
        dx_gateway = aws.directconnect.Gateway(f"{name}-gateway",
            amazon_side_asn=amazon_side_asn,
            opts=pulumi.ResourceOptions(parent=self)
        )

        # Create a private virtual interface
        private_vif = aws.directconnect.PrivateVirtualInterface(f"{name}-private-vif",
            connection_id=connection_confirmation.connection_id,
            vlan=vlan,
            address_family="ipv4",
            bgp_asn=equinix_side_asn,
            dx_gateway_id=dx_gateway.id,
            bgp_auth_key=bgp_authkey,
            # amazon_address=bgp_amazon_address,
            # customer_address=equinix_side_address
            opts=pulumi.ResourceOptions(parent=self)
        )

        # Expose the Direct Connect Gateway ID for other resources to reference
        self.dx_gateway_id = dx_gateway.id

        # Expose the IPv4 BGP peer addresses and BGP authentication key
        self.private_vif_amazon_address = private_vif.amazon_address
        self.private_vif_customer_address = private_vif.customer_address
        self.private_vif_bgp_auth_key = private_vif.bgp_auth_key

        self.register_outputs({})
```

> **_Note:_** We could specify the IP addresses with the `amazon_address` and `customer_address` fields, but this time we omit it and AWS will generate them for us

### 3. Instantiating the Custom Component Resource
In our `__main__.py` pulumi template file, we'll instantiate the new class for AWS Direct Connect. We'll provide the necessary parameters, such as the Direct Connect connection ID, Equinix-side ASN, Amazon-side ASN, VLAN, and optional BGP authentication key, to configure the Direct Connect connection.

First we will need to update the required imports, they should look like this:

```python
import os
import sys
import pulumi
import pulumi_equinix as equinix
from pulumi_google_native.compute import v1 as gcpcompute

sys.path.insert(0, os.getcwd())
from extended_gcp import CloudRouterPeerConfig
import aws_dx
import pulumi_aws as aws
```

Then, we'll move to the end of the file to configure the AWS provider and create an instance of the Direct Connect Setup class

```python
# Configure the default AWS provider
aws_provider = aws.Provider('aws-provider')

# We will need some information from the fabric connection that we added previously.
# in this case, we will need to use apply since it is an Output field where we receive
# the information asynchronously
access_point = aws_fabric_connection.z_side.apply(
    lambda z_side: z_side["access_point"]
)

# Create an instance of the Direct Connect Setup class
dx_setup = aws_dx.AwsDirectConnectSetup('my-dx-setup',
    dx_connection_id=access_point["provider_connection_id"],
    equinix_side_asn=fabric_cloud_router.equinix_asn,
    amazon_side_asn='64512', # default Amazon ASN
    vlan=access_point["link_protocol"]["vlan_tag"],
    opts=pulumi.ResourceOptions(
        depends_on=[aws_fabric_connection],
        provider=aws_provider
    )
)
```
