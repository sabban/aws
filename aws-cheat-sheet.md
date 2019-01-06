# IAM

authentification:
 * Username/passwords
 * MFA
 * Access Keys (Access Key Id/Secret Access Key ID)

4 types of roles:
1. AWS service Role (assume by service ec2, lambda, ...)
2. AWS Service-Linked Role (Amazon Lex Bots, Amazon Lex Channels)
3. Role for (cross) account access
4. Role for Identity Provider Access

 WebFederation: AssumeRoleWithWebIdentity

 AD/LDAP: AssumeRoleWithSAML

Other Feature
 * password policy in IAM
 * possibility to deactivate sts in certain regions
 * credential report
 * KMS

# STS
* assume-role
* assume-role-with-saml
* assume-role-with-web-identity
* get-session-token (MFA)

# EC2
ssh public key in metadata
when private ssh key is lost, snapshot and add public key.

In case compromised EC2
1. Shutdown the instance
2. snapshot to forensic
3. use it in an isolated VP
drmcgiftpx

## Autoscaling
* better fault tolerance
* better availability
* better cost manageability
standby possibility to debug
detach/attach possibility for instance to an ASG

# VPC
only one IGW at max per VPC
NACL
first rule matches
only way to block IP
Application load balancer span at least two subnet in two AZ with IGW

## EBS

Types
 * General Purpose SSD gp2
 maximazing throughput (160MiB/s) =max size of blocks (256ko)
 maximazing iops (10000) =min size of blocks (16ko)
 * Provisioned IOPS (32000iops/500MiB/s) SSD io1
 * Cold HDD sc1
 * Throughput Optimized HDD st1
 * Magnetic
burst=leakybucket with 5.4M credits at start in the bucket
in=3ips per Gib for gp2
max burst rate=3000
Constraints
 * Scale up only
 * Volumes must be in the same AZ as the EC2 instances
 * Change types by taking snapshot and using the snapshot a new volume
 * After change on the fly, 6 hours wait before another change
 * Volumes created from snapshots are Lazy restored from S3 -> forxce full read of the volume to force a restore

## EFS

 * grow and shrink automatically
 * support NFSv4
 * stored across multiple AZ's within a region
 * Read after write consistency

## CLI

* aws ec2 describe-isntances
* aws ec2 describe-images
* aws ec2 run-instances

## Metadata

http://169.254.169.254/latest/meta-data

## Load Balancer

* Classic
* Application Load Balancer

# SDK's

https://aws.amazon.com/tools
if default region it's us-east-1

# S3

## Properties

* object based
* unlimited storage
* 0 to 5Tb files
* files stored in buckets
* universal namespace https://s3-<region>.amazonaws.com/<bucket name>
* Read after Write consistency for PUTS of new Objects
* Eventual consistency for PUTS or DELETES on existing objects

## cross region replication
* cross region replication needs for versioning
* cross region replication works for *NEW* objects
* when update an objects ALL versions are replicated
* cannot replicate to multiple buckets or use daisy chaining 
* Delete markers are replicated, deleted versions of files are not.
* Delete individual versions or delete markers will not be replicated
* encrypted bucket are replicated

