# 06 - IAM and Access Management

> **Status: conceptual.** No IAM console page (users, roles, policies, MFA) was captured. This document explains the concepts relevant to the project and the minimum recommended setup. It does not claim any IAM configuration was built.

## What it is

AWS Identity and Access Management (IAM) controls who (principals) can do what (actions) on which resources, under which conditions. Principals are users, roles and federated identities. Permissions are expressed in JSON policies.

## Why it is needed

Every action in the console, CLI or by an instance is an API call that IAM authorises. Without least-privilege IAM, network controls alone cannot contain a compromised credential.

## How it works

- Policies are evaluated with an explicit deny overriding any allow, and an implicit deny by default.
- Roles issue temporary credentials. EC2 instances receive them through an instance profile, so no access keys are stored on the server.
- The account root user should be protected with MFA and not used for daily work.

## Evidence relevant to IAM in this project

| Observation | Source | Meaning |
|---|---|---|
| "This user is not authorized to call AWS Compute Optimizer" | EBS volume page ([08](08-ebs-and-storage.md)) | The console identity lacks that permission. This shows permissions are not unlimited, but it does not show how the identity was configured. |
| "Failed to load rule groups" for Route 53 Resolver DNS Firewall | VPC page ([02](02-vpc-and-networking.md)) | Cause unconfirmed; possibly a missing permission. |
| IAM role field of the instance | Cut off in the EC2 screenshot | Unknown whether an instance profile is attached. |

## Recommended baseline (not implemented here)

1. Enable MFA on the root user and do not create root access keys.
2. Use IAM Identity Center or individual IAM users with MFA, granted through groups.
3. Attach an instance profile to the server rather than storing keys. For example, the AWS-managed policies `CloudWatchAgentServerPolicy` (publish memory and disk metrics and logs) and `AmazonSSMManagedInstanceCore` (Session Manager access without opening SSH).
4. Use permission boundaries or service control policies where an organisation is available.
5. Review access with IAM Access Analyzer and the credential report.

Illustrative read-only policy for an auditor (not applied in this project):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:Describe*", "cloudwatch:GetMetricData", "cloudwatch:ListMetrics"],
      "Resource": "*"
    }
  ]
}
```

## Security implications

Long-lived access keys are the most common cause of cloud credential leaks. Never commit them; the repository `.gitignore` excludes common key and credential files.

## Cost implications

IAM itself is free. Related services such as CloudTrail data events and some Access Analyzer features can add cost; check current pricing.

## Real-world use cases

Separating administrators from developers and auditors, granting CI pipelines scoped deploy rights, and giving instances access to S3 or Parameter Store without secrets on disk.

## Implemented versus studied

Implemented and evidenced: none. Studied: all of the above.
