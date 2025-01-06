# Routing Tables
In AWS, routing tables provide a method of directing traffic between networks. The routing tables are software defined and controlled through Terraform.

## Types

There are two types of routing tables:

`Transit Gateway Routing Table`

:   Transit gateway routing tables are associated to a transit gateway. When traffic arrives on the transit gateway, the next hop is determined by the routing table. Typically, the next hop would be another VPC or a connection, like VPN or Direct Connect Gateway. See the [AWS documentation](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-route-tables.html) for more information.

`Subnet Routing Table`

:   Subnet routing tables are associated to subnets within a VPC. As traffic is generated within the subnet, the subnet routing table defines where the next hop is. If the destination is within the same VPC, the subnet has a local route to ensure traffic doesn't leave the subnet. For most VPCs within CDCR, the subnet table will be simple with only the transit gateway as the next hop. Certain core network functionality requires more intricate routing tables. See the [AWS documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html) for more information.

## Route Tables
### Definitions
This section details the core routing tables which control how traffic flows between AWS, on-premises, and partner accounts. The tables include two columns:

`Destination`
:   The packet destination specified in CIDR format.

`Target`
:   The next hop where traffic will be directed to.

### On-Premises Networks
Some routing tables receive routes from our on-premises networks using BGP. Although the networks are automatically advertised, it's helpful to know exactly which networks fall within this range. The on-premises routes use large ranges called summary routes. You may notice these routes overlap with other smaller ranges. With route tables, the more specific route wins.

- 153.48.0.0/16
- 134.186.130.0/23
- 67.157.34.0/24
- 67.157.35.0/24

### Transit Gateway
#### Trust

The Trust routing table is linked to networks that CDCR considers trusted. A network is considered trusted when we own and manage the security within that network.

Examples:

- On-premises network
- AWS VPC owned and managed by CDCR
- Other cloud service proivider (Azure, GCP, etc.) network owned and managed by CDCR

| Destination | Target |
| ----------- | ------ |
| `Trusted AWS CIDRs`| Propagated (Transit Gateway Attachments)  |
| `On-prem CIDRs`| Propagated (Direct Connect Gateway) |
| `0.0.0.0/0` | vpc-platform-uw2-inspection |

#### Untrust

The Untrust routing table is linked to networks that CDCR does not control. These networks are considered untrusted because we do not define the security controls within the networks. Being an untrusted network doesn't necessarily mean the traffic is malicious. All traffic destined to or received from an untrusted network is inspected by the Palo Alto Cloud NGFW for AWS.

Examples:

- Internet
- Generis AWS (SOMS ERMS application)
- Other partner networks that may integrated, like the California Department of Justice

| Destination | Target                        |
| ----------- | -------------------- |
| `0.0.0.0/0`       | vpc-platform-uw2-inspection   |

#### Inspected

The Inspected routing table must only ever be linked to the VPC that hosts the Cloud NGFW for AWS. This routing table has access to all networks to ensure the firewall can reach any destination. The default route is sent to blackhole because this table should have a route for every network either statically defined or through propagation.

| Destination | Target                        |
| ----------- | -------------------- |
| `Trusted AWS CIDRs`       | Propagated (Transit Gateway Attachments)   |
| `Untrusted AWS CIDRs`       | Propagated (Transit Gateway Attachments)  |
| `On-prem CIDRs`| Propagated (Direct Connect Gateway) |
| `0.0.0.0/0` | Blackhole |

### Subnet
#### Inspection

The Inspection VPC is where the Cloud NGFW for AWS lives. This service acts as a centralized firewall across AWS. Traffic is directed to the firewall through the routing tables. The VPC is split into 3 categories: Transit Gateway, Inspection, and Egress. Each category has a subnet in each Availability Zone (4 AZs in US West 2).

##### Transit Gateway

This subnet is connected to the transit gateway. It is used to route traffic to the firewall. Separating the firewalls allows more complex routing in the other tables. Since Cloud NGFW for AWS is a managed service, traffic is forwarded from a gateway load balancer into the Palo Alto managed AWS account.

Availability Zone - A

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `0.0.0.0/0` | gwlb-inspection-fw-a |

Availability Zone - B

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `0.0.0.0/0` | gwlb-inspection-fw-b |

Availability Zone - C

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `0.0.0.0/0` | gwlb-inspection-fw-c |

Availability Zone - D

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `0.0.0.0/0` | gwlb-inspection-fw-d |

##### Firewall

The firewall subnet is the connection point to the Palo Alto Cloud NGFW for AWS. Because traffic from either the transit gateway or the internet is directed straight to the gateway load balancer, the only traffic that hits the routing table has already been inspected by the firewall. Traffic is either directed to the transit gateway if it's internal traffic or to the egress subnet if it's internet traffic.

Availability Zone - A

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | nat-inspection-egress-a |

Availability Zone - B

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | nat-inspection-egress-B |

Availability Zone - C

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | nat-inspection-egress-c |

Availability Zone - D

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | nat-inspection-egress-d |

##### Egress

The egress subnet is where outbound internet traffic traverses the network. For example, when a server accesses a public URL like https://www.microsoft.com, the traffic is sent out the egress subnet through a NAT Gateway and the return traffic is sent back through the firewall for processing.

Availability Zone - A

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | gwlb-inspection-fw-a |

Availability Zone - B

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | gwlb-inspection-fw-b |

Availability Zone - C

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | gwlb-inspection-fw-c |

Availability Zone - D

| Destination | Target                        |
| ----------- | -------------------- |
| `10.8.0.0/24`       | Local   |
| `On-Prem CIDRs`       | Transit Gateway (tgw-uw2)   |
| `0.0.0.0/0` | gwlb-inspection-fw-d |

## Examples

Routing may seem complex until you fully understand the tables. These examples are meant to assist in understanding how traffic moves between each routing table.

### Traffic Inspection

``` mermaid
graph LR
  A[Start] --> B{Source Trusted?};
  B -->|Yes| C{Destination Trusted?};
  C -->|Yes| D[Bypass inspection];
  B -->|No| E[Inspect with firewall];
  C -->|No| E;
  D --> F[Send to destination];
  E --> F;
```

### On-Premises to AWS Trusted Application

Trusted Network to Trusted Network

``` mermaid
graph LR
  A[On-Premises] --> B[Direct Connect Gateway];
  B --> C[Transit Gateway];
  C --> D[AWS Trusted VPC];
```

### AWS Trusted Application to Internet

Trusted Network to Untrusted Network

``` mermaid
graph LR
  A[AWS Trusted VPC] --> B[Transit Gateway];
  B --> C[Inspection TGW Subnet];
  C --> D[Inspection FW Subnet];
  D --> E[Inspection Egress Subnet];
  E --> F[Internet];
```

### Generis to On-Premises

Untrusted Network to Trusted Network

Generis hosts our SOMS ERMS application. It is considered an untrusted network because we have no control over their AWS account.

``` mermaid
graph LR
  A[Generis] --> B[Transit Gateway];
  B --> C[Inspection TGW Subnet];
  C --> D[Inspection FW Subnet];
  D --> E[Transit Gateway];
  E --> F[On-Premises];
```