# AWS IAM: Users, Groups, Roles, and Policies

This guide summarizes the accompanying PDF, **A Guide to AWS IAM for Fault Tolerant, Secure, and Scalable Infrastructure**. It explains how to manage identities, delegate temporary access, and enforce permissions across your AWS environment.

## The core building blocks

Treat these four components as separate layers that combine to answer one fundamental question—*who can do what, and to which resources*:

1. **IAM Users** represent distinct individual identities (such as developers or administrators) with long-term credentials for day-to-day console or programmatic tasks.
2. **IAM Groups** organize multiple IAM Users into logical collections so permissions can be assigned and scaled by job function rather than per individual.
3. **IAM Roles** provide temporary, credential-free access via AWS Security Token Service (STS) for services, cross-account workflows, and external users.
4. **IAM Policies** act as the JSON rulebooks defining explicit `Allow` or `Deny` effects, actions, resources, and conditions.

Users, Groups, and Roles have zero permissions by default. They remain empty vessels until an IAM Policy is explicitly attached to define what they are permitted to do.

## IAM Users: managing individual identities

An IAM User represents a single person or identity that needs day-to-day operational access to your AWS environment.

| Identity type | Authentication method | Default permissions | Key point |
| --- | --- | --- | --- |
| **Root User** | Root email, password, and mandatory MFA | Unrestricted access to all AWS services | Lock away immediately and never use for daily operational tasks. |
| **IAM User** | Username/password (Console) or Access Keys (CLI/SDK); MFA recommended | Zero permissions (explicit deny by default) | Must be explicitly granted permissions via policies; ideal for individual team members like `dev-alex` who need scoped EC2 and S3 access without risking production databases. |

Never share a single IAM User across multiple people; create a distinct IAM User for each individual so actions remain auditable.

**Architecture diagram:** [Open the diagram](./diagrams/IAM_User_Dark.png)


## IAM Groups: scaling user permissions

An IAM Group is a collection of IAM Users that lets you manage permissions collectively by team or job function rather than updating profiles one by one.

| Group example from the PDF | Target team | Typical permissions | Key point |
| --- | --- | --- | --- |
| **Frontend-Devs** | Frontend developers | Access to specific S3 buckets hosting static frontend assets | Adding a new intern's IAM User automatically grants team permissions; removing them revokes access cleanly. |
| **Backend-Devs** | Backend developers | Access to DynamoDB, RDS, Aurora, and ElastiCache | Keeps database and backend service access isolated from frontend workflows. |
| **Cloud-Architects** | Cloud engineers | Broad access to EC2, VPCs, and Auto Scaling Groups | Centralizes core infrastructure management privileges. |
| **Finance-Team** | Accounting and finance | Access strictly to AWS Billing and Cost Explorer | Enforces separation of duties by preventing billing staff from modifying infrastructure. |

Policies attached to an IAM Group are automatically inherited by every IAM User placed inside that group.

**Architecture diagram:** [Open the diagram](./diagrams/IAM_Group_Dark.png)

## IAM Roles: temporary access patterns

Unlike IAM Users, Roles do not rely on passwords or permanent access keys. Instead, an entity temporarily *assumes* the Role to receive short-lived security credentials.

| Role type | Who or what assumes it | Typical fit | Real-world example from the PDF |
| --- | --- | --- | --- |
| **Service Role** | AWS resources and services | Allowing one AWS service to interact securely with another | An EC2 instance running a frontend app assumes a role to upload user profile pictures directly to an S3 bucket without hardcoded keys. |
| **Cross-Account Role** | Identities from another trusted AWS account | Sharing resources safely across separate AWS accounts | A developer in the `Development` account assumes a role in the `Production` account to inspect CloudWatch logs without needing a separate Prod user. |
| **Identity Provider (Federated) Role** | External users authenticated outside of AWS | Granting cloud access to end-users via third-party logins | A mobile game player signs in with Google; AWS trusts Google's authentication and issues a temporary role to save high scores to a cloud database. |

Never hardcode AWS usernames, passwords, or long-term access keys inside application code or server configurations—always use IAM Roles for service-to-service and external access.

**Architecture diagram:** [Open the diagram](./diagrams/IAM_Role_Dark.png)

## IAM Policies: defining permissions

Policies are structured JSON documents evaluated whenever a request is made in AWS.

| Policy category | Who creates and maintains it | Best suited to | Key point |
| --- | --- | --- | --- |
| **AWS Managed Policy** | AWS | Fast setup for common job functions (e.g., `AmazonDynamoDBFullAccess`) | Convenient for getting started quickly, though it often grants broader permissions than strictly necessary. |
| **Customer Managed Policy** | You (the AWS customer) | Enforcing the Principle of Least Privilege on specific workloads | Allows granular rules, such as restricting a developer to `s3:GetItem` on a single bucket (`FinancialData_Prod`). |
| **Inline Policy** | You (embedded directly into one identity) | Strict 1-to-1 relationships where a rule must never be shared | Rarely recommended today; deleted automatically if the parent User, Group, or Role is deleted. |

### Identity-based vs. Resource-based policies

* **Identity-based policies** are attached to an IAM User, Group, or Role and dictate what that specific identity can do.
* **Resource-based policies** are attached directly to an AWS resource (such as an S3 Bucket Policy) and dictate who is allowed to access that resource. For example, granting public read access (`s3:GetObject`) to website assets in an S3 bucket requires both a resource-based bucket policy and disabling **Block All Public Access** under the bucket's Permissions tab.

**Architecture diagram:** [Open the diagram](./diagrams/IAM_Policy_Dark.png)

## How the pieces fit together

In a resilient, production-grade cloud architecture, these IAM layers work together to minimize risk and administrative overhead:

* **Secure the foundation:** Lock away the Root account with MFA enabled, and create individual IAM Users for team members.
* **Scale with Groups:** Place those IAM Users into job-specific IAM Groups (`Frontend-Devs`, `Backend-Devs`, `Cloud-Architects`, `Finance-Team`) rather than attaching policies one by one.
* **Eliminate static secrets:** Assign Service Roles to compute resources like EC2, Cross-Account Roles for multi-account workflows, and Federated Roles for external user authentication.
* **Contain the blast radius:** Replace broad AWS Managed Policies with tightly scoped Customer Managed Policies so that even if credentials or components are compromised, unauthorized access remains strictly limited.