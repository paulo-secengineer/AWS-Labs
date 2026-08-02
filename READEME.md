# Lab: AWS VPC Gateway Endpoint for Amazon S3

## Objective
Demonstrate the configuration of an Amazon S3 VPC Gateway Endpoint, allowing EC2 instances to access S3 resources 100% privately through AWS's internal network backbone, without exposing traffic to the public internet.

## Architecture
* **VPC:** `10.1.0.0/16`
* **Subnet:** `10.1.1.0/24` (Public/Private Subnet with IGW for SSH access)
* **Resources:** EC2 Instance (Amazon Linux 2023), IAM Role with `AmazonS3FullAccess`, Gateway Endpoint (`com.amazonaws.us-east-1.s3`).

## Executed Steps
1. Created the VPC, Subnet, and Route Table with SSH access configured via Security Group.
2. Associated an IAM Instance Profile (Role) to the EC2 instance for S3 API permissions without static credentials.
3. Created the VPC Gateway Endpoint (Type Gateway) associated with the Subnet's Route Table.
4. Validated the automatic route entry injected into the Route Table pointing the S3 Prefix List to the Endpoint (`vpce-xxx`).
5. Performed practical validation by uploading objects via AWS CLI directly through the terminal.

## Commands Used

```bash
# List buckets
aws s3 ls

# Create and upload a test file via the private network
echo "Private route via Endpoint working!" > test.txt
aws s3 cp test.txt s3://lab-vpc-endppoint-austriaco-12345/
