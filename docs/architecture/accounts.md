# Accounts
There are many different accounts within our AWS Organization. Some accounts contain applications or other workloads used to support CDCR's business functions. Other accounts deal with the operational requirements of AWS itself.

## Creation
CDCR accounts are created via Terraform in the masterpayer-accounts GitHub repository. Naming should follow our standards.

For application accounts:

    cdcr-app-name-env

For platform accounts:

    cdcr-platform-name

The root email address must always use plus addressing to the AWS admin distribution list.

    cdcrawsacct+accountname@cdcr.ca.gov

## Master Payer
The master payer account was the first account created. It was created via a request to CDT. Most state departments are not allowed to have a master payer account. They create accounts underneath CDT's master payer account via a ServiceNow request. Due to CDCR's size and complexity, we were provided an exemption to the standard.

Since Rackspace won the contract for providing AWS services to the state, our AWS master payer account is in their name. Further, Rackspace applies additional policies to block access to billing information in the console. See the [billing.md](http://127.0.0.1:8000/architecture/billing/#rackspace) page for more information.

### Services
Since the master payer account is the root of our organization, several critical services are provided here.

- Control Tower - Provides AWS best practices for managing an organization with many accounts.
- Organizations - Provides a method to organize AWS accounts.
- IAM Identity Center - Provides the authentication portal that is integrated with CDCR's Entra ID tenant for single sign-on.

## Control Tower
Control Tower creates two additional accounts to perform its functions. These accounts may be referred to as Control Tower shared services or core accounts.

### Audit
The CDCR audit account is named cdcr-security-audit. This account is setup automatically by Control Tower. This account should not be modified by CDCR staff unless required to manage Control Tower. 

The audit account receives notifications though the AWS Simple Notification Service (SNS). The types of events include:

* All Configuration Events – This topic aggregates all CloudTrail and AWS Config notifications from all accounts in your landing zone.
* Aggregate Security Notifications – This topic aggregates all security notifications from specific CloudWatch events, AWS Config Rules compliance status change events, and GuardDuty findings.
* Drift Notifications – This topic aggregates all the drift warnings discovered across all accounts, users, OUs, and SCPs in your landing zone.

### Log Archive
The CDCR log archive account is named cdcr-security-logarchive. This account is setup automatically by Control Tower.

The Log Archive account contains contains a central Amazon S3 bucket for storing a copy of all AWS CloudTrail and AWS Config log files for all other accounts managed by Control Tower.The events stored in the S3 Bucket are replicated to a different S3 bucket which Splunk consumes. This ensures no processes directly interact with logging data.

## Connectivity
The connectivity account is named cdcr-platform-connectivity. This critical account provides network connectivity across the entire AWS infrastructure. This includes connectivity to on-premises and between AWS accounts. It also includes other centralized networking resources like a firewall.

### Services
* Direct Connect - Provides a dedicated circuit from our MPLS network into our AWS infrastructure. Our Direct Connect partner is Verizon.
* Transit Gateways - Transit gateways provide a method to connect many VPC networks together without the complexity of VPC peering. The transit gateway is shared to all accounts in the organization so other VPCs can connect to it.
* Firewall - We use Palo Alto's Cloud Next Generation Firewall (CNGFW) service. Although this is a managed service, traffic must be routed to Palo Alto's service to be inspected.
    - Since Network Engineering was new to Palo Alto at the time, the connectivity account also hosts a Palo Alto Panorama device for managing the CNGFW. The CNGFW could also be managed through Terraform, but the network engineering team does not have the bandwidth to learn a new language.

## Operations
The operations account is named cdcr-platform-operations. This account contains AWS services that can be shared across many accounts. For example, it's important to have a centralized DNS so that namespaces don't collide. These types of services are built in the operations account then shared to other accounts where appropriate.

### Services
* Route 53 - The operations account provides central DNS services within AWS. This includes inbound (on-prem to AWS) and outbound (AWS to on-prem) DNS requests. Non ca.gov domains are also hosted here, like successca.net.
* VPC Endpoints - VPC endpoints provide private access to public AWS services rather than routing through the public internet. For example, sending logs to CloudWatch would normally require routing to a public IP address. Instead, traffic is routed to a private endpoint that's on a private IP address. Each endpoint costs money, so rather than creating them in each account, it makes sense to centralize the endpoints where applicable.
* Systems Manager - Certain services like Patching can be centrally managed through the operations account. The patch baselines are stored in the operations account and shared to other accounts. Ohter features like the Parameter Store can be used to share parameters across all accounts.
* CloudFormation StackSets - This account is delegated administrator for StackSets in the AWS Organization. StackSets are used enforce the creation of resources or ensure compliance across many accounts. StackSets can be applied to OUs or individual accounts.
* IP Address Management - It's important to ensure IP addresses are properly managed and do not overlap with other VPCs. The AWS IPAM service provides the capability to hand out IP address space as well as view an overlapping subnets.

## Orchestration
The orchestration account is named cdcr-platform-orchestration. This account is used to orchestrate platform activities within AWS. Activities like building resources are performed through this account. If CDCR adopts a service like Terraform Cloud or SpaceLift, then this account may become obsolete.

### Services
* Terraform State - Our Terraform state files are stored in an S3 bucket in the orchestration account.
* DynamoDB - To prevent issues with multiple staff editing a Terraform project at the same time, we use a locking mechanism. The lock is stored in DynamoDB and whoever is first to edit the state wins control. Other users are blocked from editing the state until the lock is released.
* CodeCommit - An AWS version control system (VCS) similar to GitHub. AWS is no longer accepting new customers on this service so we will be migrating to GitHub to store our Terraform source code.
* CodePipeline -A continuous delivery service that automates the steps required to build resources from the source code. You design a workflow that performs different stages needed to build your source code.
* CodeBuild - This is where the actual compute activities occur to build the source code. For example, CodePipeline will call a CodeBuild project and pass in the source code. CodeBuild spins up a temporary instance to run any build commands needed.

## Security
The security account is named cdcr-platform-security. This account provides visibility to our security posture in AWS. Its purpose is to surface the various configuration and incident alerts to a single account. This allows security staff to quickly assess issues within the AWS environment rather than going to each account.

### Services
* Splunk logs - Various log sources like VPC flow logs or CloudTrail are sent to S3 buckets in the security account. Then Splunk is able to retrieve the logs from these buckets in a centralized way.
* Security Hub - Provides a comprehensive view of the security state of our AWS infrastructure. Also provides recommendations on security best practices across all accounts.