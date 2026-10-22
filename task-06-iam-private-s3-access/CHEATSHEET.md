# Task 6 — IAM and Private S3 Access Cheatsheet

## AWS Environment

- **Region:** `ap-south-1`
- **S3 bucket:** `task6-private-s3-683604924610-1792675265`
- **EC2 instance:** `i-06a150e7bc6dd5ed6`
- **Existing EC2 role:** `EC2-WebServer-Role`
- **Custom policy:** `Task6PrivateS3Policy`
- **Gateway Endpoint:** `vpce-06da5255232202fe5`
- **Route table:** `rtb-06f1a8a68736459f4`

## 1. Verify the Current AWS Identity

```bash
aws sts get-caller-identity
```

**Purpose:** Confirms the AWS identity and account used by the CLI.

The EC2 test returned an assumed-role identity for `EC2-WebServer-Role`.

## 2. Check S3 Block Public Access

```bash
aws s3api get-public-access-block \
  --region ap-south-1 \
  --bucket task6-private-s3-683604924610-1792675265
```

**Expected:** All four settings are `true`.

## 3. List Objects in the Bucket

```bash
aws s3 ls \
  s3://task6-private-s3-683604924610-1792675265/
```

## 4. Create a Test File

```bash
echo "Task 6 IAM role and S3 access test" \
  > /tmp/task6-test.txt
```

## 5. Upload a File

```bash
aws s3 cp \
  /tmp/task6-test.txt \
  s3://task6-private-s3-683604924610-1792675265/
```

## 6. Download and Verify a File

```bash
aws s3 cp \
  s3://task6-private-s3-683604924610-1792675265/task6-test.txt \
  /tmp/downloaded-task6-test.txt
```

Verify the contents:

```bash
cat /tmp/downloaded-task6-test.txt
```

## 7. Inspect the Custom IAM Policy

```bash
aws iam get-policy \
  --policy-arn arn:aws:iam::683604924610:policy/Task6PrivateS3Policy
```

This returns policy metadata, including the default version ID. To inspect the actual permissions, retrieve the policy version:

```bash
aws iam get-policy-version \
  --policy-arn arn:aws:iam::683604924610:policy/Task6PrivateS3Policy \
  --version-id "$(aws iam get-policy \
    --policy-arn arn:aws:iam::683604924610:policy/Task6PrivateS3Policy \
    --query 'Policy.DefaultVersionId' \
    --output text)"
```

## 8. List Policies Attached to the EC2 Role

```bash
aws iam list-attached-role-policies \
  --role-name EC2-WebServer-Role
```

To inspect inline policies:

```bash
aws iam list-role-policies \
  --role-name EC2-WebServer-Role
```

**Important:** Attached managed policies and inline policies are different. Check both when troubleshooting access.

## 9. Verify the S3 Gateway Endpoint

```bash
aws ec2 describe-vpc-endpoints \
  --region ap-south-1 \
  --vpc-endpoint-ids vpce-06da5255232202fe5 \
  --query 'VpcEndpoints[0].[VpcEndpointId,State,VpcEndpointType,ServiceName,RouteTableIds]' \
  --output table
```

**Expected:** The endpoint state is `available`, the type is `Gateway`, and the intended route table is associated.

## 10. Verify the S3 Route

```bash
aws ec2 describe-route-tables \
  --region ap-south-1 \
  --route-table-ids rtb-06f1a8a68736459f4 \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,DestinationPrefixListId,GatewayId,State]' \
  --output table
```

Look for an active S3 prefix-list route targeting the Gateway Endpoint. For an S3 Gateway Endpoint, the destination is normally represented by a prefix-list ID and the target by the VPC endpoint ID.

## Troubleshooting

### `AccessDenied` on S3 operations

1. Check the active identity with `aws sts get-caller-identity`.
2. Inspect the role's attached managed policies and inline policies.
3. Check bucket policies and any applicable explicit denies.
4. Verify the required action and resource ARN.
5. Check the endpoint policy and applicable organization-level controls, if configured.

### File creation returns `Permission denied`

Use a writable temporary directory:

```bash
echo "test" > /tmp/test.txt
```

### Gateway Endpoint is not available

1. Check its state using `describe-vpc-endpoints`.
2. Confirm that the intended route table is associated.
3. Inspect the route table for the S3 prefix-list route.
4. Verify that the bucket is in the expected AWS Region and that the request uses the intended S3 service endpoint.

## Security Reminder

- `EC2-WebServer-Role` still has `AmazonS3ReadOnlyAccess`, so it is not restricted exclusively to the Task 6 bucket.
- `Task6EC2S3Role` and `Task6EC2S3Profile` were created but the profile was not attached to the tested instance.
- The Gateway Endpoint uses its default policy; it is not restricted to the Task 6 bucket by the endpoint policy.
- The EC2 instance remains in a public subnet.
- Do not remove existing role policies without checking their impact on other workloads.

## Interview Summary

**Question: How can an EC2 instance access S3 without storing access keys?**

Attach an IAM role to the EC2 instance through an instance profile. The AWS SDK or CLI obtains temporary credentials from the instance metadata service, and IAM policies determine which S3 actions are allowed.

**Question: What does an S3 Gateway Endpoint do?**

It adds a route for S3 traffic through the AWS network, avoiding the need for a NAT Gateway or Internet Gateway for that matching S3 traffic.

**Question: Does enabling S3 Block Public Access guarantee that only one IAM role can access the bucket?**

No. It helps prevent public access, but private access authorization still depends on IAM policies, bucket policies, and other applicable controls.

**Question: Did this lab prove strict least-privilege access?**

No. S3 operations succeeded, but the instance continued using `EC2-WebServer-Role`, which retains broader permissions. The dedicated role was created but not attached.
