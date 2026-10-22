\# Task 6: IAM Roles and Private S3 Access Using a VPC Endpoint



\## Overview



This lab demonstrates how an EC2 instance can access an Amazon S3 bucket using an IAM role, without storing AWS access keys on the instance. It also demonstrates S3 Block Public Access and a Gateway VPC Endpoint for private network routing to S3.



\*\*AWS Region:\*\* `ap-south-1` (Mumbai)



\## Architecture



```text

EC2: Develop-machine

Instance: i-06a150e7bc6dd5ed6

&#x20; |

&#x20; | IAM role: EC2-WebServer-Role

&#x20; | Custom policy: Task6PrivateS3Policy

&#x20; |

&#x20; v

S3 Gateway VPC Endpoint

vpce-06da5255232202fe5

&#x20; |

&#x20; | Route table: rtb-06f1a8a68736459f4

&#x20; |

&#x20; v

Private S3 bucket

task6-private-s3-683604924610-1792675265

```



\## Resources Created



| Resource | Name / ID |

|---|---|

| S3 bucket | `task6-private-s3-683604924610-1792675265` |

| IAM policy | `Task6PrivateS3Policy` |

| IAM policy ARN | `arn:aws:iam::683604924610:policy/Task6PrivateS3Policy` |

| IAM role created | `Task6EC2S3Role` |

| Instance profile created | `Task6EC2S3Profile` |

| EC2 instance used for testing | `i-06a150e7bc6dd5ed6` |

| Existing EC2 role used for testing | `EC2-WebServer-Role` |

| VPC | `vpc-01e04109fea01ca2f` |

| S3 Gateway Endpoint | `vpce-06da5255232202fe5` |

| Route table | `rtb-06f1a8a68736459f4` |



\## Implementation



\### 1. Created an S3 bucket



Created a bucket in `ap-south-1` and enabled all four S3 Block Public Access settings:



\- Block public ACLs

\- Ignore public ACLs

\- Block public bucket policies

\- Restrict public bucket access



Verified the settings using `aws s3api get-public-access-block`.



\### 2. Created a custom IAM policy



Created `Task6PrivateS3Policy` with these permissions:



\- `s3:ListBucket` on the specific bucket

\- `s3:GetObject` on objects in the bucket

\- `s3:PutObject` on objects in the bucket

\- `s3:DeleteObject` on objects in the bucket



The policy uses separate bucket and object ARNs to scope the permissions correctly.



\### 3. Created an IAM role and instance profile



Created the `Task6EC2S3Role` role with a trust policy allowing the EC2 service to assume it. Attached `Task6PrivateS3Policy` and created the `Task6EC2S3Profile` instance profile.



The existing EC2 instance already used `EC2-WebServer-Role`, which supports other workloads. To avoid disrupting those workloads, the new profile was not attached to the instance. Instead, the custom policy was attached to the existing role.



\### 4. Verified access from EC2



Connected to the instance using AWS Systems Manager Session Manager.



Verified the active identity using:



```bash

aws sts get-caller-identity

```



The output showed an assumed-role identity for `EC2-WebServer-Role`.



Created a test file, uploaded it to S3, listed the bucket contents, downloaded the file, and verified its contents.



Test files:



\- `task6-test.txt`

\- `task6-endpoint-test.txt`



\### 5. Created an S3 Gateway VPC Endpoint



Created Gateway Endpoint `vpce-06da5255232202fe5` for `com.amazonaws.ap-south-1.s3`.



Associated it with route table `rtb-06f1a8a68736459f4`. Verified that the endpoint state was `available` and that the route table contained an active S3 prefix-list route pointing to the endpoint.



The route table retains its Internet Gateway default route. The EC2 instance remains in a public subnet.



\## Validation Results



| Validation | Result |

|---|---|

| S3 Block Public Access | Passed: all four settings enabled |

| IAM role authentication from EC2 | Passed |

| S3 upload | Passed |

| S3 object listing | Passed |

| S3 download and content verification | Passed |

| Gateway Endpoint state | `available` |

| S3 prefix-list route | Active |

| Strict least-privilege isolation | Not fully demonstrated |



\## Security Findings and Limitations



1\. \*\*Existing role permissions:\*\* `EC2-WebServer-Role` retains `AmazonS3ReadOnlyAccess`, along with CloudWatch, Systems Manager, and a Terraform-related inline policy. Listing other bucket names succeeded during validation. Therefore, the role is not restricted to the Task 6 bucket.

2\. \*\*Dedicated role not attached:\*\* `Task6EC2S3Role` and `Task6EC2S3Profile` were created, but the profile was not attached to the existing instance. The instance continued using `EC2-WebServer-Role`.

3\. \*\*Endpoint policy:\*\* The Gateway Endpoint has its default policy allowing all actions and resources. Effective access is still subject to applicable IAM policies and other controls.

4\. \*\*Network placement:\*\* The EC2 instance remains in a public subnet with an Internet Gateway route. The Gateway Endpoint provides a private route for matching S3 traffic, but does not make the EC2 instance private.

5\. \*\*Traffic-path evidence:\*\* The route-table configuration and successful S3 operations were verified. These checks alone do not prove the path taken by a particular request.



\## Screenshots



The `screenshots/` directory contains the lab evidence:



1\. `01-s3-public-access-block.png`

2\. `02-iam-custom-policy.png`

3\. `03-ec2-role-policies.png`

4\. `04-ec2-s3-upload.png`

5\. `05-s3-gateway-endpoint.png`

6\. `06-s3-endpoint-route.png`

7\. `07-s3-download-verified.png`



\## Key Learnings



\- IAM trust policies determine which principals can assume a role.

\- IAM permission policies determine which actions can be allowed on which resources.

\- EC2 instance profiles provide the mechanism for associating IAM roles with instances.

\- S3 bucket ARNs and object ARNs are different.

\- S3 Block Public Access settings help prevent public exposure.

\- S3 Gateway Endpoints provide private routing to S3 without requiring a NAT Gateway.

\- Existing broad permissions can prevent a least-privilege design from being demonstrated accurately.



