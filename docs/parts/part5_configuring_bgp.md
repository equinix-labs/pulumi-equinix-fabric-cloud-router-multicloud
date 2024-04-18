<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 5: Configuring BGP in Equinix Fabric

In this section, we will add the necessary code to configure BGP (Border Gateway Protocol) in the Equinix Fabric side. This configuration will establish the routing protocol for directing traffic between Equinix Fabric and AWS.

## Steps

### 1. Configuring Routing Protocol

In this step, we'll configure the direct routing protocol for the AWS connection in the Equinix Fabric side. This protocol specifies the direct routing path for traffic between Equinix Fabric and AWS.

```python
# Configure BGP in Equinix Fabric side
aws_routing_protocol_direct = equinix.fabric.RoutingProtocol("AWSRoutingProtocolDirect",
    name="FabricToAWSRoutingProtocolDirect",
    type="DIRECT",
    connection_uuid=aws_fabric_connection.id,
    direct_ipv4=equinix.fabric.RoutingProtocolDirectIpv4Args(
        equinix_iface_ip= dx_setup.private_vif_customer_address
    ),
    opts=pulumi.ResourceOptions(
        ignore_changes=["name"]
    ),
)
```

### 2. Configuring BGP Routing Protocol for AWS Connection

Next, we'll configure the BGP routing protocol for the AWS connection in the Equinix Fabric side. This protocol defines the rules and procedures for exchanging routing information between Equinix Fabric and AWS.


```python
customer_peer_ip = dx_setup.private_vif_amazon_address.apply(lambda s: s.split('/'))[0]

routing_protocol = equinix.fabric.RoutingProtocol("AWSRoutingProtocolBGP",
    connection_uuid=aws_fabric_connection.id,
    name="FabricToAWSRoutingProtocol",
    type="BGP",
    customer_asn='64512',
    bgp_ipv4=equinix.fabric.RoutingProtocolBgpIpv4Args(
        customer_peer_ip=customer_peer_ip
    ),
    bgp_auth_key=dx_setup.private_vif_bgp_auth_key,
    opts=pulumi.ResourceOptions(
        depends_on=[aws_routing_protocol_direct],
        ignore_changes=["name"]
    ),
)
```

> **_Note:_** unlike the `equinix_iface_ip` the `customer_peer_ip` cannot be a cidr notation and we need to remove the subnet mask from the string

By following these steps, we'll successfully configure BGP in the Equinix Fabric side, enabling efficient routing of traffic between Equinix Fabric and AWS.
