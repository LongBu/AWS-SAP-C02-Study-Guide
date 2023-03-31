# AWS SAP-CO2 Study Guide
The following guide is my attempt at helping myself and possibly others to pass the AWS Certified Solutions Architect Professional Exam.  I'd recommend taking Stephane Maarek's course [Ultimate AWS Certified Solutions Architect Professional 2023](https://www.udemy.com/course/aws-solutions-architect-professional/) and taking a few practice exams from [AWS Certified Solutions Architect Professional Practice Exam](https://www.udemy.com/course/aws-certified-solutions-architect-professional-aws-practice-exams/) before moving onto the exam.  

Note: The author makes no promises or guarantees on this guide as this is as stated, a guide used by myself, nothing more.  This is a work in progress and I haven't passed the test, yet.  

## Table of Contents
1. <a href="#introduction">Introduction</a>
2. <a href="#organizational-unit-ou">Organizational Unit (OU)</a>
3. <a href="#identity-and-access-management-iam">Identity and Access Management (IAM)</a>
4. <a href="#networking">Networking</a>
5. <a href="#ec2">EC2</a>
6. <a href="#containers">Containers</a>
7. <a href="#logging-and-events">Logging and Events</a>
8. <a href="#configurations-and-security">Configurations and Security</a>
9. <a href="#vpc">VPC</a>
10. <a href="#storage">Storage</a>
11. <a href="#database">Database</a>
12. <a href="#analytics">Analytics</a>
13. <a href="#infrastructure-as-code-iac">Infrastructure as Code (IAC)</a>
14. <a href="#optimization">Optimization</a>
15. <a href="#miscellaneous">Miscellaneous</a>
16. <a href="#acronyms">Acronyms</a>

## Introduction
<a href="https://d1.awsstatic.com/training-and-certification/docs-sa-pro/AWS-Certified-Solutions-Architect-Professional_Exam-Guide.pdf">AWS Certified Solutions Architect - Professional (SAP-C02) Exam Guide</a>
### Exam Content Breakdown:

| Domain  | % of Exam |
| ------------- | ------------- |
| Domain 1: Design Solutions for Organizational Complexity  | 26%  |
| Domain 2: Design for New Solutions  | 29%  |
| Domain 3: Continuous Improvement for Existing Solutions | 25% |
| Domain 4: Accelerate Workload Migration and Modernization | 20% |
| **Total** | **100%** |

## Organizational Unit (OU)

### AWS Account Organizational Unit Migration:
  * Remove the member account from the former organization [need root or IAM access to said member account and master account(s)]
  * Send invite to the member account from the prospective organization
  * Accept the invite from the prospective organization upon the member account
  * Ensure the OrganizationAccountAccessRole is added to the member account

### AWS Control Tower:
  * Easy way to setup and govern a secure and compliant multi-account AWS environment based on best practices and using AWS OU to create accounts
  * automates setup of environments in a few clicks
  * automates ongoing policy managment using guard rails:
    * SCP: preventative
    * AWS Config: detective
  * detects policy violations and remediates them
  * monitor compliance through dashboards

### Service Control Policies (SCP):
  * Policies for OUs to manage permissions within, helping accounts stay within control setting limits/guard rails
  * No permissions are granted by SCP, still IAM, but the effective permissions are the intersection of IAM, SCP, and IAM permissions boundaries allowing access
  * OU must have all features enabled to utilize.
  * Affects member accounts and attached users and roles within including the root user(s), not management accounts.
  * Doesn't affect resource based policies directly.
  * Doesn't affect service linked roles, which enable other AWS services to integrate with AWS OUs.
  * If disabled at the root account, all SCPs are automatically detached from OU under that root account.  If re-enabled all accounts there under are reverted to full AWS Access (default)
  * Must have an explicit allow (nothing allowed by default like IAM) which is similar to IAM permissions boundary (if not in boundary => deny)

## Identity and Access Management (IAM)

Allow vs Deny: If any denial in policy is present, the resource is denied regardless of allow statement(s).  The default behavior is to deny resource(s) and resource(s) need allow statements to be allowed.  

LDAP: software protocol for enabling the location of data about organizations, individuals and other resources in a network.  

Identity federation: a system of trust between two parties for the purpose of authenticating users and conveying information needed to authorize their access to resources.

User groups can only contain users

S3 Bucket Policies vs Access permissions:
  * Used to add or deny permissions across some or all S3 objects in a bucket, enabling central management of permissions
  * Can grant users within an AWS account or other AWS accounts to S3 resources
  * Can restrict based on request time (Date condition), request sent using SSL (Boolean condition), requester IP Address (Ip address condition) using policy keys
  * User access to S3 => IAM permissions
  * Instance (EC2) access => IAM role
  * Public access to S3 => bucket policy

| Type of Access Control | Account Level Control | User Level Control |
| ------------- | ------------- | ------------- |
| IAM Policies | No | Yes |
| ACLs | Yes | No |
| Bucket Policies | Yes | Yes |

### IAM Credentials Report: IAM security tool that lists all your AWS accounts, IAM users and the status of their various credentials; good for auditing permissions at the **account level**

### IAM Access Advisor: shows the service permissions granted to a user and when those services were last used; can use this information to revise policies at the **user level**

### AWS Policy Simulator: used to test and troubleshoot IAM policies that are attached to users, user groups, or resources.  

###  Identity and Access Management Access Analyzer:

### Identity and Access Management Policy Evaulation Logic:
![IAM Policy Evaulation Logic](https://docs.aws.amazon.com/images/IAM/latest/UserGuide/images/PolicyEvaluationHorizontal111621.png)

### Amazon Cognito:
  * Web Identity federation service/identity broker handling interations between application(s)/resource(s) and Web IdPs.
  * Capable of synchronizing data from multiple devices by means of SNS to send notifications to all devices associated to a given user upon data deltas (IAM policy can be tethered to user ids possibly).
  * User pool: user based; handling user registration, authentication and account recovery.
    * Compatible IdPs: Facebook, Amazon, Google, Apple, OpenID Connect providers, SAML
  * Identity pool: receives authentication token to authorize access to resources directly or through the API GW.
    * Maps to IAM role(s)
    * default IAM role(s) for authenticated/guest users

### AWS Resource Access Manager (RAM):
  * Share AWS resources that you own with other AWS accounts (within OU or any account)
  * Aids in avoiding resource duplication by sharing thing such as:
    * VPC subnets (owner can share +1 subnets with other accounts in the same OU):
      * Allows all resources (EC2, etc.) launched in the same VPC
      * Must be from the same OU
      * Can't share SGs and default VPC
      * Users can manage own resources, but can't modify, view, or delete other's resources
      * VPC by itself can't be shared
    * AWS Transit Gateway
    * Route 53 Resolver Rules
    * Licence Manager Configurations accross accounts using Private IP(s)

## Networking

### Load Balancers

#### Application Load Balancer (ALB):
  * Best suited for load balancing HTTP(s) traffic, operating @layer 7 (websockets)
  * HTTP traffic following the load balancer doesn't need encrpytion => port 80
  * Has a static DNS name (not ip) => uses private ip of associated ENI for requests forwarded to web server
  * Reserved ALB cookie names: AWSALBAPP, AWSALBTG, AWSALB
  * Can load multiple SSL certificates on one listener that specifies the hostname via SNI
  * Redirects HTTP=>HTTPS
  * Target groups => EC2 (autoscale), ECS, λ, private ip address
  * Health checks are at the target group level
  * Routing based on url, hostname, query string paramters, or headers
  * X-Forwarded-For header for client ip
  * Sticky sessions available, via cookie (AWSALB)
  * Custom Cookie name must be specified for each target group (all cookies < 4KB)
  * Cross zone LB is free and disabled by default
  * Integrates with Cognito User Pools

#### Elastic Load Balancer (ELB):
  * Legacy load balancer that can load balance http(s) applications and use layer-7 specific features, such as X-Forwarded and sticky sessions
  * Can also use strict Layer 4 load balancing for applications that rely purely on TCP protocol
  * HTTP traffic following the load balancer doesn't need encrpytion => port 80
  * Has a static DNS name (not ip) => uses private ip of associated ENI for requests forwarded to web server
  * Health checks are TCP or HTTP
  * Sticky sessions available via cookie (AWS ELB) [all cookies < 4KB]
  * Cross zone LB is free and disabled by default
  * SNI not available, supports only one certificate

#### Gateway Load Balancer:
  * Used to scale, deploy, and manage a fleet of 3rd party network virtual appliances in AWS (eg: Firewalls, Intrusion Detection and Prevention System[s], Deep Packet Inspection System[s], payload manipulation)
  * Operates @layer 3 (Network layer) ip packets
  * Combines *Transparent Network Gateway* - single entry/exit for all traffic and *Load Balancer* -distributes traffic to virtual appliances
  * Use Geneve protocol on port 6081
  * Target groups: EC2, private ip address(es)
  * Cross zone LB is $ per use and disabled by default

#### Network Load Balancer (NLB):
  * Best suited for load balancing of TCP/UDP traffic where extreme performance is required, operating @layer 4
  * Capable of handling millions of requests per second, while mantaining ultra-low latency
  * Has one static, public ip address per AZ (can attach elastic IP to this)
  * Can load multiple SSL certificates on one listener that specifies the hostname via SNI
  * Health checks are TCP or HTTP(S)
  * Target groups: EC2, private ip address(es), ALB
  * X-Forwarded-For header for client ip
  * Cross zone LB is $ per use and disabled by default
  * If targets specified via instance id => primary private ip specified in primary network interface
  * If targets specified via ip(s) => route traffic to instance via private ip from one or more network interfaces, allowing multiple applications to use the same point
  * Doesn't support SG(s), based on target configurations, ip of the client, or the *private ip address(es) of the NLB must be allowed on the web server's SG*

### Amazon CloudFront
  * Serverless service used to scale, save network bandwidth, and deliver entire website(s), including dynamic, static, streaming, and interactive content using a global network of edge locations, sending requests to nearest edge location through such options as:
   * Web distribution - websites (no architecture change)
   * RTMP - used for media streaming
   * Edge Location (λ@edge) where content is cached.  Separate from an AWS Region/AZ.  Can be written to as well
   * Origin-origin of all files CDN distributes (S3 bucket, EC2, ELB, or Route 53)
   * Distribution - name given to CDN consisting of a collection of Edge Locations
  * Can route to multiple origins based on content type (dynamic => ELB, static => S3)
  * Options for securing content: https, geo restriction, signed url/cookie, field level encryption, AWS WAF
  * Can't be associated with SGs
  * Objects can be cached for TTL
  * Can clear cached objects, but you will be charged
  * Price classes: 
   * All Regions - best performance - $$$
   * 200 - $$
   * 100 - $ only least expensive regions
  * Supports primary/secondary origins for HA/failover (specific http responses)
  * Supports field level encryption (not KMS) @edge to protecte sensitive data
  * Serverless, cheaper to scale

### CloudFront Signed URLs/Cookies:
  * Use signed URLs/cookies when you want to secure content to authorized users
  * Signed URL is for 1 individual file 
  * Signed Cookie is for multiple files (sitewide)
  * If origin is EC2, use CloudFront Signed URLs/Cookies
  * If origin is S3, use S3 Signed URL (same IAM access as creator's IAM)
  * Policies:
   * Limited life time
   * IP ranges
   * Trusted signers (which AWS accounts can create signed URLs)

### AWS PrivateLink:
  * Service allowing you to open your service in VPC to another VPC (using privatelink)
  * Allows exposure of VPC service to tens, hundres, or thousands of other VPCs
  * Doesn't require VPC peering; no route tables, NAT, IGWs, etc.
  * Requires a NLB on the service VPC and an ENI on the customer VPC
  * If issue(s): Check DNS setting Resolution in VPC and/or check the the Route Tables

### AWS Direct Connect
  * Service that makes it easy to establish a dedicated network connection from on-premises to AWS VPC.  
  * Using Direct Connect, you can establish private connectivity between AWS VPC and your datacenter/co-location environment, which can reduce network costs, increase bandwidth throughput, and provide a more consistent network experience than internet based connections
  * Can access private/public AWS resources
  * Useful for high throughput workloads (eg: lots of network traffic)
  * Good if you need a stable and reliable secure connection
  * Takes at least a month+ to setup
  * Supports IPV4/6
  * Can use Direct Connect Gateway if Direct Connect available between one VPC already
  * AWS Direct Connect + VPN => encrypted
  * Dedicated: 1 GBps to 10 GBps
  * Hosted: 50 MBps, 500 MBps, up to 10 GBps

## EC2

### Instance Purchasing Options:

#### EC2 On-Demand Instance:
  * Pay by the second for the instances launched after first minute
  * Can't be used for existing server-bound software licenses

#### EC2 Savings Plan Instance:
  * 1 or 3 year terms available
  * Committment to an amount of usage, long workload
  * Beyond usage is billed at the On-Demand rate
  * Locked to a specific instance family/region
  * Flexible across: instance size, OS, Tenancy (Host, Dedicated, Default)
  * Can be shared across AWS Organization accounts

#### EC2 Reserved Instance:
  * Committed/consistent instance configuration, including instance type and region for a term of 1 or 3 years
  * Great for cost optimization
  * Can be shared across AWS Organization accounts
  * Can't be used for existing server-bound software licenses

#### EC2 Convertible Reserved Instance:
  * One of the reserved instance purchasing options (1 or 3 years)
  * Good for long workloads with flexible instance type(s)
  * Can change EC2 type, instance family, OS, scope and tenancy

#### EC2 Dedicated Instance:
  * EC2 instances that run in a VPC on hardware dedicated to customer use
  * Physically isolated at the hose hardware level from instances belonging to other AWS accounts
  * May share hardware with other instances from the same AWS account that aren't dedicated instances
  * Can't be used for existing server-bound software licenses

#### EC2 Dedicated Host Instance:
  * EC2 instances on physical servers dedicated for customer use
  * Give additional visibility and control over how instances are placed on a physical server over time
  * Enables the use of existing server-bound software licences and address compliance/regulatory requirements
  * On-demand option: pay per second ($$$)
  * Reserved option: 1 or 3 year options with other reserved instances ($ - $$)

#### EC2 Capacity Reserved Instance:
  * Reserve On-Demand capacity in a specific AZ for any duration
  * No time committment (create/cancel anytime); no billing discounts and charged On-Demand rate regardless
  * Combine with Regional Reserved Instances and Savings Plans to benefit from billing discounts
  * Suitable for short-term, uninterrupted workloads that need to be in an AZ

#### EC2 Spot Instance:
  * Most cost-efficient EC2 instance, up to 90% off On-Demand rate
  * Can lose at any time if your max price < the current spot price
  * Not suitable for critical jobs or DBs
  * Useful for jobs such as: batch jobs, data analysis, image processing, any distributed workloads, workloads with a flexible start/end time
  * Can only cancel Spot Requests that are open, active or disabled.  Cancelling a Spot Request doesn't terminate the instances; you must first cancel the Spot Request and then terminate the Spot Instances
  * Don't use if being up for a specific time frame is necessary

#### EC2 Spot Fleets:
  * set of Spot Instances and optional On-Demand Instances
  * Will try to meet target capacity with price constraints defined via possible launch pools, instance type, OS, AZ
  * Can have multiple launch pools from which the fleet can choose
  * Stops launching instances when reaching capacity or max cost
  * Allow automatically to request Spot Instances with the lowest price
  * Strategies to allocate Spot Instances:
    * *Lowest Price*: from the pool with lowest price (cost optimization, short workloads)
    * *Diversified*: distributed across all pools (great for availability, long workloads)
    * *Capacity Optimized*: pool with the optimal capacity for the number of instances

#### EC2 Spot Blocks (aka Spot Duration):
  * "block" spot instance during a specified time frame (1 to 6 hours) without interruptions
  * In rare situations, instance may be reclaimed
  * Not available to new customers

#### EC2 SG configurations:
  * source (inbound rules) or destination (outbound rules) for network traffic specified via the following options:
    * Single ipv4 (/32 CIDR) or ipv6 (/128 CIDR)
    * Range of ipv4/ipv6 address (CIDR block notation)
    * Prefix List ID for the AWS service(s) via Gatway Endpoints, (eg: p1-1a2b3c4d)
    * Another SG, allowing instances within one SG to access instances within another.  Choosing this option doesn't add rules from the source SG, to the 'linked' SG.  You can specify one of the following:
     * Current SG
     * Different SG for the the same VPC
     * Different SG for a peer VPC in VPC peering connection

### EC2 Placement Groups

#### Cluster Placement Group:
  * Clusters instances into a low-latency group in a single AZ
  * Great network speeds (10 GBps bandwidth between instances with enhanced networking enabled; recommended)
  * If the rack fails, all instances fail at once
  * Use cases:
    * Big data job that needs to complete fast
    * Application that needs extremely low latency and high network throughput

#### Partition Placement Group:
  * Spreads instances across many different partitions (which rely on different sets of racks) within an AZ
  * Scales to 100s of instances per group
  * Up to 7 partitions per AZ and can span multiple AZs in the same region
    * Instances in a partition don't share racks with the instances in other partitions
    * Individual partition failure can affect many EC2, but won't affect other partitions
    * EC2 instances get access to the partition meta data
  * Use cases: HDFS, HBase, Cassandra, Kafka, Hadoop (distributed and replicated workloads)

#### Spread Placement Group:
  * Spreads instances across underlying hardware (max 7 instances per group per AZ)
  * Can span across AZs
  * Reduced risk of simultaneous failure
  * EC2 instances on different physical hardware
  * Use cases:
    * Maximal HA
    * Critical Application where each instance must be isolated from failure from each other

### EC2 Instance Recovery: same private IP, public IP, elastic IP, metadata, placement group, instance ID after failure

### EC2 User Data:
  * Used to perform common automated, dynamic, configuration tasks and even run scripts after and instance starts
  * When an EC2 launches, you can pass 2 types of user data, shell script(s) and cloud-init directives .  This can be passed into the launch wizard as plain text or a file
  * *By default, shell scripts entered as user data are executed with root priveleges (no sudeo command necessary)*
  * If non-root users are to have file access, modify the permissions accordingly via shell scripting
  * *By default, user data runs only during the boot cycle when the EC2 is first launched*
  * Alternatively, configuation(s) can be updated to ensure user data scripts and cloud init directives can run every time an EC2 instance is restarted

### EC2 Hibernate:
  * In-memory state is preserved
  * Instance boot time is much faster (OS is not stopped/restarted)
  * Memory state is written to a file in root EBS
  * Root EBS must be encrypted
  * Can't use hibernate beyond 60 days
  * Available for the following EC2 Options: On-Demand, Reserved, and Spot Intances
  * Use cases: long-running processing, saving memory state, services that take a long time to initialize

## Containers

### Amazon Elastic Container Registry (ECR):
  * private/public container storage alternative to docker hub
  * container =>image
  * Backed by S3
  * Access controlled by IAM
  * Harnessed by ECS, EKS, Fargate
  * Supports vulnerability scans, versioning, image tags, image lifecycle

### Amazon ECS:
  * Has to launch types: Amazon EC2 and Fargate
  * Fargate has to use either EFS volume, FSx for Windows, docker volumes, or bind mount (file or dir)
  * EC2 has to harness an EFS volume amounts instances, not EBS
  * ECS Autoscaling=>target tracking metrics:
    * ECSSVCAVECPU
    * AveCPU use
    * ECSSVCAVEMEM-average memory use
    * ALBRequestCountPerTarget-number of requests per target in ALB target group
  * Can be scaled in/out per EventBridge invoked rules/schedule
  * IAM roles: 
    * EC2 Instance Profile (EC2 only)-used by ECS agent, makes API calls to ECS service, send container image from ECR, reference sensitive data in Secrets Manager or SSM Parameter Store
    * ECS Task Role-allows each task to have a specific role, use different roles for different ECS services you run, task role is defined in the task definition

### Amazon EKS:
  * Managed node groups: 
    * Create/Manage nodes (EC2) 
    * Nodes are part of ASG managed by EKS
    * Supports On-Demand or Spot Instances
  * Self managed nodes:
    * Nodes created by you, registered to the EKS cluster and managed by an ASG
    * Can use prebuilt AMI-Amazon EKS Optimized AMI
    * Supports On-Demand and Spot Instances
  * Need to specify storage class on EKS cluster, leveraging Container Storage Interface (CSI) compliant driver: EBS, EFS (Fargate), FSx for Lustre/for NetApp ON TAP
  * Doesn't support λ, does support Fargate, Managed Node Groups, and Self-Managed Nodes
  
### AWS Fargate:
  * Severless backend harnessing ECS/EKS/ECR
  * VPCU
  * Memory
  * AWS runs ECS Tasks for you based on CPU/RAM needed, so to scale, increase the number of tasks
  * Storage resources (20 GB free)
  * No time limit like λ (15 minutes)
  * λ can hit a max concurrency problems, or sometimes can't horizontally scale and might throttle, through with error code 429
  * Harnessed EFS, doesn't support mounting EBS volumes

### Amazon Managed Service for Prometheus: serverless monitoring service harnessing PROMQL to monitor and alert on container environments upon ingestion/storage

## Logging and Events

### AWS CloudTrail:
  * Service that monitors and records account activity across AWS infrastructure (history of events/API calls)
  * Provides governance, compliance and audit for your AWS account:
   * Enabled by default
   * Trail can be applied to all regions (default) or a single region

### CloudTrail Events:
  * Able to be separated into read/write events
  * Management events (default on)
  * Data events (default off due to volume, though can be turned on to trigger/invoke)
  
### CloudTrail Insights:
  * Used to detect unusual activity in account (if enabled):
   * Inaccurate resource provisioning
   * Hitting service limits
   * Bursts of AWS IAM actions
   * Gaps in periodic maintenance
   * Analyzes normal manangement events to create a baseline to then continuosly analyze write events to detect unusual patterns (S3/CloudTrail console/EventBridge events)
   * Cloudtrail Events are stored for 90 days, though can be sent to S3 and analyzed by Athena

## Configurations and Security

### AWS Certificate Manager (ACM):
  * Service to provision , manage, import, and deploy public and private SSL/TLS certificates for use with AWS services and internally connected resources
  * Removes time consuming manual process of purchasing, uploading, and renewing SSL/TLS certificates
  * Manages certificate lifecyle centrally
  * Private Certificates use is charged a monthly fee
  * Automatically renews certificates issued by ACM

### Systems Manager (SSM) Parameter Store:
  * Can be used to store secrets
  * Each time you edit the value of a parameter a new version is created via built-in version tracking and previous version committed historically
  * Optional seamless encryption with KMS
  * Serverless, scalable, durable, easy SDK
  * Security through IAM
  * Notifications via Eventbridge
  * Can assign TTL to a parameter to force update(s) or deletion (via Policies)

|  | Standard | Advanced |
| ------------- | ------------- | ------------- |
| Total # of parameters per AWS Account-Region | 10,000 | 100,000 |
| Max Size of Parmater | 4 KB | 8 KB |
| Parameter Policies | No | Yes |
| Cost | No | Charges Apply |
| Storage Pricing | Free | .05 per advanced parameter per month |


### AWS Secrets Manager:
  * Helps manage, retrieve and rotate DB credentials, API keys, and other secrets throughout lifecycles
  * Automatic generation of secrets on rotation (uses a λ)
  * Encrypted at rest possible (via KMS)
  * Integrates with RDS
  * Auditable via CloudTrail, Cloudwatch, and SNS
  * Multi-Region Secrets - read replicas in sync with Primary Secret
  * Able to promote read replica secret to standalone secret (eg: multi-region application DB distaster recovery)
  * SSM parameter store can be utilized if wanting to track versions/values of secrets
  
### Amazon GuardDuty:
  * Threat detection service that continuously monitors AWS accounts and workloads for malicious activity to deliver detailed security findings for visibility and remediation
  * Helps against the following:
    * Can protect against CryptoCurrency attacks (has a dedicated "finding" for it)
    * Anomaly detection via ML
    * Malware scanning
    * AWS accounts
    * EC2
    * EKS
    * S3
    * EBS (malware scan[s])
  * Scans these data sources:
    * CloudTrail Events log
    * CloudTrail S3 data event logs
    * VPC Flow logs
    * DNS query logs
    * EKS audit logs
  * Disabling will delete all data, while suspending will stop analysis, but not delete

### Amazon Macie:
  * Security service using ML/NLP to discover, classify, and protect sensitive data stored in S3, such as:
   * PII
   * Dashboards/reports/alerts
   * Can also analyze CloudTrail logs (for suspicious activity)

### AWS WAF
  * *Can protect: Cloudfront, API Gateway, ALB, Appsync, Cognito User Pool
  * Web application firewall that monitors HTTP/HTTPS requests forwarded to Cloudfront, ALB, or API Gateway and lets you control access to content (via Web ACL rules)
    * Web ACL rules can be associated with Cloudfront, but not an S3 Bucket policy
    * Web ACL rules can't be associated with a NACL
    * Web ACL are regional except for Cloudfront
  * Rules can involve:
    * Up to 10000 IP rules
    * Ip rate based rules
    * Level 7 (HTTP/HTTPS)
    * IP address request origin
    * Country request origin (geo-match)
    * Values in request header(s)
    * String in request (match or regexp)
    * Length of request
    * SQL presence (SQL injection)
    * Script presence (XSS)
 
### AWS Shield: 
  * DDOS protection service that safeguards application running on AWS (layer 3/4)
  * Free for all AWS customers
  * AWS Shield Advanced costs $, but more sophisticated services concerning: ELB, EC2, CloudFront, Global Accelerator, Route 53

### AWS Firewall Manager:
  * A service to manage rules in all accounts of an AWS organization (firewall related rules)
  * Common set of security rules:
    * WAF rules (ALB, API Gateways, CloudFront)
    * AWS Shield Advanced (ALB, CLB, Elastic IP, CloudFront)
    * SGs for EC2 and ENI resources in VPC
    * Amazon Network Firewall (VPC Level)
    * Amazon Route 53 Resolver DNS Firewall
    * Policies are created at the region level
  * Rules are applied to new resources as they are created (good for compliance) across all and future accounts in Organization

### AWS Network Firewall:
  * Protect your entire Amazon VPC with a firewall from *layer 3 to layer 7 protection*
  * Traffic filtering: Allow, drop, alert to rule matches
  * Can be centrally managed cross-account/VPC by AWS Firewall Manager
  * Can inspect:
    * To/From traffic
    * Outbound to/Inbound from the internet
    * VPC to VPC traffic
    * To, From Direct Connect and Site to Site VPN
  * Internally, AWS Network Firewall uses the AWS Gateway Load Balancer
  * Supports 1000s of rules
  * IP and port blocking
  * Protocol (eg: block SMB outbound traffic)
  * Stateful domain list rule groups (eg: allow outbound traffic to *.mysite.com or 3rd party software repository)
  * Regex pattern matching
  * Active flow inspection to protect against network threats with Intrusion prevention capabilities
  * Sends logs of rule matches to S3, Cloudwatch logs, Kinesis Data Firehose

### WAF vs Firewall Manager vs Shield:
  * All used together for comprehensive protection
  * Define your Web ACL rules in WAF
  * For granular protection of your resources, WAF alone is the correct choice
  * If you want to use AWS WAF across accounts, accelerate WAF configuration, automate the protection of new resources, use Firewall Manager with AWS WAF
  * Shield Advanced adds additional features on top of AWS WAF such as dedicated support from the Shield Response Team (SRT) and advanced reporting
  * If prone to frequent DDOS=>Shield Advanced

### Security Groups (SGs):
  * Stateful connection, allowing inbound traffic to the necessary ports, thus enabling the connection
  * If adding an Internet Gateway, ensure the SG allows traffic in
  * SG => EC2 instances level, LBs, EFS, DBs, Elasticache
  * Allow rules only

### NACL Groups:
  * Stateless, thus a source port inbound will become the outbound port (or possibly taking the defined port and responding via an ephemeral port)
  * Great way of allowing/blocking ip addresses at the subnet level
  * Like a firewall controlling to/from subnet traffic
  * One NACL per subnet
  * New Subnet automatically set to default NACL which denies all inbound/outbound traffic
  * Do not modify default NACL, instead create custom NACL(s)
  * If accepting internet traffic routed via internet gateway
  * If accepting vpn or AWS Direct Connect traffic routed via Virtual Private Gateway
  * NACL rules:
    * Range from 1-32766, with a higher precedence placed on lower numbers
    * Allow and Deny rules
    * First rule match drives acceptance/denial
    * Last rule match is a catch all (\*) and denies a request in case no rules match
    * AWS recommends adding rules by an increment of 100

## VPC

### Launch Configuration (tenancy vs VPC tenancy):
  * Part of the instance config launching instances distributed across physical hardware
  * Tenancy (instance placement) default is null and the instance tenancy is controlled by the tenancy attribute of the VPC
  * Shared (default): multiple AWS accounts may share the same physical hardware
  * Dedicated Instance (dedicated): single tenant hardware used
  * Dedicated Host (host): instances run on a physical server with EC2 instance fully dedicated to a given client's use (not available for launch template configuration)

| Launch Configuration Tenancy | VPC tenancy=default | VPC tenancy=dedicated |
| ------------- | ------------- | ------------- |
| null | shared | dedicated |
| default | shared | dedicated |
| dedicated | dedicated | dedicated |

### Amazon VPC console wizard configurations:
  * VPC with public/private subnets (NA)
  * VPC with public/private subnets and AWS Site-to-Site VPN access
  * VPC with a single public subnet
  * VPC with a private subnet only and AWS Site-to-Site VPN access
  
### VPC Traffic Mirroring:
  * Allows you to capture/inspect network traffic in VPC
  * Route the traffic to security appliance(s) you manage
  * Capture the traffic: -From (source) ENIs -To (Target) an ENI or NLB
  * Capture all packets or packets of interest (optionally truncating packets)
  * Source and target can be the same or different VPCs (VPC peering)
  

### Direct Connect Gateways: Enables connections to many VPCs in different AWS regions

#### Vitual Private Gateway:
  * VPN Connector on the AWS side of the VPN connection
  * Created and attached to the VPC from which you want to create site-to-site-VPN
  * Necessary for setting up AWS Direct Connect

#### AWS VPN (AWS Site-to-Site VPN):
  * Establish on-ongoing VPN connection between on-premises data center (customer gateway) and Amazon VPC (VGW)
  * Utilizes IPSec (encrypted tunnel) to establish encrypted network connectivity between your internet and Amazon VPC over the internet
  * Connection can be configured in minutes and is a good solution if you have an immediate need, have low to modest bandwidth requirements and can tolerate the inherent variability of Internet-based connectivity
  * Need to configure VGW and Customer Gateway
  * Ensure Route Propagation enabled for VGW in the route table associated with your subnets
  * If ping needed (EC2) from on-premises, add ICMP protocol on the inbound SGs
### Transit VPC:
  * Uses customer managed EC2(s) VPN instances in a dedicated transit VPC resources with an IGW
  * Data transfer charged for traffic traversing this VPC an again from the transit VPC to on-premises network or different AWS regions
  * Consider AWS Transit Gateway (shared services VPC) as a cheaper and less maintenance alternative

### VPC Peering: 
  * Privately connects 2 VPCs on AWS's network, behaving as if the same network (works in different AWS accounts/regions)
  * Must not a have overlapping CIDRs
  * Not transitive (must be established for each VPC to communicate)
  * Must update route tables in each VPCs subnets to ensure communication between each's EC2 instance

### AWS Transit Gateway (Shared services VPC):
  * Allows transitive peering between thousands of VPCs and on-premises data centers
  * Works on a hub-and-spoke model
  * Works primarily on a regional basis, though accessible acorss multiple regions
  * Can be used across multiple AWS accounts using AWS RAM
  * Can use route tables to limit VPCs communication between one another
  * Works with Direct Connect as well as VPN connections
  * Supports IP multicast (not supported by any other AWS service)
  * Site-to-site VPN ECMP: creates multiple connections to increase bandwidth of connection to AWS

### Internet Gateway (IGW):
  * Allows resources (EC2, etc.) in a VPC to connect to the internet (IPV4/6) via route table
  * Scales horizontally, is HA and redundant
  * One VPC per IGW
  * VPC route tables need to be edited to allow internet access
  * Can't be used directly in a private subnet without NAT instance or gateway in a public subnet
  * If subnet hooked via route table up to the IGW => public
  * Acts as (NAT) for instances that are assigned public IPV4 address(es)
  * Must be created separately from a VPC

### NAT Gateway:
  * Used in a *public subnet* in a VPC, enabling instances in a *private subnet* to initiate outbound IPV4 traffic to the internet/other AWS services, but prevents inbound traffic to instances from the internet
  * Fully managed by AWS
  * Tied to AZ/subnet(s), eg: can't span multiple AZ, multiple NGW are necessary for fault tolerance/failover
  * Uses an Elastic IP
  * Requires IGW (Private subnet => NATGW => IGW)
  * No SG
  * Can't be used by EC2 in the same subnet

### NAT Instance:
  * Used in a *public subnet* in a VPC, enables instances in a *private subnet* to initiate outbound IPV4 to the internet/other AWS services, but prevents inbound traffic to instances from the internet
  * Can be used as a bastion host/server
  * Allows port forwarding
  * Managed by user (software, patches, etc.)
  * Tied to AZ/subnet(s), eg: can't span multiple AZ
  * Must disable EC2 setting: Source/Destination check
  * Must have an Elastic IP
  * Route tables need configuration to route traffic from private subnets to the NAT instance
  * Must manage SG and rules: 
    * *Inbound*-HTTP(S) from private subnets, allow SSH from the home network
    * *Outbound*-HTTP(S) to the internet

### VPC Endpoint:
  * Every AWS service is publicly exposed (public url)
  * VPC Endpoints (using AWS PrivateLink) allows connections to AWS service(s) using a private network instead of public internet
  * Redundant and scales horizontally
  * Removes the need for IGW, NATGW, etc. to access AWS service(s)
  * In case of issues: 
    * Check DNS setting resolution in VPC 
    * Check Route tables
  * Types of Endpoints:
    * Interface Endpoints: provisions an ENI (private ip) as an entry point (must attach a SG); supports most AWS services; powered by Private Link
    * Gateway Endpoints: provisions a gateway and must be used as a target in Route table; supports both *S3 and DynamoDB*
  * Gateway Endpoints are preferred most of the time over Interface Endpoints as the former is free and the latter costs $
  * Interface endpoint is preferred if access is required from on-premises (site-to-site VPN or Direct Connect), a different VPC or a different region

### VPC Flow Logs:
  * Capture information about IP traffic going into your interfaces, such as Subnet Flow Logs or ENI Flow Logs
  * Helps to monitor/troubleshoot connectivity issues
  * Can go to S3 or Cloudwatch Logs
  * Capture network information from AWS managed interfaces, too: ELB, RDS, Elasticache, Redshift, Workspaces, NATGW, Transit Gateway
  * Query via Athena on S3 or Cloudwatch Log Insights (good for troubleshooting SG and NACL issues)

### Bastion Host/Server
  * An EC2 used to SSH into private EC2 instances
  * Is the public subnet which is connected to all other private subnets
  * SG must be tightened
  * Should only have port 22 on the public CIDR available
  * NAT instances can be used as a bastion server/host
  * SG should only allow inbound from the internet on port 22
  * SG of instances must allow SG of the Bastion Host or the private ip of the Bastion host access
  
### Egress-only Internet Gateway:
  * Similar to NAT Gateway, but for ipv6 only
  * Allows instances in VPC outbound connections over ipv6 while preventing the internet to initiate an ipv6 connection to your instance(s)
  * Must update route tables

## Storage

EBS:
  * Volumes exist on EBS => virtual hard disk
  * Snapshots exist on S3 (point in time copy of disk)
  * Snapshots are incremental-only the blocks that have changed since the last snapshot are move to S3
  * First snapshot might take more time
  * Best to stop root EBS device to take snapshots, though you don't have to
  * Provisioned IOPS (PIOPS [io1/io2])=> DB workloads/multi-attach
  * Multi-attach (EC2 =>rd/wr)=>attach the same EBS to multiple EC2 in the same AZ; up to 16 (all in the same AZ)
  * Can change volume size and storage type on the fly
  * Always in the same region as EC2
  * To move EC2 volume=>snapshot=>AMI=>copy to destination Region/AZ=>launch AMI
  * EBS snapshot archive (up to 75% cheaper to store, though 24-72 hours to restore)

EFS: 
  * Linux based only
  * Can mount on many EC2(s)
  * Use SG control access
  * Connected via ENI
  * 10GB+ throughput
  * *Performance mode* (set at creation time): 
    * General purpose (default); latency-sensitive; use cases (web server, CMS); 
    * Max I/O-higher latency, throughput, highly parallel (big data, media processing)
  * *Throughput mode*: 
    * Bursting (1 TB = 50 MiB/s and burst of up to 100 MiB/s)
    * Provisioned-set your throughput regardless of storage size (eg 1 GiB/s for 1 TB storage)
  * *Storage Classes*, Storage Tiers (lifecycle management=>move file after N days):
    * Standard: for frequently accessed files
    * Infrequent access (EFS-IA): cost to retrieve files, lower price to store
  * *Availability and durability*: 
    * Standard: multi-AZ, great for production
    * One Zone: great for development, backup enabled by default, compatible with IA (EFS One Zone-IA)

### AWS FSx:
  * Launch 3rd party high performance file system(s) on AWS
  * Can be accessed via FSx File Gateway for on-premises needs via VPN and/or Direct Connect
  * Fully managed
  * Accessible via ENI within Multi-AZ
  * Types include:
    * FSx for Windows FileServer
    * FSx for Lustre
    * FSx for Net App ONTAP (NFS, SMB, iSCSI protocols); offering:
     * Works with most OSs
     * ONTAP or NAS
     * Storage shrinks or grows
     * Compression, dedupe, snapshot replication
     * Point in time cloning
    * FSx for Open ZFS; offering:
     * Works with most OSs
     * Snapshots, compression
     * Point in time cloning

### Amazon FSx for Windows:
  * Fully managed Windows file system share drive
  * Supports SMB and Windows NTFS
  * Microsoft Active Directory integration, ACLs, user quotas
  * Can be mounted on Linux EC2 instances
  * Scale up to 10s of GBps, millions IOPs, 100s of PB of data
  * Storage Options:
   * SSD - latency sensitive workloads (DB, data analytics)
   * HDD - broad spectrum of workloads (home directories, CMS)
  * On-premises accessible (VPN and/or Direct Connect)
  * Can be configured to be Multi-AZ
  * Data is backed up daily to S3
  * Amazon FSx File Gateway allows native access to FSx for Windows from on-premises, local cache for frequently accessed data via Gateway

### Amazon FSx for Lustre ("Linux" "Cluster"):
  * High performance, parallel, distributed file system designed for Applications that require fast storage to keep up with your compute such as ML, high peformance computing, video processing, Electronic Design Automation, or financial modeling
  * Integrates with linked S3 bucket(s), making it easy to process S3 objects as files and allows you to write changed data back to S3
  * Provides ability to both process 'hot data' in parallel/distributed fashion as well as easily store 'cold data' to S3
  * Storage options include SSD or HDD
  * Can be used from on-premises servers (VPN and/or Direct Connect)
  * Scratch File System can be used for temporary or burst storage use
  * Persistent File System can be used for storage / replicated with AZ

### AWS Storage Gateway:
   * Hybrid cloud storage service that provides on-premises access to virtual cloud storage
   * Most recently used date cached in gateway
   * Volume backend up by EBS snapshots
   * Tape backed up by S3, S3 (Glacier), and other software
   * S3 access supported: STD, STD IA, One Zone IA, Intelligent tiering
   * SMB/NTFS access able to integrate with Windows AD
   * Conducted and connected with AWS by: VM or physical hardware device

| Gateway | Protocol(s) |
| ------------- | ------------- |
| S3 (File) | NFS/SMB |
| FSx | SMB/NTFS |
| Tape (interface) | iSCSI |
| Volume (interface) | iSCSI |

### S3

#### S3 Batch Replication:
  * Provides a way to replicate objects that existed before a replication configuration was in place, objects that have previously been replicated, and those that failed replication
  * Differs from live replication
  * Can't use AWS S3 console to configure cross-region replication for existing objects.
  * By default, replication only supports copying new S3 objects after enabled by AWS S3 console
  * To enable live replication of existing objects, an AWS support ticket is needed to ensure replication is configured correctly
  * Can be used to copy large amounts of S3 data between regions or within regions

#### S3 Sync Command:
  * Uses copy object APIs to copy objects between S3 buckets
  * Lists source and target buckets to identify objects found in the former and not the latter as well as last modified dates that are different.
  * On a versioned bucket copies only the current version (previous copies are not copied)
  * If operation fails, can run the command again without duplicating previously copied objects
  * Can be used to copy large amounts of S3 data between regions

#### Origin Access Control (OAC):
  * Restricts access to S3 preventing public availablity, ensures access through intended Cloudfront distribution(s) [no direct access]
  * Is Replacing OAI
  * Supports all S3 buckets in all regions
  * Supports AWS KMS (SSE-KMS)
  * Dynamic requests (PUT, POST, etc.) to S3
  * Can be used to only allow authenticated access (via Cloudfront configuration), not done via OAC directly
  
#### Origin Access Identity (OAI):
  * Restricts access to S3 preventing public availablity, ensures access through intended Cloudfront distribution(s) [no direct access]
  * Is being replaced by OAC
  * Can be used to only allow authenticated access (via Cloudfront configuration), not done via OAI directly

## Database

### Relational

#### RDS Autoscaling:
  * Supports all RDS
  * Max storage threshold is the threshold you set for the auto scaling the DB instance
  * If workload is unpredictable, it is good to enable RDS auto scaling
  * Aurora migration involves significant system administration effort to migrate from RDS mysql to aurora and if auto-scaling is all that is necessary, it is the better/easier option
  * Autoscales if: 
   * Free space < 10% allocated space
   * 6 hours have passed since last modifications
   * low-storage lasts at least 5 minutes

#### Amazon RDS Proxy:
  * Fully managed DB proxy for RDS and Aurora
  * Good to use with λ under high load to avoid too many connections
  * Allows applications to pool and share DB connections with an established DB
  * Improves DB efficiency by reducing stress on DB resources (eg: CPU, RAM) and minimize open connections ( and timeouts)
  * Serverless, autoscaling, HA, multi-AZ
  * Reduced RDS and Aurora failover time by up to 66%
  * Supports RDS (MySql, Postgres, Mariadb) and aurora (MySql/Postgres)
  * No code changes required for most applications
  * Enforce IAM authorization for DB and securely store credentials in AWS Secrets Manager
  * RDS Prosy is never publicly accessible (must be accessed via VPC)

#### Aurora:
  * MySQL or Postgres
  * Better performance than RDS version
  * Lower price
  * At rest encryption via KMS
  * 2 copies in each AZ, with a minimum of 3 AZ => 6 copies
  * Shareable snapshots with other accounts
  * Replicas: MySQL, Postgres, or Aurora
  * Replicas can autoscale
  * Cross region replication (< 1 second) support available 
    * Aurora Global: multi region (up to 5)
    * Aurora Cloning: copy of production (faster than a snapshot)
  * Aurora multimaster (for write failover/high write availability)
  * Aurora serverless for cost effective option (pay per second) for infrequent, intermittent or unpredictable workloads
  * Automated backups
  * Automated failover with Aurora replicas 
    * Fail over tiers: lowest ranking number first, then greatest size
  * Aurora ML: ML using SageMaker and Comprehend on Aurora

#### Amazon Redshift:
  * fully managed, scalable cloud data warehouse, columnar instead of row based (no Multi-AZ, based on Postgres, No OLTP, but OLAP)
  * Can be server less or use cluster(s)
  * Uses SQL to analyze structured and semi-structured data across data warehouses, operational DBs, and data lakes
  * Integrates with quicksight or Tableau
  * Leader node for query planning, results aggregation
  * Compute node for performing queries to be sent back to the leader
  * Provisioning node sizes in advance
  * Enhanced VPC Routing
  * Forces all COPY and UNLOAD traffic moving between your cluster and data repositories through your VPCs, otherwise over the internet routing, including to other AWS servicesq
  * Can configure to automatically copy snapshots to other Regions
  * Large inserts are better (S3 copy, firehose)

#### Amazon Redshift Spectrum:
  * Resides on dedicated Amazon Redshift servers independent of your cluster
  * Can efficiently query and retrieve structured and semistructured data from files in S3 into Redshift Cluster tables (points at S3) without loading data in Redshift tables
  * Pushes many compute intensive tasks such as predicate filtering and aggregation, down to the Redshift Spectrum layer
  * Redshift Spectrum queries use much less of the formal cluster's processing capacity than other queries

### NoSQL

#### Amazon DocumentDB:
  * Effectively the AWS "Aurora" version of MongoDB
  * Used to store query and index JSON data
  * Similar "deployment concepts" as Aurora
  * Fully managed, HA with replication across 3 AZs
  * Storage automatically grows in increments of 10 GB, up to 64 TB
  * Automatically scales to workloads with millions of requests per secon
  * Anything related to MongoDB => DocumentDB
  * Doesn't have an in-memory caching layer => consider DynamoDB (DAX) for a NoSQL approach

#### DynamoDB:
  * (Serverless) NoSQL Key-value and document DB that delivers single-digit millisecond performance at any scale.  It's a fully managed, multi-region, multi-master, durable DB with built-in security, backup and restore, and in-memory caching for internet scale applications
  * Stored on SSD
  * Stored across 3 geographically distinct data centers
  * Eventual consitent reads (default) or strongly consistent reads (1 sec or less)
  * Session storage alternative (TTL)
  * IAM for security, authorization, and administration
  * Primay key possibilities could involve creation time
  * On-Demand (pay per request pricing) => $$$
  * Provisioned Mode (default) is less expensive where you pay for provisioned RCU/WCU
  * Backup: optionally lasts 35 days and can be used to recreate the table
  * Standard and IA Table Classes are available
  * Max size of an item in DynamoDB Table: 400KB
  * Can be exported to S3 as DynamoDB JSON or ion format
  * Can be imported from S3 as CSV, DynamoDB JSON or ion format

#### DynamoDB Global Tables:
  * Makes a DynamoDB table accesible with low latency in multiple regions
  * Active-Active replication
  * Applications can read and write to the table in any region
  * *Must enable DynamoDB Streams* as a pre-requesite
  * DynamoDB Streams have a 24 hour retention, are to a limited # of consumers, and are processed using λ triggers or DynamoDB Stream Kinesis adapter

#### DynamoDB Accelerator (DAX):
  * Fully managed, highly available, in-memory cache with microsecond latency
  * Up to 10x performance improvement, without application logic change(s)
  * 5 minute TTL (default)
  * Reduces request time from milliseconds to microseconds
  * Best and easiest option to improve DynamoDB peformance 
  * If asked about NoSQL with in-memory caching => DAX

## Analytics

### Amazon Kinesis:
  * Platform to send stream data making it easy to load and analyze as well as provide the ability to build your own custom applications for your business needs
  * Output can be classic or enhanced fan-out consumers
  * Accessed via VPC
  * IAM access => Identity-based (used by users and/or groups)
  * Types:
    * Kinesis Data Streams
    * Kinesis Data Firehose
    * Kinesis Analytics
    * Kinesis Video Streams (capture, process and store video streams)

### Amazon Kinesis Data Streams:
  * On-demand capacity mode or Provisioned mode (if throughput exceeded exception => add shard[s])
  * Can have up to 5 parallel consumers
  * Can store between 24 hours and 365 days in shard(s) to be consumed/processed/replayed by another service and stored elsewhere
  * Use fan-out if lag is encountered by stream consumers
  * Shards can be split or merged
  * 1 MB message size limit
  * TLS in flight or KMS at-rest encryption
  * *Can't subscribe to SNS*
  * *Can't write directly to S3*
  * Can output to:
    * Kinesis Data Firehose
    * Kinesis Data Analytics
    * Containers
    * λ
    * AWS Glue

### Amazon Kinesis Data Firehose:
  * Fully Managed (serverless) service, no administration, automatic scaling
  * Can use λ to filter/transform data prior to output (Better to use if filter/tranform with a λ to S3 over Kinesis Data Streams)
  * Near real time: 60 seconds latency minimum for non-full batches
  * Minimum 1 MB of data at a time
  * Pay only for the data going through
  * Can subscribe to SNS
  * No data persistence and must bre immediately consumed/processed
  * Sent to (S3 as a backup or failed case[s]):
    * S3
    * Amazon Redshift (copy through S3)
    * Amazon Elastic Search
    * 3rd party partners (datadog/splunk/etc.)
    * Custom destination (http[s] endpoint)

### Amazon Kinesis Analytics:
  * Fully Managed (serverless)
  * Can use either Kinesis Data Streams or Kinesis Data Firehose to analyze data in kinesis
  * For SQL Applications: Input/Output: Kinesis Data Streams or Kinesis Data Firehose to analyze data
  * For Apache Flink (on a cluster): 
    * Input: Kinesis Data Stream or Amazon MSK
    * Output: Sink (S3/Kinesis Data Firehose)

### Amazon Managed Streaming for Apache Kafka (MSK):
  * Alternative to Amazon Kinesis
  * Fully managed Apache Kafka on AWS
  * Allows creation, updates, or cluster deletion
  * Creates and manages Kafka broker nodes and zookeeper rules
  * Deploy Clusters in VPC multi-AZ (up to 3 for HA)
  * Automatic reconvery from common Apache Kafka failures
  * Storage on EBS as long as needed (added $)
  * MSK Serverless: run without managing capcity, automatically provisioning resources and scaling compute and storage 
  * 1 MB default message size (can be configured to be larger)
  * Kafka topics with partitions (like shards); can only add partitions to a topic
  * Output is plaintext, TLS in-flight or KMS at-rest encryption
  * Consumers: Kinesis Data Analytics for Apache Flink, AWS Glue, Streaming ETL Jobs powered by Apache Spark Streaming, λ, EC2/ECS/EKS

### AWS Glue:
  * Managed ETL service (fully serverless) used to prepare/transform data for analysis
  * Can be event driven (eg: λ triggered by S3 put object) to call Glue ETL
  * Glue Data catalog uses an AWS Glue Data Crawler scanning DBs/S3/data to write associated metadata utilized by Glue ETL, or data discovery on Athena, Redshift Spectrum or EMR
  * Glue Job bookmarks prevent reprocessing old data
  * Glue Databrew-clean/normalize data using pre-built transformation
  * Glue Studio-new GUI to create, run, and monitor ETL jobs in Glue
  * Glue Streaming ETL (built on Apache Spark Structured Streaming)-compatible with Kinesis Data Streaming, Kafka, MSK
  * Glue Elastic Views:
    * Combine and replicate data across multiple data stores using SQL (View)
    * No custom code, Glue monitors for changes in the source data, serverless
    * Leverages a "virtual table" (materialized view)

### EMR:
  * Service to create Hadoop clusters (Big Data) to analyze/process lots of data using (many) instances
  * Supports Apache Spark, HBase, Presto, Flink, etc.
  * Takes care of provisioning and configuration
  * Autoscaling and integrated with Spot Instances
  * Use cases: Data processing, ML, Web Indexing, BigData
  * Node types: 
    * Master Node: manage the cluster, coordinate, manage health-long running process
    * Core Node: run tasks and store data-long running process
    * Task Node (optional): only to run tasks-usually Spot Instances
  * Can have long-running cluster or transient (temporary) cluster
  * Purchasing options: 
    * On-demand: reliable, predictable, won't be terminated
    * Reserved: cost savings (EMR will use if available)
    * Spot instances: cheaper, can be terminated, less reliable

### Amazon Sagemaker:
  * Fully managed service for development/data science to build ML models
  * Typically, difficult to do all the process in one place and provision server(s)
    * label data
    * train and tunde model(s)
    * serve api traffic against the model(s)
    
## Infrastructure as Code (IAC)

### AWS CloudFormation:
  * Service to allow modeling, provisioning, and managing AWS and 3rd party resources via IAC at the fine graine level via template
  * Writting in JSON or YAML
  * Template can't be used to deploy the same template across AWS accounts and regions
  * Can use CloudFormation Stack Designer to visualize the stack components and their relationships
  
### AWS CloudFormation StackSets:
  * Extends the functionality of stacks by enabling you to create, update, or delete stacks across multiple accounts and regions with a single operation
  * Done via Cloudformation template
  * Using an admin account of an 'AWS Org', you define and manage AWS Cloudformation template, using the template as the basis for provisioning stacks into selected target accounts of an AWS Org across specified regions
  * Don't confuse with Cloudformation stacks which is a set of resources created and managed as a single unit from a Cloud formation template and can't be used across accounts or regions

### AWS CodeDeploy:
  * Fully managed deployment service to automate software deployments on compute service (EC2/Fargate/λ/on-premises)
  * Code/Architecture agnostic
  * Specify file(s)/folder(s) to copy and script(s) to run

### AWS Proton:
  * Service harnessing infrastructure as code or template for the sake of deploying, provisioning, monitoring and updating bye developers
  * Cloud Formation at a lower level

## Optimization

### AWS Compute Optimizer:
  * Recommends optimal AWS Compute resources (λ, EC2, EBS) for workloads to reduce costs and improve performance by using ML to analyze historical utilization metrics
  * Helps you choose optimal EC2 types, including those part of Autoscaling group based on utilization

## Miscellaneous

### AWS Amplify:
  * Complete Solution allowing front end/mobile developers to easily build, ship and host full-stack applications on AWS harnessing various AWS services as use cases evolve
  * Can use Flutter, React Native, Native languages, Web=>React, Vue, Angular, Ionic
  * Pre-built UI components
  * Figma => code => bind to data sources
  * Git configurable

### AWS Wavelength: extend vpc and it's resources via desired subnets to include a wavelength zone, embedding within 5G networks providing ultra low latency

### AWS Data Exchange: service in AWS for customers to find, subscribe to, and use third-party data in the AWS Cloud

### AWS Elastic Transcoder: highly scalable/cost efficient service to convert (or 'transcode') media files (video/audio) to format(s) that will be playable on devices, tablets, desktops, etc. The transacation takes place between a source and destination S3 bucket.  

### Amazon Personalize:
  * Fully managed ML service to build realtime personalized recommendations applications
  * Increments in days, not months (no need to train, build ML models)
  * Service ingests via S3 (read data) and/or Amazon Personalize API (realtime data integration)

### Amazon Comprehend (Medical):
  * Serverless NLP service harnessing ML to uncover valuable insights and connections in text
  * Medical version detects PHI via DetectPHI API

### Amazon Outposts: Pool of AWS compute and storage resources deployed at a customer site and is essentially part of an AWS region including:
  * VPC
  * EC2
  * ECS/EKS
  * AWS App Mesh
  * EBS
  * S3
  * Elasticache
  * EMR
  * RDS

### Amazon Polly:
  * Turn text into lifelike speech using deep learning (for talking applications)
  * Customize pronunciation of words with pronunciation lexicons that are harnessed by the Sythesize Speech Operation
  * Can map stylized words and/or acronyms to resultant output
  * Generate more customized output from text marked up with SSML including:
    * breating, whispering
    * emphasis on words
    * phonetic pronunciation

### Amazon Textract:
  * Extracts text, handwriting and data from any scanned documents (eg: forms, tables, etc.) using ML
  * Read from any type of document (PDFs, images, etc.)
  * Good for invoices, financial reports, medical records, insurance claims, taxforms, ids, passports

### Amazon Transcribe:
  * Automatically convert speech to text
  * Uses Deep Learning - Automatic Speech Recognition (ASR)
  * Use cases:
    * Transcribe customer calls
    * Automate close capitioning/subtitles
    * Generate metadata for media assets to create full scaleable architecture
  * Can remove PII using redaction
  * Supports automatic language identification for multi-lingual audio
  
### Amazon Lex: ASR to convert speech to text
  * Natural language understanding to recognize parts of speech/text
  * Helps to build chatbots, call center bots

### Amazon Connect: Receives calls, creates contact flows, cloud-based virtual contact center
  * Can integrate with other CRM systems or AWS
  * No upfront payments, 80% cheaper than traditional contact center solutions
  * Phone call => Connect =>Streaming data=>Lex=>trigger=>λ=>action

### Amazon Translate:
  * Natural and accurate language translation
  * Allows localization of content (eg applications/websites) for international users, and to easily translate large volumes of text efficiently 

### Amazon Kendra:
  * Intelligent enterprise document search service powered by ML that allows users to search across different content repositories with built in connectors.  Learns from user interaction/feedback for incremental learning.
  * Can return precise answer or pointers to document(s)
  * Allows discovery of information spanning all connected data allocated in AWS (given permission) and any 3rd party connected data (salesforce, servicenow, DBs, Microsoft One Drive, etc.)
  * Can use other services in AWS to preprocess content to text that is searchable/indexable

### Amazon Forecast:
  * Fully managed service using ML to deliver highly accurate forecasts (time series data)
  * 50% more accurate than looking at the data itself
  * Reduces forecasting time from months to hours
  * Use cases: Product Demand Planning, Financial Planning, Resource Planning
  * Data => S3 => Amazon Forecast => Model => Predictions

### Amazon Pinpoint:
  * Scalable 2-way (outbound/inbound) marketing communications service
  * Supports email, SMS, push, voice, in-app messaging
  * Ability to segment and personalize messages with the right content to customers
  * Possiblilty to receive replies
  * Scales to billions of messages per day
  * Use cases: run campaigns by sending marketing, bulk transactional SMS messages
  * Alternative to SNS or SES
  * In SNS and SES, you manage each messages audience, content and delivery schedule
  * In pinpoint, you create message templates, delivery schedules, highly targeted segments and full campaigns
  * Stream events (eg: text delivered) to SNS, Kinesis Firehose, Cloudwatch, etc.

### Amazon Rekognition:
  * Find objects, people, text, scenes in images and videos using ML
  * Facial analysis and search to perform user verification, people counting
  * Create a DB of "familiar faces" or compare against celebrities
  * Use cases: 
   * Labeling
   * Text detection
   * Face detection and analysis (gender, emotions, age range, etc.)
   * Face search and verification
   * Celebrity recognition
   * Pathing (eg: for sports game analysis)
   * Content Moderation (inappropriate, unwanted, or offensive images/videos)
    * Social media/broadcast media/advertising/e-commerce
    * Confidence level of content flags/gates (threshold configuration based)
    * Flag sensitive content for manual review in A2I
    * Help comply with regulations

## Acronyms

| Acronym  | Definition |
| ------------- | ------------- |
| ACL | Access Control List |
| ACM | AWS Certificate Manager |
| AD | Active Directory |
| ADFS | Active Directory Federation Services |
| ALB | Application Load Balancer | 
| AOF | Append Only File |
| API | Application Programming Interface |
| A2I | Amazon Augmented AI |
| ARN | Amazon Resource Name |
| ASG | Autoscaling group |
| ASR | Automatic Speech Recognition |
| AWS | Amazon Web Services |
| AZ | Availability Zones |
| CDC | Change Data Capture |
| CDN | Content Delivery Network |
| CMS | Content Management System |
| DAX | DynamoDB Accelerator |
| DB | Database |
| DNS | Domain Name System |
| EBS | Elastic Block Store |
| ECMP | Equal cost multi-path routing |
| ECR | Elastic Container Registry |
| ECS | Elastic Container Service |
| EFA | Elastic Fabric Adapter |
| EFS | Elastic File System |
| EKS | Elastic Kubernetes Service |
| ELB | (Classic) Elastic Load Balancer |
| EMR | Elastic Map Reduce |
| ENA | Elastic Network Adapter |
| ENI | Elastic Network Interface |
| ETL | Extract, Translate, Load |
| FTPS | File Transfer Protocol over SSL |
| GW | Gateway |
| GLB | Gateway Load Balancer |
| HA | High Availability |
| HC | Health Check |
| HSM | Hardware Security Module |
| IA | Infrequently Accessed |
| IAC | Infrastructure as Code |
| IAM |  Identity and Access Management |
| IdP | Identity Provider |
| IGW | Internet Gateway |
| LB | Load Balancer |
| LDAP | Lightweight Directory Access Protocol |
| KMS | Key Management Service |
| ML | Machine Learning |
| MPI | Message Parsing Interface |
| MSK | Managed Streaming Kafka |
| NAT | Network Address Translation |
| NLP | Natural Language Processing |
| OAC | Origin Access Control |
| OAI | Origin Access Identity |
| OLAP | Online Analytical Processing |
| OS | Operating System |
| OU | Organizational Unit |
| PHI | Protected Health Information |
| PII | Personally Identifiable Information |
| PPS | Packets Per Second |
| RAM | AWS Resource Access Manager |
| RDS | Relational Database |
| SAML | Security Assertion Markup Language |
| SASL | Simple Authentication and Security Layer |
| SES | Simple Email Service |
| SFTP | Secure File Transfer Protocol |
| SG | Security Group |
| SNI | Server Name Indication |
| SNS | Simple Notification Service |
| SQS | Simple Queue Service |
| SRT | Shield Response Team |
| SSL | Secure Sockets Layer |
| SSM | Systems Manager |
| SSML | Speech Synthesis Markup Language |
| STS | Security Token Service |
| SCP | Service Control Policies  |
| S3 | Simple Storage Service |
| TLS | Transport Layer Security |
| TTL | Time to live |
| VGW | Virtual Private Gateway |
| VM | Virtual Machine |
| VPC | Virtual Private Cloud |
| VPN | Virutal Private Network |
