---
icon: lock-keyhole
---

# IAM

* **User:** can belong to multiple group or don't have to belong to a group (not best practice). **IAM user** represents the person or service that  interacts with AWS.
* **Groups:** can only contain users not other groups.
* **Roles:** for EC2, Lambda, Cloud9, or other service / application to perform action on the account. **Trust policies** are only associated with **IAM Roles**  to specify the trusted entities that can assume the role.
* **Policies:** JSON-format documents that outline permissions for AWS identities (such as groups, users, and roles) and can be divided into two main types: **Identity-based** and **Resource-based** policies.

:exclamation: **Identity-based:** policies attached to users, groups, roles to grant THEM permission to perform actions on AWS resources.&#x20;

* **AWS Managed Policies**: These are predefined policies created and maintained by AWS, when you create a Cloud9 role, AWS may automatically attach an AWS managed policy for you.
* **Customer Managed Policies**: These are policies that you create and manage yourself, you have rollback and version control.

:exclamation: **Resource-based:** policies attached directly to resource policies like S3 bucket or Lambda function, AWSServiceRoleForAWSCloud9. They define **Trust Entities that can assume the role.**

* **Trust Policies**: This is a specific type of resource-based policy that is attached to an IAM role. It defines the **trust relationships** and specifies which identities (users, services, or accounts) are allowed to assume the role.
* **IAM Credentials report:** lists all account's users and the status of their credentials. Account level report.
* **IAM Access Advisor:** breaks down the permissions granted to a user, role, or group and when those services were last accessed. User level report that shows the **"Last Accessed"** time for each service, indicating when that service was last used by the entity.
* IAM Access Analyser:&#x20;
* **Best practice:** use temporary security credentials (for IAM roles) instead of generated **access keys.**
  * Delete any access keys to your AWS account root user.
* **Shared responsibility model for IAM:** User is responsible for MFA on accounts, appropriate permissions and policies.
*   **Best practice:** for root and IAM users to use both password + MFA, rotate access keys.

    * AWS provides guidelines for access key rotation every 90 days.

    <figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/IAM.png" alt=""><figcaption></figcaption></figure>

## Policy Types

* **WS Managed Policy** = Simple, broad, AWS-handled 😇
* **Customer Managed Policy** = Flexible, reusable, version-controlled 🔧
* **Inline Policy** = Specific, one-to-one, tightly bound 2KB 🔒

***

**1. AWS Managed Policy 📦🔧**

* **Maintained by AWS**: AWS keeps these up to date, adding new APIs and features as they are released.
* **Predefined, general-purpose permissions**: Great for common tasks like `AdministratorAccess`, `PowerUserAccess`, or `ReadOnlyAccess`.
* **Automatically updated**: Whenever AWS adds new functionality, these policies are updated automatically without any action from you.
* **No version control**: You can’t roll back to a previous version since AWS is responsible for updates.

📝 _Use case_: These are great for quick, broad permissions (e.g., administrators) where you don’t need granular control but still want reliable, managed permissions. ⚡

**2. Customer Managed Policy 🛠️👥**

* **Created and managed by the customer**: You define the permissions exactly how you need.
* **Reusable**: You can apply it to multiple IAM entities (users, groups, or roles) across accounts.
* **Version controlled & rollback**: Keep track of policy changes and roll back to previous versions if needed. Useful for maintaining and auditing access over time.
* **More flexible**: Unlike AWS Managed Policies, you can add, remove, or modify permissions as you see fit.

🎉 _Use case_: When you need fine-grained, customizable access control for your specific use case. Best when managing permissions at scale across multiple users and roles. 🌍

**3. Inline Policy 📝🔒**

* **Strict 1-to-1 relationship**: Inline policies are attached directly to an IAM principal (user, group, or role) and can’t be reused across others.
* **No versioning**: Once created, you cannot keep track of changes, and you can’t roll back to previous versions. Once it's deleted, it's gone forever.
* **Max size of 2 KB**: These are ideal for simpler, smaller permission sets due to size constraints.
* **Policy is deleted if the IAM principal is deleted**: The inline policy is permanently tied to the user or role it’s attached to. _If the principal is deleted, the inline policy goes with it._

