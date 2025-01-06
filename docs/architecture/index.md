# Overview

AWS provides many different services to its customers. It's essential to structure those services to ensure operational and budgetary goals are met. Many services and resources can be shared across the entire environment. For example, AWS Security Hub provides a single pane of glass look into the security posture of our AWS environment. It would not make sense to create many different Security Hubs.

## Accounts

An AWS account represents a formal business relationship between CDCR and AWS. Our cloud resources and data are contained in an AWS account.

* Resources container – An AWS account is the basic container for all the AWS resources that CDCR creates. Every resource is uniquely identified by an Amazon Resource Name (ARN) that includes the account ID of the account that contains, or owns, the resource.
* Security boundary – An AWS account is also the basic security boundary for AWS resources. Resources that we create in our accounts are available to users who have credentials for our accounts.

With an organization as large as CDCR, there are many different solutions, teams, and needs. Rather than create all resources under a single AWS account, we divide up the resources into multiple AWS accounts.

## Organizations

AWS Organizations helps us centrally manage and govern the CDCR AWS environment. Using Organizations, we can create accounts and allocate resources, group accounts into an organization structure, apply policies for governance, and simplify billing by using a single payment account for all the accounts. The single payment account is sometimes referred to as the master payer or payer account.

When a new AWS account is provisioned, it is tied to our AWS organization. You can read more about [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html) from their documentation.

## Control Tower

Many large enterprises use AWS for their infrastructure needs. Over time, AWS found patterns and practices that generally work best for most organizations. To help customers quickly apply governance, security, and other best practices, AWS created a service called [Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html) to help customers set up their AWS environment. CDCR implemented Control Tower from the beginning.

## Organizational Structure

Our AWS accounts are organized in a structure to facilitate operational needs, policies, and security requirements. This structure is not set in stone and may change to fit our needs. Accounts can be moved into a different OU if needed.

### Security Organizational Unit
The security OU contains accounts created by Control Tower. It is not the OU where we would manage security services. The accounts in the Security OU handle auditing and permissions for accounts managed by Control Tower.

### CDCR Organizational Unit
The CDCR OU is where most resources are created. Our workloads or applications are buit under the CDCR OU. It also includes shared resources used by many accounts like Route 53 for DNS.

#### Platform
The Platform OU is for operational accounts that facilitate the use of AWS. There are many common basic needs like connectivity, DNS, and identity that are required to build applications. The Platform OU contains the foundational building blocks needed to run AWS services.

#### Application
The Application OU contains accounts where we build solutions to solve business needs. It is further divided into Internal and External applications. An internal application is only accessible from the CDCR network. An external application is accessible from the internet. This allows security policies to be applied differently based on which OU the account is in. Further, there are Production and Non-Production OUs to also apply different policies.

### Sandbox
This OU is meant for testing out new services. It is the least restrictive OU, but should not contain any production/real data.

