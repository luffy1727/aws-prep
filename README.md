# Tushig`s journey on becoming AWS solutions architect .

I will be conducting 2 hours(min) study sessions until the day of my test, everyday.

##### The session will consist of:
1. Reviewing past sessions
2. Watching AWS lessons from udemy (Thank you @lbayarkhuu for the shared account)
3. 1 practice exam on the studied subjects (https://digitalcloud.training/)

## Day 1 notes:

##### Elastic IP
- You can only have 5 Elastic IP in your account
- Using Elastic IP as a LB is highly discouraged.
##### EC2 Spot instance
- Discount of up to 90% compared to On-demand
- The hourly spot price varies based on offer and capacity
- 2 minutes grace period to choose to stop or terminate your instance.
- **Spot Block** - blocks the spot instance for a time frame(1 to 6 hours) without interruptions
- Spot instances are good for:
        - batch jobs, data analysis, workloads that are resilient to failures
- Spot instances are not good for:
        - critical jobs, databases  
- Disabling Spot Request does not stop the instances. Therefore, in order to cancel the spot instances, one most cancel the spot request first and disable the instances by afterwards.
- **Spot Fleets** = set of Spot instances + (optional) On-Demand Instances
        - It will launch instances from the launch pool(instance type, OS, AZ) until it hits the capacity or the max cost.
        - Strategies to allocate spot instances:
    1. lowestPrice(cost, great for short workload)
    2. diversified(great for availability,long)
    3. capacityOptimized 
##### EC2 Placement Groups
- **Cluster** - low latency(10Gbps), in a same AZ, on a same hardware. Good for Big Data job that needs to complete fast.
- **Spread** - Can span across multiple AZs, limited to **7 instances per AZ**, 'Reduced risk' of every instance failing
![image](images/placement_spread.png)

    Good for critical applications where each instance must be iso from each other
- **Partition** - Up to 7 partitions per AZ, can span accross multilple AZs in the same region, up to 100s of EC2 instances, each partition iso from each other
    - use cases: HDFS, Hbase, Cassandra, and Apache kafka 
![image](images/placement_partition.png)
## Day 2 notes:
### EC2
##### Elastic Network Interfaces (ENI)
- Logical component in a VPC that represents a virtual network card. (Basically what gives an instance an access to network.)
- ENI HAS:
    - Primary private IPv4, one or more secondary IPv4
    - One Elastic IP (IPv4) per private IPv4
    - One Public IPv4
    - One or more security gorups
    - A MAC address
- Bound to single AZ
- Can reattach to another EC2
##### EC2 Hibernate
- In-memory (RAM) is preserved (boot time is much faster)
Under the hood:
        The RAM is written to a file in the root EBS volume

        Instance RAM size < 150GB
        Available for On-Demand and Reserved instances
        EBS volume must be encrypted and can hold 150 gb obviously
        Instance cannot be hibernated more than 60 days
##### EC2 Nitro
- Underlying platform for next generation EC2 instances
- Allows for better performance:
        -Better networking options (enhanced networking, HPC, IPv6)
        - **Higher Speed EBS (64000 EBS IOPS, MAX 32000 on Non-Nitro)**
- Better underlying security
- Instance types example:
        - A1, Anything after C5
        - all the above with metal in the name: a1.metal, c5 metal etc
##### EC2 vCPU

    vCPU = number of threads on a CPU 
##### EC2 Capacity Reservations

    You can basically reserve EC2 instances on AWS for specific timeframe with specific instance types.
- will get billed as soon as it starts
##### Elastic block store (EBS)
- It is basically cloud USB stick
- It allows your instances to persist data, even after their termination
- Can only be mounted to one instance at a time
- Locked to single AZ (if in us-east-1a, cannot be used in us-east-1b)
- Can be copied across AZs using snapshots.
- Free tier: 30GB of free EBS storage of type General Purpose (SSD) or Magnetic per month
- Have a provisioned capacity(size in GBs and IOPS)
        - will get billed for all the provisioned capcity
        - can increase capcity over time
![image](images/EBS.png)
- By default, when instance is terminated, root EBS will be deleted. It can be changed
- By default, when instance is terminated, EBS volumes other than root will not be deleted
##### EBS Volume Types
- 6 types:
    - gp2/gp3(SSD): General purpose SSD volume (1gb - 16tb)
        - **gp2**: 
            -Small gp2 volumes can burst IOPS to 3000
            -size of the volume and IOPS are linked, MAX IOPS is 16000
            - 3 IOPS per GB, means at 5334 GB we are at max IOPS
        - **gp3**: 
            - can burst from 3000 to 16000 IOPS and throughput from 125 MB/s to 1000 MB/s  
    - io1/io2(ssd):  Provision IOPS SSD volume (4 GB - 16 TB) for low-latency high-throughput workloads
        - Critical business applications with sustained IOPS
        - Applications that needs more than 16000 IOPS
        - Great for DB workloads
        - Max PIOPS: 64000 for Nitro EC2 & 32000 for others
        - Can increase PIOPS independently from storage size
        - io2 have more durability and more per GB at the same price as io1
        - Can use io2 Block express (4gb - 64tb):
            - sub millisecond latency
            - Max PIOPS: 256000 with an IOPS:GB ratio of 1000:1
        - Supports EBS multi-attach
    - st1 (HDD): Throughput optimized HDD volume for freq accessed, throughput intensive workloads (125mb to 16tb)
        - Big data, data warehouse, log processing
        - Max throughput: 500MB/s - max IOPS 500
    - sc1 (HDD): Cold HDD volume for less freq accessed workloads (125mb to 16tb)
        - Data that is very rarely accessed (e.g:your mother`s phone number)
        - Lowest cost
        - Max throughput: 250MB/s - max IOPS 250
- ONLY gp2/gp3 or io1/io2 can be used as root volume

##### EBS Multi Attach - io1/io2 family
- Attach the same EBS to a multiple EC2 instance in the same AZ
- Each instance has full read & write permissions to the volume
- Use case:
    - Achieve higher availability in clustered Linux applications
    - Applications must manage concurrent write operations
- Must use file system that is 'cluster-aware'

##### EBS Snapshots
- Make a backup (snapshot) of EBS at any point in time
- Recommended to detach from instance to do snapshot
- Can copy snapshots across AZs or even Regions

##### EBS Encryption
- When encrypted EBS volume is created everything will be encrypted:
    - data at rest
    - data in flight between instances
    - all snapshots
    - all volumes that are created from those snapshots
- Encryption and decryption is handled behind the scenes
- Encryption has a minimal impact on latency
- Encryption from KMS (AES-256)
###### HOW TO ENCRYPT UNENCRYPTED VOLUME:
- create EBS snapshot
- encrypt the snapshot (using copy -> ? might have to look into it)
- create new EBS volume from the snapshot
- attach the volume to the original instance

##### Amazon Machine Images (AMI)
- Customization for EC2 instances
    - Add your own software, config, OS, monitoring etc
    - Faster boot since all your software is pre packaged
- Built for specific regions (can be copied across regions)
- Can launch instance from:
    - Public AMI (AWS provided)
    - Own AMI
    - AWS marketplace AMI (kinda like github i guess)
##### EC2 Instance Store
        - Basically a physical hard drive attached to the physical server
- Better I/O performance since it is attached physically
- If the attached EC2 instance is terminated the storage will be lost
- Good for: buffer / cache/ scratch data / temp content
## Day 3 notes:
##### EBS RAID Options
RAID is possible as long as your OS supports it.
- RAID 0: to increase performance/ Risk more
    ```
    It basically adds 2 EBS volumes (size, IOPS and everything) to increase performance with a increased risk of failure, because if one of the disk fails everything fails.
    ```
![image](images/RAID0.png)
- RAID 1: to increase fault tolerance
    ```
    It basically mirrors one EBS to another EBS by sending the data to both of them at the same time just like your ex-gf. 
    ```
![image](images/RAID1.png)
###### DO NOT NEED TO KNOW THESE
- RAID 5
- RAID 6
- RAID 10

#### EFS - Elastic File System
```
Managed NFS(Network file system) that can be mounted on many EC2 across different AZ
```
- **Available across Multilple AZs**
- Highliy available, scalable, expensive (3xgp2), pay per use
- Uses NFSv4.1 protocol
- **Only compatible with Linux based AMI**
- Uses Security group to control access to EFS
- Encryption at rest using KMS
```
Generally, EFS is very high performant and the scaling is done automatically
```
- Performance mode (set when creating the EFS)
    - General purpose(default): webserver, cms etc
    - Max I/O (higher latency, throughput) : big data, media processing etc
- Throughput mode
    - Bursting (default): 1TB = 50MB/s + burst of up to 100MB/s
    - Provisioned: set your throughput regardless of size, ex: 1GB for 1TB storage
- Storage tiers
    - Standard: for frequently accessed files
    - Infrequent access(EFS-IA): cost to retrieve files, lower price to store

### LB - Load Balancer
#### Why use Load Balancer?
- Spread load across multiple instances
- Expose a single point of access
- Handle failures
- Regular health check
- SSL termination (HTTPS) for your websites
- enforce stickiness with cookies
- High availability across zones
- Separate private and public traffic
- AWS takes care of upgrades, maintenance, high availability
#### 3 Types of load balancer on AWS:
- Classic LB (v1-old gen) - 2009
    - Layer 4 - TCP
    - Layer 7 - HTTP, HTTPS
    - Health checks are tcp or http based
    - fixed hostname: XXX.region.elb.amazonaws.com
    - Support only one SSL certificate
- Application LB (v2-new gen) - 2016
    - Layer 7 - HTTP, HTTPS, WebSocket
    - Can load balance to multiple applications (e.g. containers)
    - Can redirect from http to https
    - Route based on path in URL
    - Route base on hostname in URL(some.example.com & other.example.com)
    - Route based on Query string, Headers
    - Supports multiple listeners with multiple SSL certificates
    ```
    Good for microservices or other container based applications because it has port mapping feature to redirect to a dynamic port in ECS
    ```
    - Target groups:
        - EC2 instances
        - ECS tasks
        - Lambda functions
        - IP addressses

        **ALB can coute to multiple target groups**
    - fixed hostname: XXX.region.elb.amazonaws.com
    - x-forwarded-proto = header for forwarded client ip
- Network LB (v2-new gen) - 2017
    - Layer 4 - TCP, TLS(Secure TCP) & UDP
    - Handle millions of request per secons, Less latency
    - One static IP per AZ
    - Not included in the AWS free tier
    - Supports multiple listeners with multiple SSL certificates     
#### Good to know
- Can setup both private and public LB
- LBs can scale but not instantaneously
- ELB access logs will log all access requests
- Cloudwatch Metris will give you aggregate statistics
- LB Errors 503 means at capacity or no registered targets
- Can use cookie to implement stickiniess so that the same client is always redirected to the same instance (only CLB or ALB)
#### Cross-zone Load Balancing
``
If this option is enabled, all of the incoming traffic will be distributed evenly across the instances regardless of which AZs they are in.
``
- ALB
    - always on can`t be disabled
    - no charges for inter AZ data
- NLB
    - disabled by default
    - you pay charges for inter AZ data
- CLB
    - if created through console => enabled by default
    - if created through CLI/API => disabled by default
    - No charges for inter AZ data
## Day 4 notes:
##### SSL certificates
- ```The load balancer uses X.509 certificate```
- ```Can manage certificates using ACM (AWS certificate manager)```
- Can create/upload your own certificates
- https listener:
    - must specify default certificate
    - can add an optional certs
    - **Clients can use SNI (Server name idication) to specift the hostname they reach**
        - SNI solves the problem of loading multiple SSL certificates onto one webserver
        - only supports ALB, NLB and CloudFront

 ##### Connection Draining
 ``` "Connection Draining" for CLB or "Deregistration delay" for ALB & NLB```
- Time to complete "in-flight" requests while the instance is unhealthy or deregistering.```
- Between 1 to 3600 seconds, default is 300
- Can be desabled
#### Auto Scaling Group
**Attributes:**
- A launch config
    - AMI + instance type
    - EC2 User Data
    - EBS volumes
    - Security groups
    - SSH key pair
- Min/ Max size/ Init capacity
- network + subnets information
- LB information
- Scaling policies-> what will trigger scale in and out
![image](images/ASG.png)

## Day 5 notes:
##### Scaling policies:
- Target Tracking Scaling
    -  Most simple to set up
    - E.g: I want the average ASG CPU to stay at around 40%
- Simple/ Step Scaling
    - if cpu > 70%  + 2 instance elif cpu < 30%  - 1 instance
- Scheduled Actions
    - I want +2 instance on Mondays at 2pm

##### Scaling Cooldowns:
- CD between scaling actions. (kind of obvious if u think about it)
- In addition to default CD, can create specific cd for **simple scaling policy**
- `specific cds do not add up, instead they override the default CD`
- Can/Should override the CD for scale-in(terminate instances) policies
##### Good to know:
- ASG Default termination policy:
1. Find the AZ which has the most number of instances
2. If there are multiple instances in the AZ to choose from, delete the one with oldest config.
`ASG tries to balance the number of instances across AZ by default`
- Lifecycle hooks:
`Basically an ability to send the instance to a wait state to perform actions before it goes in service or terminated`
![image](images/lifecycle_hooks.png)
- Launch Template vs Launch Config
`Both can choose the specific params such as ID of the AMI, type, key, user data etc but`**always go with the launch template** 
    
    - Launch template
    1. newer
    2. AWS recommended
    3. Can have multiple version
    4. Can create params for re-use and inheritance
    5. Can provision using both On-demand and spot instances
    6. Can use t2 unlimited burst feature
    - Launch config
    1. old - legacy
    2. Must be re-created every time you change something

## Day 6 notes:
#### AWS RDS:
- Postgres
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Aurora
###### Why RDS over deploying DB on EC2?
- Automated provisioning, OS patching
- Continuous backups and restore to specific timestamp
- Monitoring dashboard
- Read replicas for improved read peformance
- Multi AZ setup for Disaster Recoivery
- Maintenance windows for upgrades
- Scaling capability (vertical and horizontal)
- Storage backed by EBS (gp2 or io1)
- **BUT you can`t ssh into your instances** 

`RDS - Backups`
Automated Backups:
- Daily backup (during maintenance window)
- Transaction logs are backed up by RDS every 5 minutes
- Ability to restore to any point in time
- 7 days retention (can be increased to 35)
- DB Snapshots are manually triggered by the user

`RDS - Storage Auto Scaling`
Have to set Maximum storage threshold.
Automatically modifiy storage if:
- Free storage < 10% allocated storage
- Low storage lasts at least 5 minutes
- 6 hours have passed since last modification

`RDS - Read Replicas`
- up to 5 read replicas
- has 3 scope options 
    - Within AZ (free)
    - Cross AZ (free)
    - Cross Regions ($$$)
- Replication is async
- Replicas can be promoted to their own DB (oh geez im a real DB now)
- Applications must update the connection string to leverage read replicas. (I actually don`t understand this sentence)

`RDS - Multi AZ(Disaster Recovery)`
- Sync Replication
- One DNS name
- Increase availability
- Failover in case of loss of AZZ, los of network, instance or storage failure
- Not used for scaling
##### Good to know:
- The READ replicas can be setup as the standy DB for multi AZ
- From single AZ to Multi Az operation has zero downtime

`RDS Security - Encryption`
- At rest encryption
    - Possibility to encrypt the master & read replicas with AWS KMS
    - Encryption is defined at launch time
    - **If the master is not encrypted, the read replicas cannot be encryped**
    - Transparent Data Encryption (TDE) available for Oracle and SQL server
- In-flight encryption
    - Obviously SSL certficate to encrypt data to RDS in flight
    - Provide SSL options wtih trust certificate when connecting to db
    - To **enforce** SSL:
        - PostgreSQL: rds.force_ssl =1 in the AWS RDS console
        - MySQl: Within the DB:
        ```GRANT USAGE ON *.* 'mysqluser'@%' REQUIRE SSL;```

`RDS Security - Network * IAM`
- Network Security
    - RDS databases are usually deployed within a private subnet, not in a public one
    - RDS security works by leveraging security groups
- Access Management
    - IAM policies help control who can manage AWS RDS (who can delete, create read replica etc)
    - Uses normal username & password to login to the database. 
    - However, IAM-based authentication can be used to login into RDS with MySQL & PostgreSQL
 #### AWS Aurora
 Faster, better MySQL
 - failover is basically instant
 - storage grows automatically
 - can have 15 replicas
 - replication is much faster
 - cloud native
 - cost is 20% more
 ![image](images/aurora.png)
 - Has writer DNS
 - Has reader DNS 
 - Can create custom endpoint for the read replicas. However, when custom endpoint is created, the default endpoint gets removed.

#### AWS Redshift
 - Based on PostgreSQL, not used for OLTP (online transaction processing)
 - is OLAP (analytics)
 - 10x better performance than other data warehouses,

## Day 7 notes:
#### AWS ElastiCache
`AWS managed Redis or Memcached`
- Redis
    - Multi AZ with auto failover
    - Read replicas to scale reads
    - High availability
    - Data durability using AOF persistence
    - Backup and restore features
    **this is basically memory RDS**
- Memcached
    - Multi-node for partitioning of data(sharding)
    - No high availablity
    - non persistent
    - no backup and restore
    - multi-threaded architecture

`Elasticache Security`
- All cache in Elasticache
    - do not support IAM authentication
    -
- Redis AUTH
    - you can set password/token when you create redis
    - support SSL in flight encryption
- Memcached
    - Supports SASL-based authentication
`Pattersn for ElastiCache`
- Lazy loading: all the read data is cached, data can become stale in cache
- Write through: Adds or update data in the cache when written to a DB
- Session store: stor temp session data in cache

`Redis Sorted Sets`
- Gurantee bothh uniqueness and element ordering
#### AWS Route 53
`is basically a AWS managed DNS`
In AWS, the most common records are:
- A: hostname to IPv4
- AAAA: hostname to IPv6
- CNAME: hostname to hostname
    - app.mydomain.com => test.anything.com
    - DOES NOT WORK FOR ROOT DOMAIN
    (has to be something like app.tushig.com not tushig.com)
- Alias: hostname to AWS resource
    - app.mydomain.com => test.amazonaws.com
    - WORKS FOR ROOT DOMAIN
    - free of charge
    - Native health checks

Route 53 can use 
- public domain names you own
- private domain names that can be resolved only your private VPCs

Route53 is a global service

You pay $0.50 per month per hosted zone

## Day 8 notes:

#### AWS S3:
- AWS buckets global unique name
- AWS buckets region based service
- You don`t actually create directories within buckets but instead you create objects with very long "keys"
- The **key** is the FULL path
    - s3://my-bucket/*my_folder/somefolder/photos.gif*
    - "*my_folder/somefolder/photos.gif*" is the key

##### S3 Object:
- Object values are the content of the body
    - Max object size is 5TB
    - if uploading more than 5gb, use `multi-part upload`
- Metadata
- Tags
- Version ID
    - enabled at the bucket level
    - same key will just increase the version
    - Any file before versioning will have *null* as a version
    - When deleting versioning option it does not delete the previous versions

##### S3 Encryption:
###### 4 methods of encrypting:
`SSE short for server side encryption`
- SSE-S3:
    - AWS handles the keys
    - AES-256
    - must set header: "x-amz-server-side-encryption": "AES256"
- SSE-KMS:
    - AWS Key management service to manage keys
    - gives control on who has what keys + audit trail
    - must set header:
    "x-amz-server-side-encryption": "aws:kms"
- SSE-C:
    - When you want to manage keys
    - S3 does not store the encryption key you provide
    - HTTPS **must** be used
    - Encryption key must be provided in every http headers.
    - Must use the data key when retrieving
- Client side encryption: 
    - Client library such as the AWS s3 encryption cleint
    - basically encrypts the file and sends it to s3. (same logic applies to retrieving)
##### S3 Security:
- User based
    - IAM policies - who can access this API etc
- Resource based
    - Bucket policies - bucket wide rules
    - Object access control list
    - Bucket access control list
##### S3 MFA-Delete
- Can only be done using CLI
- Only the bucket owner(root account) can enable/disable MFA-Delete
##### S3 Access logs
- AWS Athena
- **DO NOT set your logging bucket to be the monitored bucket** (it will create a logging loop)
##### S3 Replication (CRR & SRR)
- Must enable versioning in source and destination
- Does not copy the existing files
- Any DELETE operation is not replicated
- There is no "chaining" effect of replication
- Cross region replication
    - Use cases: compliance, lower latency access, preolication across accounts

- Same region replication
    - log aggregation, live replication between prod and test accounts
- Buckets can be in different accounts
- Copying is async
- must give proper IAM permissions to S3
##### S3 Storage Classes
- Amazon S3 Standard - General purpose
    - High durability (99.999999999%) of objects across multiple AZ
    - Sustain 2 concurrent facility failures
- Amazon S3 Standard - Infrequent Access (IA)
    - lower cost than s3 standard
    - Use cases: Data store for disaster recovery, backup
- Amazon S3 One Zone - Infrequent Access
    - 99.5% availability
    - Low cost compared to IA (by 20%)
    - Use Cases: Storing secondary backup copies of on-premise data, or data you can re-create(thumbnails etc)
- Amazon S3 Intelligent Tiering
    - Small monthly monitoring and auto-tiering fee
    - Automatically moves objects between `General purpose` and `IA`
- Amazon Glacier
    - Cold archive/backup
    - Data is retained for longer term (10s of years)
    - Alternative to on-premise magnetic tape storage
    - cost per storage per month ($0.004/ GB) + retrievel fee
    - each item in Glacier is called "Archive" (up to 40TB)
    - Archives are stored in "Vaults"
    - 3 retrieval options:
        - Expedited (1 to 5 minutes)
        - Standard (3 to 5 hours)
        - Bulk (5 to 12 hours)
    - Minimum storage duration of 90 days
- Amazon Glacier Deep Archive
    - 3 retrieval options:
        - Standard (12 hours)
        - Bulk (48 hours)
    - Min storage duration 180 days.
    - cheaper

![image](images/s3_storage.png)

## Day 8 notes:

##### S3 Performance
- Can get 3500 PUT/COPY/POST/DELETE and 5500 GET/HEAD requests per second per prefix in a bucket
    - example:
    - bucket/folder1/sub1/file => prefix: /folder1/sub1
    - bucket/folder1/sub2/file => prefix: /folder1/sub2
    Will get the performance for each prefix!
- Can be limited by KMS limitations
- Multi-part Upload:
    - recommended for files > 100MB
    - required for files > 5GB
    - parallel upload by divide and conquer
- Transfer acceleration
    - increase transfer speed by transferring the file to edge location first which will forward the data to the s3 location
    ![image](images/accelerator.png)
- S3 Byte-Range Fetches
    - Divide and conquer for fetch/get requests

##### S3 Requester Pays
- Basically the person who is requesting the object will pay for the networking cost for retreiving the object.
- The requester must be authenticated in AWS (pretty obvious)
- Helpful when sharing large items with other accounts
##### AWS Athena
- Serverless
- Analytics against S3
- Uses SQL language
- has JDBC / ODBC driver
- Charged per query and amount of data scanned
- Supports CSV, JSON, ORC, Avro and Parquet
`Whenever you want to analyze something on S3 (logs): Use Athena`

### AWS Lambda:
- Easy pricing
    - Pay per request and compute time
    - Free tier of 1,000,000 AWS Lambda requests and 400,000 GBs of compute time
- Easy monitoring with AWS CloudWatch
- Increasing RAM will also improve CPU and network
##### AWS Lambda Limits:
- Execution
    - Memory Allocation: 128MB - 10GB(64MB increments)
    - Maximum execution time: 900 seconds(15 minutes)
    - Environment variables(4KB)
    - Disk capacity in the "function container" (in /tmp): 512MB
    - Concurrent executions: 1000 (can be increased)
- Deployment:
    - Lambda function deployment compressed size: 50MB
    - Uncompressed: 250MB
    - Can use /tmp to load files at startup
    - size of env variables: 4KB

#### DynamoDB
- Fully managed, Highly available with replication across 3AZ
- NoSQL
- Scales to massive workloads, dist database
- Millions of req per seconds, trillions of row, 100s of TB of storage
- Fast and consistent in performance
- Integrated with IAM for security, authorization and administration
- Enables event driven programming with DynamoDB streams
- Low cost and auto scaling capabilities

##### DynamoDB - Dax
- Cache for DynamoDB
- Solves hot key problem
- 5 minutes TTL by default
- Up to 10 nodes in the cluster
- Multi AZ
- Secure

##### DynamoDB - Streams
- Changes in DynamoDB (Create, Update, Delete) can end up in a DynamoDB Stream
- This stream can be read by AWS lambda
    - React to changes in real time (welcome email etc)
    - analytics
    - elasticserach
- Could implement Cross Region Replication using Streams
- Stream has 24 hours of data retention

##### DynamoDB - New Features
- Transactions
    - All or nothing type of operations
    like SQL flush type stuff
- On demand
    - No capacity planning needed
    - 2.5x more expensive than provisioned capacity
- Global Tables(Cross Region Replication)
    - Active replication
    - Must enable streams
    - Usefol for low latency, DR purposes
- Capacity planning
    - Planned capacity: provision WCU & RCU, can enable auto scaling
    - On-demand capacity: get unlimited WCU & RCU, no throttle, more expensive

#### AWS API Gateway
- AWS lambda + API gateway: no infra to manage
- Support for the websocket protocol
- Handles API versioning
- Handles different environments
- Handles security
- Create API keys, handle request throttling
- Swagger / Open API import to quickly define APIs
- Transform and validate requests and responses
- Generate SDK and API specifications
- Cache API responses
- API owners can set a rate limit of 1,000 requests per second for a specific method in their REST APIs (returns 429 HTTP response when the limit is exceeded) 


## Day 9 notes:

#### AWS CloudFront:
- is CDN
- Improves read performance, content is cached at the edge
- 216 Point of Presence globally
- DDoS protection, integration with Shield, AWS web application Firewall
- Can expose external HTTPS and can talk to internal HTTPS backends

##### Cloudfront Pricing
- The cost of data out per edge location varies(More data less cost)
- You can reduce the number of edge locations for cost reduction
- 3 price classes:
    1. Class All: all regions - best performance
    2. Class 200: most regions, but excludes the most expensive regions
    3. Class 100: only the last expensive regions
##### CloudFront vs S3 Cross Region Replication

- CloudFront:
    - Global Edge network
    - Files are cached for a TTL
    - Great for **static** content that must be available **everywhere**
- S3 CRR:
    - Must be setup for each region
    - Files are updated in near real-time
    - Read only
    - Great for **dynamic** content that needs to be available at low-latency in **few regions**

##### CloudFront Signed URL / Signed Cookies vs S3 Signed URLS 
- Cloud Front Signed URL
    - Allow access to a path, no matter the origin
    - Account wide key-pair, only the root can manage it
    - Can filter by IP, path, date, expiration
    - Can leverage caching features
- S3 Pre-Signed URLs
    - Issue a request as the person who signed the URL
    - Uses the IAM key of the signing IAM principal
    - Limited lifetime

![image](images/cloudfront-signed-url.png)

## Day 10 notes:

### Networking - VPC

##### CIDR - IPv4

`
10.0.100.0/32
`
- 10.0.100.0 - Base IP
- /32 - Subnet Mask

10.0.100.0/n = 10.0.100.0 - 2^(32-n)

/32 allows for 1 IP = 2^0
/31 allows for 2 IP = 2^1
etc

- `10.0.0.0 - 10.255.255.255` -> Big private network
- `172.16.0.0 - 172.31.255.255` -> Default AWS range
- `192.168.0.0 - 192.168.255.255` -> home network

**AWS RESERVES 5 IP ADDRESS FROM EACH ALLOCATED CIDR BLOCK**


# Practice tests

### Practice test numero uno, attempt one
`Result : 34/65 `
`Percentage: 52%` 
- Notes:
    - DDOS = AWS Shield
    - Messaging broker = Amazon MQ
    - EFS-IA lifecycle policy max day = 90
    - **AWS Resource Access Manager (RAM)** is a service that enables you to easily and securely share AWS resources with any AWS account or within your AWS Organization. You can share AWS Transit Gateways, Subnets, AWS License Manager configurations, and Amazon Route 53 Resolver rules resources with RAM.
    - **Amazon Macie** is an ML-powered security service that helps you prevent data loss by automatically discovering, classifying, and protecting sensitive data stored in Amazon S3
    - Always remember that the messages in the SQS queue will continue to exist even after the EC2 instance has processed it, **until you delete that message**
    - Active directory = Microsoft Active Directory xD
    - Memory utilization is not readily available in CloudWatch
    - Need to find out what Direct connect and File gateway are.
### Practice test numero dos
`Result : 44/65 `
`Percentage: 67%` 
- Notes:
    - TAPE = slow
    - **AWS Directory Service Simple AD** provides subset of features of **AWS Directory Service**
    - Read questions more carefully i guess haha
    
### Practice test #2, attempt one
`Result : 34/65`
`Percentage: 52%`
- Notes:
    - ECS with open source = EKS
    - You must install the CloudWatch agent on the instance to view **SwapUtilization** metric
    - You can use Amazon SNS message filtering to assign a filter policy to the topic subscription
    - If it is moving large amount of data, it is almost always AWS Snowball
    - To host a static web on S3 and routing with 53, the bucket name and the domain has to be same(if i want tushig.fucking.sucks.com the bucket name has to be tushig.fucking.sucks)
    - assess and audit all the configurations in their AWS account => AWS Config
    - Since it is a Windows bastion, you should allow RDP access and not SSH as this is mainly used for Linux-based systems.

### Practice test #3, attempt one
`Result : 39/65`
`Percentage: 60%` Definitely could`ve gotten better
- Notes:
    - Cannot use standby RDS instance for read operations while primary instance running
    - A Kinesis data stream stores records from 24 hours by default to a maximum of 168 hours.
    - `Active-Active` failover - When you want *all* your resources available 
    - `Active-Passive` failover - When you want your *primary* resource available 
    - The largest object that can be uploaded to S3 in a single PUT is 5 GB 
    - 1 subnet maps 1 AZ
    - Columnar data = AWS Redshift
    - There is a vCPU-based On-Demand Instance limit per region

### Practice test #4, attempt one
`Result : 38/65`
`Percentage: 58%` Definitely could`ve gotten better
- Notes:
    9.11.14.17.21.26.28.30.33 it says enhanced but idk ?.34.39.
    - **Magnetic volumes** provide the lowest cost per gigabyte of all EBS volume types and are ideal for workloads where data is accessed infrequently, and applications where the lowest storage cost is important.
    - **An Elastic Fabric Adapter (EFA)** is a network device that you can attach to your Amazon EC2 instance to accelerate High Performance Computing (HPC) and machine learning applications.
    - Weirdly, A **storage optimized instance** is designed for workloads that require high, sequential read and write access to very large data sets on local storage. 
    - **Transit Gateway**![image](images/transit-gateway.png)