🛠️ _Use case_: Great for specific, tightly scoped permissions that are tied to a single pricipal. Think of it like a **personal** permission set — _one user, one policy_. 🤖

## IAM Policies ( Group, Inline)

* Policies can be attached to IAM users, groups, or roles. Give broad permissions.
* Groups <mark style="color:red;">can’t be nested</mark>, can contain users not groups.
* Granular principle of <mark style="color:red;">least privilege</mark>. Never grant more permissions than needed.
* Users inherit policies from groups they're in.
* AWS evaluates <mark style="color:red;">permissions based on the most restrictive policy</mark>. If the IAM policy allows access but the bucket policy denies it, the user will be denied access.
* AWS IAM policy evaluation, <mark style="color:red;">explicit</mark> <mark style="color:red;">DENY  always takes precedence over allows.</mark>
* <mark style="color:red;">Default Deny,</mark> if no allow or deny is found, access is denied by default.
* A "deny by default" model for permission

```mermaid
graph LR
  A[Decision starts at 'Deny'] --> B([Evaluate All Available Policies])
  B --> C{Explicit Deny? }
  C -->|Yes| D[Final decison=Deny]
  C -->|No| F{Explicit Allow?}
 
 
  F -->|Yes| K[Final decision=Allow]
  F -->|No| E[Final decison=Deny]





```

* **Group** policies allow for easy control of permissions for all users in that group simultaneously.
* **Inline** policies - attached directly to a single principal(user, group, or role), no version control, rollback. Good practice for Highly Specific Access or Temp Access requirements. Policies maintain a strict one-to-one relationship between a policy and an identity. They are deleted when you delete the identity.

### Achieving Fine Grained Access / Layered security

When evaluating if a principal can perform an action IAM policies will be evaluated in combination with S3 bucket policies attached to resource. Such as evaluate both IAM and bucket policies.

:x: For anythis the principle of <mark style="background-color:red;">**explicit deny overriding allow**</mark><mark style="background-color:red;">.</mark>

1. &#x20;IAM policy for Devs group can grant explicit "Allow" access to the S3 bucket action (`"Action": "s3:GetObject"`for all objects, but Bucket policy (_also a JSON documents specifies permissions at the bucket level_) can enforce additional restrictions on specific objects / certain documents with explicit "Deny" = <mark style="color:red;">**DENY**</mark>

<details>

<summary>Layered Access Control</summary>

Access to the sensitive documents will be denied due to the explicit deny in the bucket policy.

```json
//Bucket policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": [
        "arn:aws:s3:::mybucketname/payemnts/sensitive-document-1.txt",
        "arn:aws:s3:::mybucketname/payemnts/sensitive-document-2.txt"
      ]
    },
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mybucketname/*"
    }
  ]
}

```



</details>



2. IAM Policy for XYZRole is empty (meaning it does not explicitly allow or deny any actions in S3) + Bucket policy with explicit RWX permission for XYZRole configured with explicit "Allow" = <mark style="color:green;">**ALLOW**</mark>

AWS evaluates permissions based on the following ->Explicit Deny? No -> Explicit Allow -> Yes, inside bucket policy -> Grants permission if there is no conflicting deny.

Even if the IAM policy does not provide permissions, the explicit allow in the bucket policy overrides the lack of permissions in the IAM policy.

<details>

<summary>Explicit Allow for specific ARNs</summary>

```json
// Must provide exact role that is allowed, i.e., specific role ARNs
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::ACCOUNT_ID:role/Role1",
          "arn:aws:iam::ACCOUNT_ID:role/Role2"
        ]
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mybucketname/*"
    }
  ]
}

```



</details>



3. IAM Policy Allowing Read, Write, and Execute (RWX) attached either to Role or User, including a service role like one assigned to an EC2 instance + S3 Bucket Policy with an explicit "Deny" for RWX = <mark style="color:red;">**DENY**</mark>

* Environment-Based Access Control (e.g., Dev, Test, Production), ie deny access for non-production instances to specific folders within the bucket.
* Use IP condition to  block access from instances outside the designated VPCs, getting network-based security for data, ensuring that only instances from approved locations can access sensitive buckets.