## Diagram of Organizational Structure
``` mermaid
---
title: AWS Organizational Structure for Accounts
---
stateDiagram-v2
  state "Root OU" as Root

  Root --> masterpayer
  Root --> Security
  Root --> CDCR
  Root --> Sandbox
  

  state "masterpayer-1339240" as masterpayer
  state "Security" as Security
  state "CDCR" as CDCR
  state "Sandbox" as Sandbox

  state "Control Tower" as masterpayer
  state "Organizations" as masterpayer
  state "SSO" as masterpayer
  state "AWS Config" as masterpayer

  Security --> cdcr_security_audit
  Security --> cdcr_security_logarchive
  
  state "cdcr-security-audit" as cdcr_security_audit
  state "Used to assume roles" as cdcr_security_audit
  state "No resources are built here" as cdcr_security_audit

  state "cdcr-security-logarchive" as cdcr_security_logarchive
  state "CloudTrail" as cdcr_security_logarchive

  CDCR --> cdcr_platform
  CDCR --> cdcr_applications

  state "Platform" as cdcr_platform
  state "Applications" as cdcr_applications

  cdcr_platform --> cdcr_platform_operations
  cdcr_platform --> cdcr_platform_security
  cdcr_platform --> cdcr_platform_orchestration
  cdcr_platform --> cdcr_platform_connectivity
  
  state "cdcr-platform-operations" as cdcr_platform_operations
  state "Route 53" as cdcr_platform_operations
  state "Shared VPC Endpoints" as cdcr_platform_operations
  state "Systems Manager" as cdcr_platform_operations
  state "Backup" as cdcr_platform_operations
  state "CloudFormation Stacksets" as cdcr_platform_operations
  state "Private CA" as cdcr_platform_operations
  state "KMS" as cdcr_platform_operations
  state "IPAM" as cdcr_platform_operations

  state "cdcr-platform-security" as cdcr_platform_security
  state "Security Hub" as cdcr_platform_security
  state "Security Lake" as cdcr_platform_security
  state "Macie" as cdcr_platform_security
  state "Inspector" as cdcr_platform_security
  state "GuardDuty" as cdcr_platform_security
  state "Detective" as cdcr_platform_security
  state "CloudTrail (Replica)" as cdcr_platform_security
  state "Splunk Log Forwarder" as cdcr_platform_security

  state "cdcr-platform-orchestration" as cdcr_platform_orchestration
  state "Terraform State" as cdcr_platform_orchestration
  state "Terraform DynamoDB" as cdcr_platform_orchestration
  state "Code Commit" as cdcr_platform_orchestration
  state "Code Build" as cdcr_platform_orchestration
  state "Code Pipeline" as cdcr_platform_orchestration
  state "Service Catalog" as cdcr_platform_orchestration

  state "cdcr-platform-connectivity" as cdcr_platform_connectivity
  state "Transit Gateway" as cdcr_platform_connectivity
  state "Direct Connect" as cdcr_platform_connectivity
  state "Palo Alto Cloud NGFW" as cdcr_platform_connectivity
  state "Palo Alto Panorama" as cdcr_platform_connectivity


  cdcr_applications --> cdcr_applications_internal
  cdcr_applications --> cdcr_applications_external

  state "Internal" as cdcr_applications_internal
  state "External" as cdcr_applications_external

  cdcr_applications_internal --> cdcr_applications_internal_prod
  cdcr_applications_internal --> cdcr_applications_internal_nonprod

  state "Production" as cdcr_applications_internal_prod
  state "Non-Production" as cdcr_applications_internal_nonprod

  cdcr_applications_internal_prod --> cdcr_app_example_prod

  state "cdcr-app-example-prod" as cdcr_app_example_prod
  state "Application Load Balancer" as cdcr_app_example_prod
  state "EC2" as cdcr_app_example_prod
  state "Security Group" as cdcr_app_example_prod
  state "RDS" as cdcr_app_example_prod
  state "VPC" as cdcr_app_example_prod
  state "TGW Attachment" as cdcr_app_example_prod

  cdcr_applications_internal_prod --> cdcr_app_widget_prod

  state "cdcr-app-widget-prod" as cdcr_app_widget_prod
  state "Application Load Balancer" as cdcr_app_widget_prod
  state "Elastic Container Registery" as cdcr_app_widget_prod
  state "Elastic Container Service" as cdcr_app_widget_prod
  state "Security Group" as cdcr_app_widget_prod
  state "Aurora DSQL" as cdcr_app_widget_prod

  cdcr_applications_external --> cdcr_applications_external_prod
  cdcr_applications_external --> cdcr_applications_external_nonprod

  state "Production" as cdcr_applications_external_prod
  state "Non-Production" as cdcr_applications_external_nonprod

  Sandbox --> cdcr_sbx_infrastructure

  state "cdcr-sbx-infrastructure" as cdcr_sbx_infrastructure
  state "EC2" as cdcr_sbx_infrastructure
```