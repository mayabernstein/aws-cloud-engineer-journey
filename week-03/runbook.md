# Week 3 Runbook: EC2 Deployment and Troubleshooting

## Lab 3.1 — Deploy PropertyLite to EC2

### Objective
Deploy the PropertyLite Flask API to an Amazon EC2 instance and verify that its endpoints are accessible from my laptop.

### Deployment Steps
1. Launched an EC2 instance named `property-api-01` using Amazon Linux 2023 and a Free Tier-eligible instance type.
2. Created an SSH key pair named `training-key` and used it to connect to the instance as `ec2-user`.
3. Configured the security group to allow SSH traffic on port 22 from my IP address and API traffic on port 8080.
4. Used the EC2 user-data bootstrap script to install Python and Flask, create the application files, and start the API.
5. Connected to the instance and checked the application files under `/opt/property-api`.

### Troubleshooting and Resolution
The initial bootstrap script did not correctly create the application files. The cloud-init log showed a warning that the `PYEOF` here-document was not properly terminated. As a result, the application did not start correctly, and connections to port 8080 initially failed.

To resolve the issue, I copied the original `app.py` from my Week 0 PropertyLite project to the EC2 instance and restored it to `/opt/property-api/app.py`. I also created the `rets_property_sample.csv` file using the three sample property records specified in the lab instructions. After ensuring both files were present, I started the Flask application.

### Verification
I tested the API from the EC2 instance and then from my laptop.

- Health endpoint: `curl http://<PUBLIC-IP>:8080/health`
  - Returned `{"status":"ok"}`.
- Property endpoint: `curl http://<PUBLIC-IP>:8080/properties/R100234`
  - Returned the Sacramento property record with listing ID `R100234` and a system price of `450000`.

I captured a screenshot showing both successful `curl` responses as evidence that the API was accessible over the network.

## Lab 3.2 — EBS Snapshot Practice

### Objective
Create an EBS snapshot of the root volume attached to `property-api-01`.

### Steps and Verification
1. Opened the EC2 Volumes page and selected the root volume attached to the instance.
2. Created an EBS snapshot of the volume.
3. Checked the Snapshots page and confirmed that the snapshot status reached **Completed**.

### Key Takeaway
An EBS snapshot is a point-in-time backup of a volume. A new EBS volume can be created from the snapshot to recover data or restore a volume if the original is lost or damaged.

## Debug Lab 3.3 — SSH Connection Troubleshooting

### Objective
Understand how to diagnose an SSH connection timeout when connecting to an EC2 instance.

### Diagnostic Checks
I checked the following components:

1. **Instance state:** Confirmed that the EC2 instance was running.
2. **Public IPv4 address:** Checked the instance's public address before connecting.
3. **Security group:** Verified that inbound SSH traffic was allowed on port 22 from my IP address.
4. **Internet gateway:** Confirmed that an internet gateway was attached to the VPC.
5. **Route table:** Verified that the subnet had a route for `0.0.0.0/0` through the internet gateway.
6. **Network ACL:** Confirmed that inbound and outbound traffic was allowed by the network ACL.
7. **SSH authentication:** Confirmed that SSH authentication succeeded and an interactive session opened.

### Findings and Lessons Learned
The SSH connection ultimately worked, and the checks did not establish that a stale public IP was the cause of an SSH timeout. However, this exercise demonstrated how to troubleshoot connectivity by checking the instance state, security group, public IP address, internet gateway, route table, and network ACL.

A security group's **My IP** rule uses the public IP address detected when the rule is created. If that address changes, the rule may need to be updated to allow SSH from the new address.

## Cleanup

After capturing the API test evidence, I terminated the `property-api-01` EC2 instance through the EC2 console.

I then checked the EC2 Volumes page to verify that the instance's root EBS volume was deleted. The completed EBS snapshot is a separate resource and should be reviewed independently because it may continue to incur storage charges.

## Deliverables

- `week-03/runbook.md` — deployment steps, troubleshooting, snapshot practice, and cleanup documentation.
- `week-03/propertylite-curl-results.png` — screenshot showing successful responses from the `/health` and `/properties/R100234` endpoints.