<details>

<summary>Conditional IP </summary>

*   An **IAM policy** attached to a user or role with:

    ```json
    jsonCopy code{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:*",
          "Resource": "arn:aws:s3:::mybucketname/*"
        }
      ]
    }
    ```
*   An **S3 bucket policy** with:

    ```json
    jsonCopy code{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Deny",
          "Principal": "*",
          "Action": "s3:*",
          "Resource": "arn:aws:s3:::mybucketname/*",
          "Condition": {
            "IpAddress": {
              "aws:SourceIp": "198.51.100.0/24"
            }
          }
        }
      ]
    }
    ```

</details>

* Time-Limited Access&#x20;

EC2 instances will be denied permission to upload objects, regardless of other IAM permissions outside the time specified.

<details>

<summary>Conditional Write To Bucket   </summary>

{% code overflow="wrap" %}
```json
//We allow instance to upload to bucket s3:PutObject only between 8am and 
// 6pm.
// Using both allow and deny makes the policy clear, as it explicitly states what is allowed and what is not and minimises the risk of accidental writes during off-hours
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:role/EC2XYZRole"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mybucketname/*",
      "Condition": {
        "DateGreaterThan": {"aws:CurrentTime": "2024-10-25T08:00:00Z"},
        "DateLessThan": {"aws:CurrentTime": "2024-10-25T18:00:00Z"}
      }
    },
    {
      "Effect": "Deny",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:role/EC2XYZRole"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mybucketname/*",
      "Condition": {
        "DateLessThan": {"aws:CurrentTime": "2024-10-25T08:00:00Z"},
        "DateGreaterThan": {"aws:CurrentTime": "2024-10-25T18:00:00Z"}
      }
    }
  ]
}


```
{% endcode %}

</details>

### Dynamic IAM Policy -> leverage ${aws: variable}

* Dynamic policies evaluate conditions in real-time when an API request is made.&#x20;
* AWS variables are resolved when policy is evaluated.
* Dynamic policies can include conditions based on the requester's attributes, the resource's attributes, the time of the request.

<details>

<summary>Common Dynamic Variables summary </summary>

#### 1. **`aws:username`**

* Represents the IAM username of the requester.
* **Example use**: Restrict access to resources specific to the user.

#### 2. **`aws:userid`**

* Represents the unique identifier for the IAM user or role.
* **Example use**: Can be used for fine-grained access control based on the user’s unique ID.

#### 3. **`aws:requester`**

* Represents the account ID or ARN of the requester.
* **Example use**: Restrict actions based on the account making the request.

#### 4. **`aws:PrincipalTag`**

* Represents the tags attached to the IAM principal (user or role).
* **Example use**: Implement access controls based on IAM tags like `Role` or `Environment`.

#### 5. **`aws:currentTime`**

* Represents the current UTC time.
* **Example use**: Restrict access to certain resources during specific times of day or specific dates.

#### 6. **`aws:MultiFactorAuthPresent`**

* Indicates whether MFA (Multi-Factor Authentication) is enabled for the requester.
* **Example use**: Require MFA for sensitive actions (e.g., deleting resources).

#### 7. **`aws:requestTag`**

* Represents tags attached to the request (e.g., EC2 instance tags).
* **Example use**: Grant access to resources based on resource tags, like `Environment` or `Department`.

#### 8. **`aws:SourceIp`**

* Represents the IP address from which the request is made.
* **Example use**: Restrict access based on IP address (e.g., allowing access from specific ranges or locations).

#### 9. **`aws:TokenIssueTime`**

* Represents the time when temporary credentials (session tokens) were issued.
* **Example use**: Restrict access based on the age of temporary credentials.

#### 10. **`aws:PrincipalOrgID`**

* Represents the AWS Organization ID of the principal making the request.
* **Example use**: Grant access to resources only to principals within a specific AWS Organization.

#### 11. **`aws:ExternalId`**

* Used to provide an external ID in cross-account scenarios.
* **Example use**: Verify that a third party assuming a role uses the correct external ID.

