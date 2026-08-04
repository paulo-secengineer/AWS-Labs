# AWS Private Network Architecture: Gateway and Interface Endpoints

## Overview

This repository demonstrates a secure Amazon Web Services (AWS) VPC architecture that enables an Amazon EC2 instance to privately interact with AWS services without traversing the public internet. 

The lab combines two distinct VPC Endpoint types to maintain strict network isolation:
1. **Gateway Endpoint:** Directs Amazon S3 traffic through the VPC Route Table using AWS Prefix Lists.
2. **Interface Endpoint (AWS PrivateLink):** Provisions an Elastic Network Interface (ENI) with a private IP in the subnet for AWS Systems Manager (SSM) Parameter Store access, combined with Private DNS resolution.

---

## Architecture Diagram

```text
               +-------------------------------------------------------------+
               | VPC (10.0.0.0/16)                                           |
               |                                                             |
               |   +-----------------------------------------------------+   |
               |   | Subnet (10.0.1.0/24)                                |   |
               |   |                                                     |   |
               |   |   [ EC2 Instance ]                                  |   |
               |   |          |                                          |   |
               |   |          +---> Private ENI (SSM Endpoint)           |   |
               |   +----------|------------------------------------------+   |
               |              |                                              |
               |              +---> Route Table (Target: vpce-s3)            |
               +--------------|----------------------------------------------+
                              |
                              v
                +----------------------------+
                | AWS Internal Network       |
                +----------------------------+
                 /                          \
                v                            v
      [ Amazon S3 Bucket ]          [ AWS SSM Parameter Store ]

Key Components & Configuration
1. Identity and Access Management (IAM)
An IAM Role was attached to the EC2 instance providing the necessary service-level authorization:

AmazonS3FullAccess: Grants read/write access to S3 resources.

AmazonSSMReadOnlyAccess: Grants read access to parameters stored in SSM Parameter Store.

2. Networking Setup
VPC: 10.0.0.0/16 with enableDnsSupport and enableDnsHostnames attributes set to true.

Subnet: 10.0.1.0/24.

Security Groups:

EC2 Security Group: Inbound SSH (Port 22) restricted to administrator IP. Outbound HTTPS (Port 443) allowed.

Interface Endpoint Security Group: Inbound HTTPS (Port 443) allowed from the EC2 Security Group.

3. VPC Endpoints Configuration
Gateway Endpoint (com.amazonaws.us-east-1.s3):

Type: Gateway

Mechanism: Automatically injects the S3 Prefix List route into the associated VPC Route Table.

Cost: Free tier / no hourly charge.

Interface Endpoint (com.amazonaws.us-east-1.ssm):

Type: Interface (AWS PrivateLink)

Mechanism: Provisions an Elastic Network Interface (ENI) within the subnet.

Private DNS: Enabled to override standard public endpoints (ssm.us-east-1.amazonaws.com) to resolve directly to the ENI private IP (10.0.1.x).

Verification & Testing
All verification steps were performed directly from the EC2 instance via SSH to confirm private routing.

Test 1: S3 Gateway Endpoint Verification
Executes S3 actions via the internal AWS backbone using route table redirection:

Bash
aws s3 ls
Result: Returns a list of S3 buckets successfully without public internet routing.

Test 2: SSM Interface Endpoint Verification
Retrieves a stored parameter using Private DNS and the dedicated PrivateLink ENI:

Bash
aws ssm get-parameter --name "db_pass" --region us-east-1
Result:

JSON
{
    "Parameter": {
        "Name": "db_pass",
        "Type": "String",
        "Value": "12345",
        "Version": 1,
        "LastModifiedDate": "2026-08-04T01:31:41+00:00",
        "ARN": "arn:aws:ssm:us-east-1:248812875626:parameter/db_pass",
        "DataType": "text"
    }
}
Conclusion
This implementation validates the core architectural differences between Gateway and Interface Endpoints in AWS. By leveraging IAM roles for authorization alongside VPC Endpoints for private network transport, resources remain isolated, compliant, and secure from external exposure.
