# Task 6: IAM Roles and Private S3 Access Using a VPC Endpoint

## Overview

This lab demonstrates how an EC2 instance accesses an Amazon S3 bucket using IAM role-based authentication without storing AWS access keys on the instance. It also covers S3 Block Public Access, IAM permissions, and an S3 Gateway VPC Endpoint.

**AWS Region:** `ap-south-1` (Mumbai)

## Architecture

```text
EC2 Instance
i-06a150e7bc6dd5ed6
        |
        | IAM role: EC2-WebServer-Role
        | Custom policy: Task6PrivateS3Policy
        |
        v
S3 Gateway VPC Endpoint
vpce-06da5255232202fe5
        |
        | Associated route table:
        | rtb-06f1a8a68736459f4
        |
        v
Private S3 Bucket
task6-private-s3-683604924610-1792675265
```

> **Important:** The diagram shows the logical relationship between the resources. The endpoint provides a private route for matching S3 traffic; the route configuration and successful S3 operations alone do not prove the network path taken by an individual request.

## AWS Resources

| Resource | Name / ID |
|---|---|
| Region | `ap-south-1` |
| S3 bucket | `task6-private-s3-683604924610-1792675265` |
| Custom IAM policy | `Task6PrivateS3Policy` |
| Custom policy ARN | `arn:aws:iam::683604924610:policy/Task6PrivateS3Policy` |
| Dedicated IAM role created | `Task6EC2S3Role` |
| Instance profile created | `Task6EC2S3Profile` |
| EC2 instance tested | `i-06a150e7bc6dd5ed6` |
| Existing EC2 role used | `EC2-WebServer-Role` |
| VPC | `vpc-01e04109fea01ca2f` |
| S3 Gateway Endpoint | `vpce-06da5255232202fe5` |
| Route table | `rtb-06f1a8a68736459f4` |

## Implementation Steps

### 1. Created the S3 bucket

Created an S3 bucket in `ap-south-1` and enabled all four S3 Block Public Access settings:

- Block public ACLs
- Ignore public ACLs
- Block public bucket policies
- Restrict public bucket access

Verified the configuration using the S3 API.

### 2. Created a custom IAM policy

Created the `Task6PrivateS3Policy` policy with the following permissions:

- `s3:ListBucket` on the specific bucket
- `s3:GetObject` on objects within the bucket
- `s3:PutObject` on objects within the bucket
- `s3:DeleteObject` on objects within the bucket

The policy separates bucket-level and object-level ARNs to scope permissions correctly.

### 3. Created an IAM role and instance profile

Created the `Task6EC2S3Role` role with a trust policy that allows the EC2 service to assume the role. Attached `Task6PrivateS3Policy` and created the `Task6EC2S3Profile` instance profile.

The existing EC2 instance was already using `EC2-WebServer-Role`, which supports other workloads. To avoid disrupting those workloads, the new instance profile was not attached to the instance. Instead, the custom policy was attached to the existing role.

### 4. Tested S3 access from EC2

Connected to the EC2 instance using AWS Systems Manager Session Manager.

Verified the active AWS identity using:

```bash
aws sts get-caller-identity
```

The output showed an assumed-role identity for `EC2-WebServer-Role`.

Performed the following tests:

- Listed the S3 bucket contents.
- Created a test file.
- Uploaded the file to S3.
- Downloaded the uploaded object.
- Verified the downloaded file contents.

Test filenames included:

- `task6-test.txt`
- `task6-endpoint-test.txt`

### 5. Created an S3 Gateway VPC Endpoint

Created Gateway Endpoint `vpce-06da5255232202fe5` for the S3 service:

`com.amazonaws.ap-south-1.s3`

Associated the endpoint with route table `rtb-06f1a8a68736459f4`.

Verified that:

- The endpoint state was `available`.
- The route table contained an active S3 prefix-list route targeting the Gateway Endpoint.

The route table retains its Internet Gateway default route, and the EC2 instance remains in a public subnet.

## Validation Results

| Validation | Result |
|---|---|
| S3 Block Public Access | Passed — all four settings enabled |
| IAM role authentication from EC2 | Passed |
| S3 object listing | Passed |
| S3 upload | Passed |
| S3 download and content verification | Passed |
| Gateway Endpoint state | `available` |
| S3 prefix-list route | Active |
| Strict least-privilege isolation | Not fully demonstrated |

## Security Findings and Limitations

### 1. Existing IAM role permissions

`EC2-WebServer-Role` retains `AmazonS3ReadOnlyAccess`, along with CloudWatch, Systems Manager, and a Terraform-related inline policy.

Listing other bucket names succeeded during validation. Therefore, the role is not restricted exclusively to the Task 6 bucket.

### 2. Dedicated role not attached

`Task6EC2S3Role` and `Task6EC2S3Profile` were created, but the instance profile was not attached to the existing EC2 instance. The instance continued using `EC2-WebServer-Role`.

### 3. Default endpoint policy

The Gateway Endpoint uses its default policy, which allows all actions and resources at the endpoint-policy layer. Effective access is still governed by applicable IAM policies, bucket policies, and other relevant controls.

### 4. Network placement

The EC2 instance remains in a public subnet with an Internet Gateway route. The Gateway Endpoint provides a private route for matching S3 traffic, but it does not make the EC2 instance private.

### 5. Traffic-path verification

The endpoint state, route-table configuration, and successful S3 operations were verified. These checks alone do not establish that a particular request traversed the Gateway Endpoint.

## Screenshots

The `screenshots/` directory contains the following lab evidence:

1. `01-s3-public-access-block.png`
2. `02-iam-custom-policy.png`
3. `03-ec2-role-policies.png`
4. `04-ec2-s3-upload.png`
5. `05-s3-gateway-endpoint.png`
6. `06-s3-endpoint-route.png`
7. `07-s3-download-verified.png`

## Key Learnings

- IAM trust policies determine which principals can assume a role.
- IAM permission policies define which actions can be allowed on which resources.
- EC2 instance profiles associate IAM roles with EC2 instances.
- S3 bucket ARNs and object ARNs are different.
- S3 Block Public Access helps prevent unintended public exposure.
- S3 Gateway Endpoints provide private routing to S3 without requiring a NAT Gateway for that S3 traffic.
- Existing broad permissions can prevent a least-privilege design from being demonstrated accurately.
- Successful S3 access and endpoint configuration do not, by themselves, prove the path taken by an individual request.

## Conclusion

This lab demonstrated IAM role-based authentication from EC2, S3 object operations, S3 Block Public Access, and Gateway Endpoint configuration.

The tests succeeded, but strict least-privilege isolation was not established because the EC2 instance continued using the existing `EC2-WebServer-Role`. The dedicated `Task6EC2S3Role` was created but not attached to the instance.

**Security note:** Do not store AWS credentials, access keys, or secret values in this repository. Review existing role permissions before changing or removing policies used by other workloads.
