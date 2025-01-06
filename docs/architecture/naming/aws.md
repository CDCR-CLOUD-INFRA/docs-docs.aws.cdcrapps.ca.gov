# AWS Resources
This section contains the naming and tagging standards for resources built in AWS.

## Style
Unless noted, all resources shall use lower case and dashes as a separator. 

If the name portion of the resource needs a separator, use an underscore.

    vpc-uw2-platform-{==sharedservices_endpoints==}-prod-01

## Region Codes
When naming resources, it helps to include the region where the resource lives. Some resources are not region specific. In this case, the region is considered global. In AWS, global resources are typically accessed through the US East 1 region.

| Region | Abbreviation |
| ---- | ---- |
| global | gbl |
| us-west-1 | uw1 |
| us-west-2 | uw2 |
| us-east-1 | ue1 |
| us-east-2 | ue2 |
| us-gov-west-1 | gw1 |
| us-gov-east-1 | ge1 |

## Terms and Abbreviations
| Name | Value | Description |
| ---- | ----- | ----- |
| app | Application | Resources built for a specific application/workload/solution
| pltfrm | Platform | Resources built as part of the basic components to run services in AWS
| wrktype | Workload Type | The type of workload the resource is used for, either app or platform
| az | Availability Zone | The [availability zone](https://docs.aws.amazon.com/whitepapers/latest/get-started-documentdb/aws-regions-and-availability-zones.html) where a resource is created
| prod | Production | Resources intended for customer access
| dev | Development | Resources intended for IT staff to build and work on
| tst | Test | Resources intended to review before moving to production
| int | Internal| Resources restricted to CDCR network access only
| ext | External | Resources that are accessible from the public internet
| mgmt | Management | Resources intended for management or administration

## Resource Types
### Networking
| Resource | Convention | Example | Notes |
| ----------- | ------ | ---- | ---- |
| [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) | vpc-REGION-WRKTYPE-NAME-ENV-NUMBER | vpc-uw2-app-pathfinder-prod-01 | |
| [Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)| snet-REGION-WRKTYPE-NAME-ENV-AZ-NUMBER | snet-uw2-app-pathfinder-prod-a-01 | |
| [Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html) | rtb-REGION-WRKTYPE-NAME-ENV | rtb-uw2-app-pathfinder-prod | Additional identifiers may be appended if needed to identify the associated subnet | |
| [Security Group](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) | secgrp-REGION-WRKTYPE-NAME-PURPOSE-ENV | secgrp-uw2-app-pathfinder-storage-prod | |
| [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html) | igw-REGION-WRKTYPE-NAME-ENV-NUMBER | igw-uw2-pltfrm-inspection-prod-01 | |
| [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) | natgw-REGION-WRKTYPE-NAME-ENV-NUMBER | natgw-uw2-app-pathfinder-prod-01 | |
| [Endpoints](https://docs.aws.amazon.com/whitepapers/latest/aws-privatelink/what-are-vpc-endpoints.html) | vpce-REGION-WRKTYPE-APPNAME/SHARED-SERVICENAME | vpce-uw2-app-pathfinder-s3.us-west-2-amazonaws.com or vpce-uw2-pltfrm-shared-monitoring.us-west-2.amazonaws.com | |
| [Prefix List](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-managed-prefix-lists.html) | pl-REGION-NAME | pl-uw2-tanium | |
| [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) | tgw-REGION | tgw-uw2
| [Transit Gateway Attachment](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-vpc-attachments.html) | tgwa-REGION-WRKTYPE-NAME-ENV-NUMBER | tgwa-uw2-app-pathfinder-prod-01 | For platform, the environment is assumed to be production.
| [Transit Gateway Route Table](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-route-tables.html) | tgwrtb-REGION-WRKTYPE-NAME-PURPOSE | tgwrtb-uw2-platform-connectivity-inspected | |
| [Direct Connect Gateway](https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-gateways-intro.html) | dxgw-REGION-WRKTYPE-NAME | dxgw-gbl-pltfrm-connectivity | Direct Connect Gateways are always global and should be platform.
| [Direct Connect Connection](https://docs.aws.amazon.com/directconnect/latest/UserGuide/dedicated_connection.html) | dxc-REGION-WRKTYPE-CITY-CIRCUITID | dxc-uw1-platform-sanjose-c1234567 | The city is where the circuit cross connect is located. The circuit ID is provided by the vendor (e.g. Verizon) |
| [Direct Connect Virtual Interface](https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html) | dxvif-REGION-WRKTYPE-NAME-CITY-CIRCUITID | dxvif-uw1-platform-connectivity-sanjose-c1234567 | The city is where the circuit cross connect is located. The circuit ID is provided by the vendor (e.g. Verizon) |


### Storage
Here's where naming standards for storage will go.

### Compute
Here's where naming standards for compute will go.

## Tags
| Name | Values | Purpose |
| ---- | ------ | ------- | 
| CostCenter | FA000000_CC00000 | Combination of the Functional Area and Cost Center. Used by EIS Budget team to identify costs
| Application | APPNAME / PLATFORM | Used by the [AWS myApplications](https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/aws-myApplications.html#myApp-pricing) service to identify costs and security issues related to a particular app.
| Environment | Production / NonProduction | Whether the resource is a production or non-production workload
| Terraform | true / false | Whether the resource is managed through Terraform.
| GitRepository | URL | The URL to the Git repository where the Terraform source code resides