\# Task 6 — IAM and Private S3 Access Cheatsheet



\## AWS Environment



\- Region: `ap-south-1`

\- Bucket: `task6-private-s3-683604924610-1792675265`

\- EC2 instance: `i-06a150e7bc6dd5ed6`

\- Existing EC2 role: `EC2-WebServer-Role`

\- Custom policy: `Task6PrivateS3Policy`

\- Gateway Endpoint: `vpce-06da5255232202fe5`

\- Route table: `rtb-06f1a8a68736459f4`



\## 1. Verify the current AWS identity



```bash

aws sts get-caller-identity

```



\## 2. Check S3 Block Public Access



```bash

aws s3api get-public-access-block \\

&#x20; --region ap-south-1 \\

&#x20; --bucket task6-private-s3-683604924610-1792675265

```



Expected: all four public access settings are `true`.



\## 3. List objects in the bucket



```bash

aws s3 ls \\

&#x20; s3://task6-private-s3-683604924610-1792675265/

```



\## 4. Create a test file



```bash

echo "Task 6 IAM role and S3 access test" \\

&#x20; > /tmp/task6-test.txt

```



\## 5. Upload a file



```bash

aws s3 cp /tmp/task6-test.txt \\

&#x20; s3://task6-private-s3-683604924610-1792675265/

```



\## 6. Download a file



```bash

aws s3 cp \\

&#x20; s3://task6-private-s3-683604924610-1792675265/task6-test.txt \\

&#x20; /tmp/downloaded-task6-test.txt

```



Verify:



```bash

cat /tmp/downloaded-task6-test.txt

```



\## 7. Inspect the custom IAM policy



```bash

aws iam get-policy \\

&#x20; --policy-arn arn:aws:iam::683604924610:policy/Task6PrivateS3Policy

```



\## 8. List policies attached to the EC2 role



```bash

aws iam list-attached-role-policies \\

&#x20; --role-name EC2-WebServer-Role

```



\## 9. Verify the S3 Gateway Endpoint



```bash

aws ec2 describe-vpc-endpoints \\

&#x20; --region ap-south-1 \\

&#x20; --vpc-endpoint-ids vpce-06da5255232202fe5

```



\## 10. Verify the S3 route



```bash

aws ec2 describe-route-tables \\

&#x20; --region ap-south-1 \\

&#x20; --route-table-ids rtb-06f1a8a68736459f4 \\

&#x20; --query 'RouteTables\[0].Routes\[\*].\[DestinationCidrBlock,DestinationPrefixListId,GatewayId,State]' \\

&#x20; --output table

```



Look for the active S3 prefix-list route whose target is the Gateway Endpoint.



\## Troubleshooting



\### AccessDenied on S3 operations



\- Check the active identity with `aws sts get-caller-identity`.

\- Inspect the role's attached and inline policies.

\- Check bucket policies and any applicable explicit denies.

\- Confirm the action and resource ARN match.



\### File creation returns Permission denied



Use a writable temporary directory:



```bash

echo "test" > /tmp/test.txt

```



\### Gateway Endpoint is not available



\- Check its state using `describe-vpc-endpoints`.

\- Confirm the intended route table is associated.

\- Inspect the route table for the S3 prefix-list route.



\## Security Reminder



`EC2-WebServer-Role` still has `AmazonS3ReadOnlyAccess`. The role is not strictly limited to the Task 6 bucket. The dedicated `Task6EC2S3Role` was created but not attached to the existing instance.



Do not remove existing role policies without checking their impact on other workloads.