#### 12. **`aws:UserAgent`**

* Represents the user-agent string of the requester.
* **Example use**: Restrict access based on the type of client or tool used for the request.

#### 13. **`aws:RequestId`**

* Represents the unique identifier of the request.
* **Example use**: Logging and tracking request-specific data.

#### 14. **`aws:SecureTransport`**

* Indicates whether the request was made over a secure channel (HTTPS).
* **Example use**: Require HTTPS for certain actions to ensure encrypted communication.

#### 15. **`aws:PrincipalServiceName`**

* Represents the name of the AWS service that made the request.
* **Example use**: Restrict actions based on the AWS service making the request.

#### 16. **`aws:RoleSessionName`**

* Represents the session name used when assuming a role.
* **Example use**: Grant or deny access based on role session names.

#### 17. **`aws:Region`**

* Represents the AWS region in which the request is made.
* **Example use**: Restrict access to resources based on region.

#### 18. **`aws:RequestTime`**

* Represents the timestamp of when the request was made.
* **Example use**: Apply conditions based on the time of the request.

#### 19. **`aws:PrincipalType`**

* Represents the type of principal making the request (e.g., `IAMUser`, `AssumedRole`, etc.).
* **Example use**: Implement role-based access control based on principal type.

#### 20. **`aws:VersionId`**

* Represents the version ID of an object (e.g., in S3).
* **Example use**: Restrict actions on specific versions of resources.

#### 21. **`aws:SourceVpce`**

* Represents the VPC endpoint ID from which the request was made.
* **Example use**: Restrict access to VPC endpoints for internal use.

#### 22. **`aws:VpcId`**

* Represents the VPC ID from which the request was made.
* **Example use**: Restrict access to certain resources to specific VPCs.

</details>



For example, IAM policy that allows a **user** to perform actions on their own bucket directory, you can use <mark style="color:red;">policy variables.</mark> **How it works? -> Dynamic Resolution with standard policy variables**

* Attach to all users, instead of impractically create individual policies

**Policy Example**:

1. No hardcoding of usernames. **`Resource: "arn:aws:s3:::example-bucket/${aws:username}/*"`**: This restricts the access to only the objects stored under the specified S3 bucket, and dynamically substitutes `${aws:username}` for the IAM user's username. For example, if the user’s IAM username is `john_doe`, the effective resource ARN will be `arn:aws:s3:::example-bucket/john_doe/*`,
2. The policy <mark style="color:green;">**allows**</mark> you to access objects in the S3 bucket, but **only from a specific IP range**, like `192.168.1.0/24` (which represents your office's network).&#x20;
   1. **`aws:SourceIp: "192.168.1.0/24"`**: This means that only requests from the IP address range `192.168.1.0/24` will be allowed to perform the `s3:GetObject` action. Any request from outside this range will be denied.

```jsonp
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::workstuff-bucket/julia-23w4e/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "192.168.1.0/24" //outside the allowed range no access 
          // even though you're logged in as julia-23w4e
        }
      }
    }
  ]
}

```

**Enable Users to Manage their Credentials ->** The following policy permits any IAM user to perform any of the key and certificate related actions on their own credentials.

```
{
   “Version” : “2012-10-17”,
   “Statement” : [ {
       “Effect” : “Allow”,
       “Action” : [ “iam:*AccessKey*”, “iam:*SigningCertificate*” ],
       “Resource” :
       [ “arn:aws:iam::123456789012:user/${aws:username}” ]
     }
   ]
}
```



## MFA&#x20;

MFA -> password + a security device&#x20;

* Virtual MFA device ( Google Authenticator; Authy)
* Physical key, like YubiKey (U2F)
* Hardware key fob MFA device for AWS GovCloud(US)
* Hardware key fob

## IAM Roles ( for services, apps, user)

> Common roles include EC2 instance, Lambda functions

Services, apps, user can temporarily assume certain permissions without needing to share long-term credentials like access keys with IAM Roles.

The **Lambda Execution Role** is an **IAM Role** that has both **permissions** (via a Permission Policy) and a **Trust Policy** attached. Lambda **Resource-Policy** controls which **external entities** (other AWS services, accounts, or users) have permission to **invoke** or interact with the Lambda function ( ie **S3 or API Gateway to invoke** it,&#x20;

1.  **Permission policy:** permissions i.e. **what actions** (API calls) app/service can perform.

    &#x20;\- You provide IAM Role when you create a function, and Lambda assumes the roles when it needs to read objects from an S3 bucket, the Permission Policy grants `s3:GetObject.`

    &#x20;\-  Lambda writes to DynamoDB.
2.  **Trust Policy:**  specifies who (or what) is allowed to assume the role.

    \- For AWS Lambda, the **service principal** is `"lambda.amazonaws.com"`, which is allowed to take the action `"sts:AssumeRole"`. This enables Lambda to assume the role and execute the function on behalf of the service, while gaining the permissions needed to call the AWS Security Token Service (AWS STS) `AssumeRole` action.

### Common use cases

* **AWS Lambda Execution Roles** ( IAM role to grant the function permissions to access other AWS services, like read/write to s3 bucket and access CloudWatch logs).
* **EC2 Role** (ex: IAM role with permissions to access s3  bucket without embedding access keys in the instance).
* AWS **Service Roles** ( Services assume the Role to perform actions on your behalf, for example, AWSCloud9ServiceRolePolicy, a managed policy attached to a service-linked role (often prefixed with `AWSServiceRoleFor`) which grants Cloud9 the necessary permissions to perform actions in your AWS account.

<details>

<summary><strong>AWSCloud9ServiceRolePolicy - AWS Managed</strong></summary>

```json
// Cloud9 standard aws maanged policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances",
                "ec2:CreateSecurityGroup",
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceStatus",
                "cloudformation:CreateStack",
                "cloudformation:DescribeStacks",
                "cloudformation:DescribeStackEvents",
                "cloudformation:DescribeStackResources"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:TerminateInstances",
                "ec2:DeleteSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:DeleteStack"
            ],
            "Resource": "arn:aws:cloudformation:*:*:stack/aws-cloud9-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:instance/*",
                "arn:aws:ec2:*:*:security-group/*"
            ],
            "Condition": {
                "StringLike": {
                    "aws:RequestTag/Name": "aws-cloud9-*"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "ec2:ResourceTag/aws:cloudformation:stack-name": "aws-cloud9-*"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "arn:aws:license-manager:*:*:license-configuration:*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:ListInstanceProfiles",
                "iam:GetInstanceProfile"
            ],
            "Resource": [
                "arn:aws:iam::*:instance-profile/cloud9/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": [
                "arn:aws:iam::*:role/service-role/AWSCloud9SSMAccessRole"
            ],
            "Condition": {
                "StringLike": {
                    "iam:PassedToService": "ec2.amazonaws.com"
                }
            }
        }
    ]
}
```



</details>



* **Federated User** (For AD or SAML based Identity provider set up federation, allowing users to assume IAM roles in AWS to access resources based on their identity in the external system)
* **Cross-Account** Access
* **Break Glass** Emergency Roles ( on call engineers using emergency procedure to assume the role in case of A,B,C)&#x20;





## Access Keys :key: for CLI, SDK (associated with IAM User)

> Are long-term credentials for an IAM **User** or root user. Give programmatic access to the AWS CLI or AWS API (directly or using the AWS SDK).&#x20;
>
> * Generated in management console.
> * Same keys for CLI and any SDK.
> * Users generate and  manage their own keys.
> * Applications / scripts use the access key pair to sign requests to AWS services.

&#x20;**Access keys have two parts:**

1. Access key ID like username (for example: `AKIAIOSTUTORIALSDOJO`)
2. Secret access key like password (for example: `wJalrXUtnFEMI/K7MDENG/bTutorialsDojoKEY`







#### Useful Links

[https://docs.aws.amazon.com/IAM/latest/UserGuide/reference\_policies\_elements.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html)

[https://docs.aws.amazon.com/IAM/latest/UserGuide/reference\_policies\_variables.html#policy-vars-using-variables](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html#policy-vars-using-variables)
