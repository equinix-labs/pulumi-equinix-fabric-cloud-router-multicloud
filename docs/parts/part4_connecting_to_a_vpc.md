<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 4: Configuring Direct Connect to AWS VPC

In this part, we will create a new Pulumi Component Resource class to handle the configuration of Direct Connect to an AWS VPC. This component will allow us to efficiently attach Direct Connect to multiple VPCs, enhancing the scalability and flexibility of our infrastructure setup.

## Steps

### 1. Implementing a new Custom Pulumi Component Resource for VPC attachment

Back to our `aws_dx.py` file, we'll create a new Python class called `DirectConnectVPCAttachment`. This class will encapsulate the logic for setting up Direct Connect to an AWS VPC, including the creation of VPN gateways, customer gateways, VPN connections, and virtual private gateways, and of course the attachment.

```python
import pulumi
from pulumi_aws import ec2, directconnect

class DirectConnectVPCAttachment(pulumi.ComponentResource):
    def __init__(self, name: str,
                 vpc_id: pulumi.Input[str],
                 direct_connect_gateway_id: pulumi.Input[str],
                 *,
                 opts: pulumi.ResourceOptions = None):
        super().__init__('custom:resource:DirectConnectVPCAttachment', name, {}, opts)

        # Create a VPN Gateway
        vpn_gateway = ec2.VpnGateway("vpn-gateway",
            tags={"Name": "vpn-gateway"},
            )

        # Attach the VPN Gateway to the VPC
        vpc_attachment = ec2.VpnGatewayAttachment("vpc-attachment",
            vpc_id=vpc_id,
            vpn_gateway_id=vpn_gateway.id,
            )

        # Create a Customer Gateway
        customer_gateway = ec2.CustomerGateway("customer-gateway",
            bgp_asn=65000,
            ip_address="172.31.0.1",
            tags={"Name": "customer-gateway"},
            )

        # Create a VPN Connection
        vpn_connection = ec2.VpnConnection("vpn-connection",
            customer_gateway_id=customer_gateway.id,
            vpn_gateway_id=vpn_gateway.id,
            type="ipsec.1",
            )

        # Create a Virtual Private Gateway
        virtual_private_gateway = ec2.VpnGateway("virtual-private-gateway",
            amazon_side_asn=64512,
            tags={"Name": "virtual-private-gateway"},
            )

        # Attach the Virtual Private Gateway to the VPC
        vpn_gateway_attachment = ec2.VpnGatewayAttachment("vpn-gateway-attachment",
            vpc_id=vpc_id,
            vpn_gateway_id=virtual_private_gateway.id,
            )

        # Register the custom resource with Pulumi state
        self.register_outputs({})
```

By encapsulating this logic within a reusable component, we can easily attach Direct Connect to multiple VPCs as needed.

### 2. Instantiating the Custom Component Resource

In our `__main__.py` pulumi template file, we'll instantiate the new class We'll provide the necessary parameters, such as the VPC ID, Direct Connect Gateway ID, and the Route Table ID.

```python
# If no default VPC exists, the provider creates a new default VPC
vpc_default = aws.ec2.DefaultVpc("default", 
    tags={
        "Name": "Default VPC"
    }
)

# Create an instance of the Direct Connect VPC attachment using the VPC ID
dx_vpc_attachment = aws_dx.AwsDirectConnectVpcAttachment('my-dx-vpc-attachment',
    vpc_id=vpc_default.id,
    dx_gateway_id=dx_setup.dx_gateway_id,
    route_table_id=vpc_default.default_route_table_id,
    opts=pulumi.ResourceOptions(
        provider=aws_provider
    )
)
```

> **_Note:_** For this example we have used `aws.ec2.DefaultVpc`, This resource has special caveats to be aware of when using it. if a default VPC exists, this provider does not create this resource, but instead “adopts” it into management. If no default VPC exists, the provider creates a new default VPC. Please read the documentation before using this resource.
