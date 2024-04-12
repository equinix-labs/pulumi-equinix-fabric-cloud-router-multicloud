<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 4: Configuring Direct Connect to AWS VPC

In this part, we will create a new Pulumi Component Resource class to handle the configuration of Direct Connect to an AWS VPC. This component will allow us to efficiently attach Direct Connect to multiple VPCs, enhancing the scalability and flexibility of our infrastructure setup.

## Steps

### 1. Implementing a new Custom Pulumi Component Resource for VPC attachment

Back to our `aws_dx.py` file, we'll create a new Python class called `DirectConnectVPCAttachment`. This class will encapsulate the logic for setting up Direct Connect to an AWS VPC, including the creation of VPN gateways, gateway route propagation and association.

```python
# Class to connect Direct Connect Gateway to a VPC
class AwsDirectConnectVpcAttachment(pulumi.ComponentResource):

    def __init__(self, name, vpc_id, dx_gateway_id, route_table_id, opts=None):
        super().__init__('custom:resource:AwsDirectConnectVpcAttachment', name, None, opts)

        # Create a new VPN Gateway and attach it to the VPC
        vpn_gateway = aws.ec2.VpnGateway(f"{name}-vpn-gateway",
            vpc_id=vpc_id,
            opts=pulumi.ResourceOptions(parent=self)
        )

        # Automatic route propagation between the VPN gateway and the route table of the VPC
        aws.ec2.VpnGatewayRoutePropagation(f"{name}-route-propagation",
            vpn_gateway_id=vpn_gateway.id,
            route_table_id=route_table_id,
            opts=pulumi.ResourceOptions(parent=self)
        )

        # Associate the Direct Connect Gateway with the VPN Gateway
        aws.directconnect.GatewayAssociation(f"{name}-association",
            dx_gateway_id=dx_gateway_id,
            associated_gateway_id=vpn_gateway.id,
            opts=pulumi.ResourceOptions(parent=self)
        )

        # Expose the VPN Gateway ID for other resources to reference
        self.vpn_gateway_id = vpn_gateway.id

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