## Different tiers
* Standard (durable, immediatly available, frequently accessed)
* S3 IA (durable immediatly available, infrequently accessed)
* S3 RRS (Reduced redundancy storage

## Objects
* Key (name)
* Value (data)
* Version ID
* Metadata
* Subresousrces
 * ACL
 * Torrent

## Encryption

* SSE:
 * S3 managed Keys (SSE-S3)
 * AWS Key Management Service, (Managed Keys SSE-KMS)
 * SSE with customer provided keys (SSE-C)
* Client Side Encryption

## Versioning

* store all version of an object
* great backup tool
* If activated cannot remove versioning Can only suspend it
* Integrate with lifecycle rules
* MFA Delete capability

## Lifecycle management

* with or without versioning
READ FAQ

## multipart upload
* required from 5Go

# Content Delivery Network
* Edge Location: location where content is cached (different from AZ)
* Origin origin of all files distritubed by the CDN (S3 Bucket, EC2 instance, ELB, Route53)
* Distribution: collection of Edge Locations
* Edge location are not just read only
* objects are cache for the file of the TTL
* possibility of clear cache, but this is charged

# Storage Gateway (check for novelty!)

virtual appliance (VMware ESXi or MS hyperV) asynchronously replicate S3 (or Glacier)
 * File Gateway (NFS)
 * Volumes Gateway (iSCSI) (backed up as point-in-time snapshots of volumes and stored in the cloud as Amazon S3 incremental snapshots) (EBS or S3?)
   * Stored Volumes (1Gb-16Tb)
   * Cached Volumes (recent data stored locally only) (1Gb-32Tb)
 * Tape Gateway (VTL) (used for backup)

# Snowball
* Snowball (Import/Export from S3)
* Snowball edge (onboard storage and compute capabilities)
* Snowmobile (storage truck) (~100Pb)

# Transfer acceleration
 * use local edge location

# Databases

* RDS OLTP
  * MySQL Multi-AZ deployments for high availability and read replicas for horizontal scaling
  * PostgreSQL Multi-AZ deployment for high availability and read replicas for horizontal scaling.
  * Aurora Each instance is a multi-AZ cluster (up to 15 read replicas to increase performance or multi-AZ for high availabily)
    aurora has storage auto scaling
    snapshot saved in S3
    two types of replicas:
    * replicas with same underlying storage with low impact on master, enables failover without dataloss
	* readreplicas
  * MariaDB has support for Multi-AZ deployment and read replicas.
  * Oracle
  * Microsoft SQL Server
Two mechanisms for backing up.
  * automated backups (RDS feature which create a storage volume snapshots of DB instance 1 retention day by default up to 35)
  * manual snapshots
Limitations:
  * mssql not able to scale storage
* DynamoDB No SQL
  * fully managed database
  * both document and key-value data
  * stored on SSD storage
  * spread across geographically distinct data centres
  * two models:
    * Eventual Consistent Reads (Default and Best read Performance)
    * Strongly consistent reads
  * Consists of
    * tables
    * Items (rows)
    * Attributes (columns)
  * nested (until 35 nested)
  * Charged by
    * provisioned thoughput Capacity
    * charged per Gb
  * Two types of primary keys
    * partition key (hash key) (single attribute primary key)
    * sort key (range key) (composite primary keys)
  * Index
    * Local secondary index: (same partition, different sort key, only when table created)
    * Global secondary index: (different partition, different sort key, at creation time or later)
  * scan vs query
    * query: With a partition key attribute name and distinct value to search for
    * scan: Scan operation returns all the data attributes for every items with the possibility of use the ProjectExpression
  * provisioned throughput
    * read 4k + number of reads/s (/2 if eventually consistent)
    * write 1k + number of write/s
    * error 400 ProvisionedThroughputExceededException (for a table or for one or more secondary indexes)
  * BatchWriteItem
    * puts or delete mutliple items in one or more tables
    * checks for unprocessed items until all items have been processed
  * DynamoDB Streams
    * KEYS_ONLY
    * NEW_IMAGE entire item after the update
    * OLD_IMAGE entire item before the update (all can be rebuilt with the actual value)
    * NEW_AND_OLD_IMAGES
    CONCURRENT WRITES ?
* RedShift OLAP
* Elasticache (In memory caching)
  * Memcached
  * Redis

# SQS: Simple queue service
* 256ko of text data (can use S3 for larger message)
* fifo not garanteed
* 12 hours visibility period by default
* a message can be delivered twice
* ChangeMessageVisibility to change the visibility timeout of a SQS message
* possibility of fanout with multiple SQS queues subscribed to a SNS topic
* up to 14 days retention

# Amazon Kinesis
* Kinesis Streams
  * collect and process large streams of data in real time
    Scenario : real time data analytics
  * 24 hour retention
* Kinesis Analytics
SQL request on data stream

* Kinesis Firehose
load data to S3 or Redshift
support compression and encryption (SSE, KMS)


# KMS
Muti-tenant
KMS region localized
EBS, S3, Redshift, elastic transcoder, WorkMail, RDS, ...
Two types of key
* customer provided
* AWS provided
Two types of permissions
* key administrative permissions
* key usage permissions 

# AWS CloudHSM
dedicated required for FIPS 140-2 Compliance

# Cloudtrail
* Send to S3 bucket or optionally cloudwatch
* delivered every 5 (active) minutes  with up to 15 minutes delay
* notifications available
* can be aggregated accross regions
* can be aggregated accross accounts
Validation with sha-256 with public key against AWS' private key

When logs are sent to cloudwatch a 256kb limit is applied, logs are
not sent if the evfent is oversized.

# Cloudwatch

## Cloudwatch
* monitoring

## Cloudwatch Logs
Can be stored indefinitely (not user S3)

## Cloudwatch Events
Near real-time stream of system events
* Events:
 * AWS ressource state change
 * Aws CloudTrail (API calls)
 * custom events (code)
 * scheduled
* Rules
* Targets (lambda, SNS, SQS, Kinesis)

## Cloudwatch Alarms
sent to SNS or autoscaling
states:
OK
ALARM
INSUFFICIENT_DATA
alarm frequency has to equal or higher than metrics
alarm accross dimension (autoscaling group for example)

FAQ https://aws.amazon.com/cloudwatch/faqs

## Cloudwatch agent


Required:
* Two roles
 * A role for configuration with policies
  * CloudWatchAgentAdminPolicy
  * AmazonEC2RoleforSSM
 * A role for production with policies
  * CloudWatchAgentServerPolicy
  * AmazonEC2RoleforSSM
* Internet acces required to communicate with SSM and cloudwatch endpoints
* (Optionnally) SSM agent installed, configuration file stored in SSM parameter store

# AWS Config
once per region and region specific
* Enables:
  * Compliance auditing
  * Security analysis
  * Rsource tracking
* Provides
  * Configurations snapshots and logs config changes of AWS resources
  * Automated compliance checking
Components
* Dashboard
* Rules
  * Managed
  * Custom
* Resources
* Settings
Compliance checks:
  * Trigger Periodic
  * Configuration snapshot delivery (filterable)
Managed Rules
  * About 40
  * Basic but fundamental
Requires:
  * IAM role with RO permissions to the recorded ressources
  * Write access to S3 logging bucket (updated each 6 hours)
  * Publish access to SNS
Configuration Item
 * Metadata (version, configuration id, md5 timestamp, state id)
 * Attributes
 * Relationships
 * Current configuration (describe or list)
 * Related events (cloudtrail event id)

FAQ https://aws.amazon.com/config/faqs

# AWS Inspector
* Agent installed on ec2
* Need for a role (Read only access to EC2 instances)
* Assessment targets : set of EC2 instances to be assess against (group by tags)
* AWS agents
* Assessment templates (Define a specific configuration as to how an assessment is run)
* Rules Pacakges & Rules
 * CVE
 * CIS operating System Security Configuration Benchmarks
 * Security Best Practice
 * Runtime Behavior Analysis
* Telemetry data collected from an instance
* Assessment reports
* Findings: potential security issue or risk (comes with explanation and guidance to solve and a level  high, medium, low, informational)
-> Use AWS lambda to schedule assessment run.

Limits:
* max 500 agents per assessment (hard limit)
* max 50k assessments run per accounts
* max 500 assessments templates
* max 50 assessments target

# AWS Trusted Advisor
* Cost optimiszation
* availability
* performance
* Security

# AWS Key Management Service
Multi-tenant hardware (not as CloudHSM)
region localized

# AWS Shield
 * Free service for protecting all AWS customers on ELB, cloudfront and Route 53
 * Advanced provides enhanced protections 3000$ per month
    * provides DDOS response team
    read the afferent whitepaper

# AWS WAF
Application Load Balancer or cloudfront

# AWS Certificate Manager
* renew automatically if domain purchased in route53 if not for a route53 hosted zone
* Load balancers and cloudfront
* Cannot export certificate

# API Gateway
Throttling at 10000 requests per seconds, or 5000 at burst
caching base on ttl
default 300s, max at 3600s, min at 0s (disabled)

# GuardDuty
Analyze
* cloud trail event logs
* VPC flow logs
* DNS logs
* optionally supports cloudwatch event log
Machine learning
* 1 whitelist possible
* 6 threatlist possible

possible to link multiple AWS account: members account send copy of
finding to master account but master account can't disable members'
account guardduty

Use specific IAM permissions to create a service-linked role that
allow GuardDuty to retrieve needed information.


# OpsWorks
managed by layers
lifecycle events
* setup: event occurs when an instance has finished booting
* configure: enters or leaves online associate or disassociate EIP, attach or detach ELB to a layer on ALL instances
* deploy: occurs when you run the deploy command on an instance
* undeploy: occurs when you  delete application or run undeploy
* shutdown runs when an instance is shutdown but before instance is terminated. Allow cleanup
 Instances:
* 24/7 instances
* time based instances
* load-based instances
five applications version (rollback to try to return to the precedent)
from chef 11.10 chef allow external sources of recipes (berkshelf)
Databags:
global json objects accessible within the chef framework include STACK, LAYER, APP, INSTANCE
Properties:
* use of an agent on each instance
* heartbeat style health check with "auto-heal"
auto-heal in case connection is lost:
EBS-backed:stop-start
instance store: terminate-launch
-> configure event

# Elastic Beanstalk
deploy, monitor, scale
applications can have multiple environments
swappable url
sqs is used to message between different beanstalk environment
RDS is not copied when cloning EB environment
node;js
php
python
ruby
IIS
java
golang
tomcat
docker container

ebextensions=customizations
yaml
possibilty to choose a leader instance for environment creation (to make commmand run once)
call to cloudformation !!

EB with docker
-> need Dockerfile
-> can use a docker registry

# AWS CloudHSM

dedicated appliance
deployed in one vpc
scalability, availability, performance are relying on customer's use
can be multi-region with the right setup
api call are logged in cloudtrail
internal log are sent to a syslog IP endpoint
need an ec2 instance with python2.7, pip, and aws aws-cloudhsm-cli

# AWS CLI cheatsheet
* Autoscaling
  * enter-standby
  * exit-standby
  * create-launch-configuration
  * delete-launch-configuration
  * update-auto-scaling-group
  * put-lifecycle-hook
  * put-scaling-policy
* Cloud watch
  * put-metric-data
  * put-metric-alarm
  * disable-alarm-actions
  * enable-alarm-actions
  * set-alarm-state
  * list-metrics
  * get-metric-statistic
* Dynamodb
  * get-item, batch-get-item, query, scan
  * put-item, update-item, delete-item, batch-write-item
  * create-table
  * update-table
* EC2
  * run-instances
  * stop-instances
  * start-instances
  * terminate-instances
  * describe-instances
  * wait
  * create-image
  * create-snapshot*
  * copy-image
  * copy-snapshot*
  * create-volume*
  * describe-tags*
* S3
  * mb, rb
  * mv, rm
  * sync
  * website
* S3API
  * head-object
  * head-bucket
  * get/put bucket-versioning
  * get/put bucket-acl
  * put-bucket-notification-cofniguration (SNS, lambda)
* SQS
  * add-permission
  * change-mesage-visability
  * set-queue-attributes
  * send-, receive, delete-message
  
 Direct connect (vpn)
 KMS (rotation)
 SSM
 system manager patch manager
 SSO from AD with or without cognito
 Config 

1ok
2ok
3ok
4ok
5ok
6ok
7ok
8ko
9ko
10?
11ok
12ok
13ok
14ko
15ko
16ok
17ok
18ok
19ko
20ok
