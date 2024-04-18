<!-- See https://squidfunk.github.io/mkdocs-material/reference/ -->
# Part 8: Testing Connectivity between GCP and AWS Instances

In this final part, we'll test the connectivity between the VM instances deployed in Google Cloud Platform (GCP) and Amazon Web Services (AWS). We'll utilize the integrated SSH options provided by the AWS and GCP portals to connect to the instances and perform a ping using the private IP addresses.

## Steps

### 1. Connect to GCP Instance via SSH

- Open the Google Cloud Platform (GCP) Console.
- Navigate to the Compute Engine section.
- Locate the GCP instance named `gcp-vm-instance`.
- Click on the SSH button next to the instance to open a terminal window.

### 2. Connect to AWS Instance via SSH

- Open the Amazon Web Services (AWS) Management Console.
- Navigate to the EC2 Dashboard.
- Locate the AWS instance named `aws-vm-instance`.
- Select the instance and click on the Connect button.
- Follow the instructions to connect to the instance via SSH using the provided terminal command.

### 3. Perform Ping Test

Once connected to both instances via SSH terminals, perform a ping test using the private IP addresses of the instances. Execute the following command in each terminal:

```bash
ping <private_ip_address_of_other_instance>
```

Replace `<private_ip_address_of_other_instance>` with the private IP address of the instance in the other cloud.

### 4. Verify Connectivity

Observe the ping responses to verify connectivity between the instances. Successful ping responses indicate that the private connection between Google Cloud Platform (GCP) and Amazon Web Services (AWS) is operational, allowing for seamless communication between the instances.

By following these steps, we'll confirm the successful establishment of connectivity between the VM instances deployed in GCP and AWS, validating the effectiveness of the private interconnection setup.