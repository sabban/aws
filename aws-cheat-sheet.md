# IAM

# STS

# EC2

## EBS

Types
 * General Purpose SSD gp2
 * Provisioned IOPS SSD io1
 * Cold HDD sc1
 * Throughput Optimized HDD st1
 * Magnetic

Constraints
 * Scale up only
 * Volumes must be in the same AZ as the EC2 instances
 * Change types by taking snapshot and using the snapshot a new volume
 * After change on the fly, 6 hours wait before another change

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
  * MariaDB has support for Multi-AZ deployment and read replicas.
  * Oracle
  * Microsoft SQL Server
Two mechanisms for backing up.
  * automated backups (RDS feature which create a storage volume snapshots of DB instance 1 retention day by default up to 35)
  * manual snapshots
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
    CONCURRENT WRITES ?
* RedShift OLAP
* Elasticache (In memory caching)
  * Memcached
  * Redis

# SQS: Simple queue service
* 256ko of text data
* fifo not garanteed
* 12 hours visibility period by default
* a message can be delivered twice
* ChangeMessageVisibility to change the visibility timeout of a SQS message

# KMS
Muti-tenant

# AWS CloudHSM
dedicated required for FIPS 140-2 Compliance

# Cloudtrail
* Send to S3 bucket
* delivered every 5 (active) minutes  with up to 15 minutes delay
* notifications available
* can be aggregated accross regions
* can be aggregated accross accounts
Validation with sha-256 with public key against AWS' private key

# Cloudwatch

## Cloudwatch
* monitoring

## Cloudwatch Logs
Stored indefinitely (not user S3)

## Cloudwatch Events
Near real-time stream of system events
* Events:
 * AWS ressource state change
 * Aws CloudTrail (API calls)
 * custom events (code)
 * scheduled
*Rules
*Targets (lambda, SNS, SQS, Kinesis)

FAQ https://aws.amazon.com/cloudwatch/faqs

# AWS Config
* Enables
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
Compliance checks
 * Trigger Periodic
 * Configuration snapshot delivery (filterable)
Managed Rules
 * About 40
 * Basic but fundamental
Requires:
 * IAM role with RO permissions to the recorded ressources
 * Write access to S3 logging bucket
 * Publish access to SNS

FAQ https://aws.amazon.com/config/faqs