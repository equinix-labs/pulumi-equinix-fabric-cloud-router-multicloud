<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 6: Deploying VM Instances in Both Clouds

In this part, we'll deploy VM instances in both Google Cloud Platform (GCP) and Amazon Web Services (AWS), configuring firewall rules to allow private traffic between the respective VPCs and enabling SSH connections. This setup will demonstrate the private connection between the two clouds, allowing for seamless communication between resources.

## Steps

### 1. Deploying VM Instances in Google Cloud Platform (GCP)

In this step, we'll deploy a VM instance in Google Cloud Platform (GCP) to demonstrate the private connection between GCP and AWS. Along with the VM, we'll configure firewall rules to allow traffic from the AWS VPC and enable SSH connections.

```python
# GCP Instance Configuration
gcp_instance = gcpcompute.Instance('gcp-vm-instance',
    project=gcp_project,
    name='gcp-vm',
    machine_type='e2-micro',
    zone=f"{gcp_region}-a",
    disks=[
        gcpcompute.AttachedDiskArgs(
            boot=True,
            initialize_params=gcpcompute.AttachedDiskInitializeParamsArgs(
                source_image="projects/debian-cloud/global/images/family/debian-11",
            )
        ),
    ],
    network_interfaces=[gcpcompute.NetworkInterfaceArgs(
        network=gcp_vpc_network.self_link,
    )]
)

# GCP Firewall Rules
gcpcompute.Firewall('gcp-firewall-rule-aws',
    network=gcp_vpc_network.self_link,
    allowed=[
        gcpcompute.FirewallAllowedItemArgs(ip_protocol="tcp", ports=["0-65535"]),
        gcpcompute.FirewallAllowedItemArgs(ip_protocol="udp", ports=["0-65535"]),
        gcpcompute.FirewallAllowedItemArgs(ip_protocol="icmp")
    ],
    source_ranges=[vpc_default.cidr_block], # CIDR block of the AWS VPC
)

gcpcompute.Firewall('gcp-firewall-rule-ssh',
    network=gcp_vpc_network.self_link,
    allowed=[
        gcpcompute.FirewallAllowedItemArgs(ip_protocol="tcp", ports=["0-65535"]),
        gcpcompute.FirewallAllowedItemArgs(ip_protocol="udp", ports=["0-65535"])
    ],
    source_ranges=["0.0.0.0/0"],
)
```

### 2. Deploying VM Instances in Amazon Web Services (AWS)

Next, we'll deploy a VM instance in Amazon Web Services (AWS) to complement the connectivity demonstration. We'll set up a security group to allow traffic from the GCP VPC and enable SSH connections.

```python
# AWS Security Group Configuration
aws_security_group = aws.ec2.SecurityGroup('aws-security-group',
    vpc_id=vpc_default.id,
    ingress=[aws.ec2.SecurityGroupIngressArgs(
            from_port=0,
            to_port=0,
            protocol='-1',  # `-1` indicates all protocols
            cidr_blocks=[gcp_cidr_block],  # Block of the GCP VPC
        ),
        aws.ec2.SecurityGroupIngressArgs(
            from_port=22,
            to_port=22,
            protocol='tcp',
            cidr_blocks=["0.0.0.0/0"], # Allows SSH connection
        )],
    egress=[aws.ec2.SecurityGroupEgressArgs(
        from_port=0,
        to_port=0,
        protocol='-1',
        cidr_blocks=["0.0.0.0/0"], # Allows all outbound traffic
    )]
)

# AWS EC2 Instance
aws_instance = aws.ec2.Instance('aws-vm-instance',
    instance_type='t2.micro',
    ami=pulumi.Output.all().apply(lambda args: get_ami_id(args)),
    vpc_security_group_ids=[aws_security_group.id]
)
```

Certainly! Here's the final step:

### 3. Exporting VM Instance Private IPs

Finally, we'll export the private IP addresses of both the GCP and AWS instances. With these exported values, we'll be able to ping each instance from the other once connected to each machine to demonstrate the interconnectivity between GCP and AWS.

```python
# Export GCP and AWS instance private IPs
pulumi.export('gcp_instance_private_ip', gcp_instance.network_interfaces[0].network_ip)
pulumi.export('aws_instance_private_ip', aws_instance.private_ip)
```
