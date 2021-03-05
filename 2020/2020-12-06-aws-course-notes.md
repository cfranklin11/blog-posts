Points of Discussion

- 4/1
  - Pros/Cons of different auto-scaling options
    - ASGs
    - ECS
      - w/ EC2
      - Fargate
      - Cluster Capacity Provider
  - Must enable ECS_ENABLE_TASK_IAM_ROLE in `/etc/ecs/ecs.config` to allow ECS tasks to endorse IAM roles?
- 18/1
  - ElasticBeanstalk seems good, but haven't heard of many people using it. Do many people us it? If not, why?
  - What are the pros/cons of AWS CodePipeline vs other self-hosted CI/CD services?
    - Pros seem to be it'a all AWS, so integration/authentication is easier/better
    - Cons seem to be shit UI/UX
  - More detail/explanation?
    - SNS/Lambda good for deletion of branches, pushes to master, notify build system
    - CloudWatch good for PR updates, commit comments
    - Seems to be "Git things" vs "Git service things"
  - Mentioned Terraform, Chef, etc. while talking about CodeDeploy, then said that CodeDeploy _doesn't_ provision servers, which I thought was the main purpose of those other tools, so what are the similarities/differences exactly?

# AWS Certified Developer

## I. AWS Fundamentals: IAM & EC2

### A. IAM

- AWS services are separated into regions (e.g. ap-southeast-2, us-west-1) and availability zones (AZ), which are separate data centres within each region (denoted by letters, e.g. ap-southeast-2b, us-west-1a).
- Regions are completely separate (no shared resources), AZs are connected on the same network.
- IAM is organised into users (people), groups (function/job or team), role (internal to AWS, apply to machines)
  - One user per person
  - One role per application
  - Apply policies to users or groups to define authorized activity
- IAM Federation is for large companies that want to integrate their existing user system into AWS.
  - Uses the SAML standard (Active Directory)

### B. EC2

- Instances have characteristics:
  - RAM
  - CPU
  - I/O (disk performance)
  - Network (bandwidth, latency)
  - GPU
- T2/T3 are "burst" instances
  - Poor CPU performance in general, but can spike to good CPU under load
  - Use up "burst credits", when they run out, CPU becomes bad

#### 1.**SSH**

- Create a security group that allows access via port 22 to be able to SSH into the server.
- To SSH into the server, use `ec2-user@<IP Address>`
- By default the `.pem` SSH key file is too open ("Permissions 0644"), which means AWS won't let you use it to SSH into a server
  - To fix this, run `chmod 0400 <EC2 key file>.pem` to change the permission level
- Troubleshooting
  - Connection timeout error is an issue with security group settings or due to a firewall
  - Connection refused means reachable but no SSH utility: restart the server
  - Permission denied: Wrong or missing key file or wrong user
- EC2 Instance Connect (SSH via browser) only works with AWS Linux servers

#### 2. **Security Groups**

- Security groups control inbound and outbound traffic for AWS servers (i.e. firewalls)
- Can apply to multiple servers, and each server can have multiple groups
- Bound to a given region and/or VPC
- Live outside EC2 instances, so blocked traffic doesn't appear in server logs et al
- By default all inbound traffic is blocked and all outbound traffic is allowed
- A security group can authorise other security groups
  - So when server A has security group A, which authorises access for security group B, and server B has security group B, server B is authorised to access server A regardless of IP rules/restrictions
- Security groups can reference IP addresses, CIDR blocks, or other security groups

#### 3. **IP**

- IPv4 is most commong (IPv6 is mostly for IoT)
- IPv4 has format [0-255].[0-255].[0-255].[0-255]
- Public IP is accessible from the internate; private IP only accessible within private network
- IPs must be unique within their respective networks (globally for public IPs, within the given network for private IPs)
- Elastic IPs can be assigned to servers, but they should be avoided
  - Better to use a random public IP and register a DNS to it
- Restarting a server can change its public IP

#### 4. **EC2 User Data**

- Run basic scripts to set up a new server

#### 5. EC2 Launch Types

- On demand instances: short workload, predictable (hourly) pricing
  - Pay for what you use (uptime only)
  - Highest cost, but nothing upfront, nor any commitments
- Reserved (minimum 1 year)
  - Reserved instances: long workloads (e.g. databases)
    - Pay upfront and long-term commitment, but up to 75% discount
    - Reserve specific instance type for 1-3 years
  - Convertible reserved instances: flexible instance types (can change server size over the year+)
    - Up to 54% discount
  - Scheduled reserved instances: fixed schedule for the year+ (e.g. run jobs on certain days/times)
    - For when you don't need 100% up time
- Spot insances: short workloads, cheap, can lose instances (not reliable)
  - Up to 90% discount
  - Bid on server space
  - Can get kicked off at any time while using a server based on how much you bid (i.e. cheapskates get booted first)
  - Good for workloads resilient to interruption/failure
  - Good to combine launch types to minimise costs (e.g. spot instances to handle peak usage + on-demand to handle baseline)
- Dedicated instances: no sharing hardware
  - Automatic instance placement
- Dedicated hosts: book an entire server, control instance placement
  - More control, visibility into sockets + physical cores
  - 3 year contract
  - Useful for cases where software has complicated licensing
  - Sometimes needed for compliance/regulatory reasons

#### 6. Elastic Network Interfaces

- Part of a VPC, represents a virtual network card
- Has a primary, private IPv4, one or more secondary IPv4s
  - Can also have an Elastic IP or public IPv4
  - Can have one or more security group
  - Can have a MAC address
- ENIs can be moved among EC2 instances (e.g. for failovers)

#### 7. **Pricing**
- Pricing is per hour and depends on:
  - Region
  - Instance type
  - Launch type (e.g. on-demaind, spot)
  - Billed by second (minimum of 60 seconds)
- You don't pay while instance id stopped

#### 8. **AMIs**
- You can use an existing AMI (Amazon Machine Image) or create a custom one
- Custom AMIs have some advantages:
  - Pre-install software
  - Faster boot times (due to pre-installed)
  - Custom configuration
  - More control over software & updates
  - Optimised for a given task/use case
- Custom AMIs are region specific

## II. AWS Fundamentals: ELB & ASG

### A. Scaling & Availability

- Vertical scaling: get a bigger instance/server
- Horizontal scaling: get more instances/servers
  - Auto-scaling group
  - Load balancing among instances
- High availability: run instances across multiple AZs

### B. Load balancing

- Load balancers forward traffic to one or more server instances
- Spread load across servers and give single point of access (DNS or IP) to the application
- Handle server failures by redirecting traffic to healthy servers
- Perform regular health checks on servers
- Provides SSL termination (HTTPS)
- Enables high availability
- Separates public & private traffic
- Health check: ping a route & port (e.g. /health:80) to check for 200 response
- Security groups:
  - Load balancer accepts ports 80/443 from all addresses
  - EC2 accepts port 80 from the load balancer security group only
- Load balancer troubleshooting
  - 4xx = client induced errors
  - 5xx = server induced errors
  - 503 = at capacity or no registered target servers
- Stickiness: always assign a given client to the same target (uses cookie to keep track of target)
  - May create traffic imbalances
  - Configured at the target-group level
- Cross-zone balancing
  - Load balancers distribute traffic across all AZs that have targets regardless of which AZ they're in

#### 1. Elastic Load Balancer (ELB)

- Managed load balancer from AWS
- Costs more, but takes care a lot of the stup & maintenance work & integrates with other AWS services
- Exposes static DNS (URL) to public traffic
- Classic load balancer (v1, old generation, 2009)
  - Supports: HTTP, HTTPS, TCP
- Application load balancer (v2, new generation, 2016)
  - Supports: HTTP, HTTPS, WebSockets
- Network load balancer (v2, new generation, 2017)
  - Supports: TCP, TLS (secure TCP), UDP
- Internal (private) or external (public) load balancers available in ELB
- Cross-zone balancing always on and free of charge
- Connection Draining ("deregistration delay" in the context of target groups):
  - Time to complete in-flight requests while the instance is de-registering or unhealthy
  - Instance waits a given amount of time for existing connections to finish while the LB redirects new connections to healthy instances
  - Set time limit to low value if requests are quick, high value if they take awhile

#### 2. Classic Load Balancer (CLB)

- Supports TCP (layer 4) and HTTP/HTTPS (layer 7)
- Cross-zone balancing available but off by default, free of charge

#### 3. Application Load Balancer (ALB)

- Supports HTTP/HTTPS, HTTP/2, and WebSocket (layer 7)
- Can balance to multiple applications across machines or on the same machine (e.g. multiple containers)
- Routing to different target groups via:
  - path in URL
  - hostname in URL (domain or subdomain)
  - query string or headers
- Has port mapping for redirecting to dynamic ports (useful for containers or microservices)
- Target groups can be:
  - EC2 instances (via HTTP)
  - ECS tasks (via HTTP)
  - Lambda functions (via HTTP translated into JSON event)
  - (private) IP addresses
- Has a fixed hostname (but not static IP)
- Includes client IP & protocal info in special headers: `X-Forwarded-Port` & `X-Forwarded-Proto` for use by the target group
- Create rules for using target groups via load balancer "Listeners"

#### 4. Network Load Balancer (NLB)

- Supports TCP & UDP (layer 4)
- Can handle more requests with less latency than ALBs (high performance)
- Has one static IP per AZ and supports assigning Elastic IP
- Don't have security groups
  - Configure targets' security group to allow TCP on target port from anywhere (0.0.0.0/0)
- Cross-zone balancing disabled by default, costs money for inter-AZ traffic balancing

#### 5. SSL/TLS

- SSL certificate allows traffic between client & load balancer to be encrypted (in-flight encryption)
- Secure Sockets Layer: encrypts the connection
- Transport Layer Security (TLS): newer version
- Most use TLS now, but everyone still calls it SSL
- SSL certificates issued by Certificate Authorities (CA)
- Certificates have expiration dates and must be renewed
- You can manage certificates via AWS Certificate Manager (ACM)
- When creating an HTTPS listener:
  - Specify default certificate
  - Can add optional list of certificates to support multiple domains
  - Clients can use Server Name Identification (SNI) to specify target hostname
- Optionally create security groups to support older versions of SSL/TLS
- Server Name Indication (SNI):
  - Solves problem of using multiple certificates on single web server to serve multiple websites (load balancer)
  - Client indicates hostname of target server in SSL handshake
  - Server returns correct certificate for that server or default one
  - Only works on ALB, NLD, or CloudFront

### C. Auto-scaling groups

- Can be configured w/ load balancer to automatically connect new instances with the LB
- Settings:
  - AMI & instance type
  - EC2 user data
  - EBS volumes
  - security group(s)
  - SSH key pair
  - min size, max size, & initial capacity
  - network & subnet
  - load balancer info
  - scaling policies
- Alarms trigger scale out/in based on scaling policies
  - based on metrics averaged across entire ASG
  - accepts custom metrics (but data must be sent manually to CloudWatch)
- ASGs use Launch configurations or Launch Template (newer)
  - Updating ASG requires new configuration/template
- IAM role assigned to an ASG also gets assigned to EC2 instances in that ASG
- ASGs are free; you just pay for resources being used (EC2, EBS, etc.)

#### 1. Scaling policies

- Target tracking scaling
  - Target resource usage (e.g. 40% CPU across all servers)
- Simple/Step scaling
  - Trigger alarm at defined usage (e.g. 75% CPU), then take defined action (add 2 servers to group)
  - Can trigger scale up or down events
- Scheduled actions
  - Scale up or down a defined amount at scheduled days/times
- There are cooldown periods to allow scale up/down actions to take effect before re-assessing if more action is needed
- Can override default cooldown with custom, group-specific cooldowns
- Common to give scale-down actions shorter cooldowns, because the response is quicker (i.e. shutting down servers is quicker than spinning them up)

## III. EC2 Storage: EBS & EFS

### A. Elastic Block Store (EBS)

- Network drive attached to an EC2 server while it runs (common for DBs)
- Can be detached from an EC2 and attached to another one easily (but only one at a time)
- Locked to an AZ
  - Can migrate to another AZ by taking a snapshot, then restoring the snapshot in another AZ
- EBS volumes get terminated when the attached EC2 is terminated by default (but can be changed via settings)
- Define size (GB, I/O Ops per Secons or IOPS) on creation
  - Billed by provisioned capacity, not capacity used
  - You can increase the size later
- EBS types:
  - GP2 (SSD): general purpose, balances price & performance
  - IO1 (SSD): high performance SSD (use: critical DBs)
  - ST1 (HDD): low-cost HDD for frequently accessed, high throughput (use: big data)
  - SC1 (HDD): lowest-cost HDD for less-frequently accessed data
- ONly GP2 or IO1 can be boot volumes

#### 1. GP2 (SSD)

- Like t2 instances, can temporarily burst IOPS
- Has max IOPS per GB of storage (3 IOPS per GB)
- Minimum 100 IOPS, burst to 3,000 IOPS, maximum 16,000 IOPS

#### 2. IO1 (SSD)

- Higher IOPS limit than GP2, but more expensive
- Mostly for large, business critical DBs
- Set IOPS independent of memory, but like GP2, bases max IOPS on storage size
- Minimum 100 IOPS, maximum 64,000 IOPS (Nitro) or 32,000 IOPS (other)

#### 3. ST1 (HDD)

- Mostly for streaming workloads (big data, log processing, kafka)
- Lots of throughput (MiBs/second)
- Max 500 MiB/second

#### 4. SC1 (HDD)

- high throughput, but less-frequently accessed than ST1 workload
- Max 250 MiB/second

#### 5. Instance store

- Instance store is ephemeral storage
- Physically attached to instance harddrive
- Mostly for caching, buffers, temporary content
- Better I/O performance, very high IOPS (no network)
- data survives reboots, but is lost on termination
- Can't resize after creation
- Backups are manual

### B. Elastic File System (EFS)

- Managed network file system (NFS) that can be mounted on EC2 instances
- Usable by EC2s across AZs
- Highly available & scalable, but expensive
- Pay for what you use
- Can connect to multiple EC2s across AZs at the same time
- Use cases: content management, web serving, data sharing
- Uses NFSv4.1 protocol
- Control access with security groups
- Works on Linux-based servers only
- Encryption at rest using KMS
- POSIX filesystem (which is Linux only)
- Can scale up a lot (up to thousands of NFS clients and 10+ GB of throughput)
- Performance mode (set at creation time):
  - General purpose: latency-sensitive use cases (web serving, CMS)
  - Max I/O: higher latency, higher throughput, highly parallel (big data, media processing)
- Storage tiers
  - Standard: frequently-accessed files
  - Infrequently-accessed (EFS-IA): cost to retrieve files, but lower cost to store
  - Lifecycle management setting will automatically move files from Standard to EFS-IA after X number of days

## IV. AWS Fundamentals: RDS & Aurora & ElastiCache

### A. Relational Database Service (RDS)

- Availabe DBs:
  - MySQL
  - Postgres
  - MariaDB
  - Oracle
  - Microsoft SQL Server
  - Aurora (AWS's DB)
- Managed services:
  - Automated provisioning, OS patching
  - Backups & restores (Point in Time Restore)
  - Monitoring dashboards
  - Read replicas
  - Multi-AZ setup for disaster recovery (DR)
  - Maintenance windows for upgrades
  - Scalibility (horizontal and/or vertical)
  - Storage backed by EBS (GP2 or IO1)
- No SSH connection (unlike a DB in an EC2 instance)
- Automated backups:
  - Full back up every day
  - Transaction logs backed up every 5 minutes
  - Backups last 7 days by default (up to 35 days)
- DB snapshots:
  - Manually triggered backup
  - Retained as long as you want

#### 1. Read replicas vs Multi-AZ

- Read Replicas
  - Up to 5 replicas
  - Can be same AZ, cross AZ, or cross region
  - Async replication, so reads are _eventuall_ consistent
  - Requires application to use connection that has read replicas enabled
  - Use case: analytics/reporting uses replica to not slow down main app
  - Network costs
    - Cross-AZ traffic/data costs extra, so cross-AZ replicas can be expensive
- Multi-AZ
  - Synchronous replication to standby instance in different AZ
  - Automatic failover to standby (better availibility)
  - No application change needed
  - Not used for scaling, but can use a read replica as a multi-AZ standby as well

#### 2. Security & Encryption

- At-rest encryption
  - can apply to master & read replicas with AWS Key Management Service (KMS) AES 256 encryption
  - Must be defined at launch time
  - If master isn't encrypted, replicas _cannot_ be encrypted
  - Transparent Data Encryption (TDE) available for Oracle & SQL Server
- In-flight encryption
  - Uses SSL certificates to encrypt data
  - To enable for Postgres, need to use a Parameter Group in the AWS console (rds.force_ssl=1)
  - For MySQL (within the DB), run `GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;`
- Snapshots default to whether the main DB is encrypted or not
- Can copy an unencrypted snapshot and make the copy encrypted
- Can use this to migrate from unencrypted DB to encrypted by restoring the DB from the encrypted snapshot
- Deploy to private subnet, not public (i.e. not exposed to public internet)
- Uses security groups to manage access
- Regular username/password to log into the DB like normal
  - Can use IAM for RDS MySQL or Postgres
  - Get access token from IAM API (15 min lifetime)

### B. Aurora

- Works with either MySQL or Postgres DB drivers (configure at creation time)
- "AWS cloud optimized": 5x performance over MySQL, 3X over Postgres
- Storage automatically grows 10GB at a time, up to 64TB
- Can have 15 replicas (as opposed to 5), and replication is faster
- Failover is instantaneous + good HA
- Costs ~20% more than RDS
- 6 copies of your data across 3 AZs
  - 4 copies needed for writes
  - 3 copies needed for reads
  - Self-healing with peer-to-peer replication (fixes bad/corrupt data)
  - Storage is striped across 100s of volumes for reliability, shared across DBs & AZs
- 1 DB takes writes (master)
- Automatic failover in 30 secs if master goes down
- Master + up to 15 read replicas serve reads
- A Writer Endpoint receives write calls and directs to master (redirecting to a new master in case of failover)
- A Reader Endpoint receives read calls and directs to read replicas with load balancing
  - Load balancing happens at connection level
- Similar security features as RDS
- Aurora Serverless: automated DB instantiation & auto-scaling based on usage
  - Good for infrequent jobs
  - No capacity planning needed
  - Cost per second of use
  - Proxy Fleet manages scaling/instantiation
- Global Aurora
  - Cross-region read replicas
  - Aurora Global DB (recommended)
    - 1 primary region for read & writes
    - Up to 5 secondary regions for read only
    - Up to 16 read replicas per secondary region
    - Can promote a secondary region to primary, changes in < 1 minute

### C. ElastiCache

- Managed Redis/Memcached
- Write scaling w/ sharding; read scaling w/ replicas
- Multi-AZ with failover
- A lot of the same benefits of RDS
- Cache DB calls or store user sessions (especially good for multi-instance or application architectures)
- Redis
  - Multi-AZ with auto-failover
  - Scaling with read replicas
  - Data durability using AOF assistance (i.e. if server restarts)
  - Backups & restores
  - Cluster mode for greater scalibility/availability
- Memcached
  - Multi-node for data partioning (sharding)
  - Non-persistent
  - No backup, no restore
  - Multi-threaded architecture
- Caching considerations
  - Is it safe to cache? (how quickly does data change?)
  - Is caching effective?
    - slow to change, few keys to check = effective
    - change quickly and/or lots of keys to check = ineffective
  - Good data structure for caching? (i.e. key/value pair structure)
- Lazy Loading / Cache-Aside / Lazy Populations
  - Request hits cache or proceeds to DB and writes to cache
  - Pros:
    - Only requested data is cached
    - Node failures aren't fatal, just have to warm the cache (i.e. re-request & re-write)
  - Cons:
    - Cache miss is slow: 3 requests to complete (read cache, read DB, write cache)
    - Stale data: changes in DB don't update cached data
- Write Through
  - Update cache when data in DB changes
  - Pros:
    - Cache data is never stale
    - Write penalty = 2 calls (write DB _and_ write cache), which fewer than 3 and less-noticeable from UX perspective
  - Cons:
    - Missing data from cache until update (mitigation is to use Lazy Loading as well)
    - A lot of cache data will never be read
- Cache Evictions & Time-to-live (TTL)
  - 3 forms of cache eviction:
    - Delete data manually
    - Memory is full and data hasn't been used recently (LRU)
    - Time-to-live limit (i.e. configuration such that data only lasts X amount of time)

## V. Route 53

- Managed Domain Name System (DNS)
- Common AWS DNS records:
  - A: hostname to IPv4
  - AAAA: hostname to IPv6
  - CNAME: hostname to hostname
  - Alias: hostname to AWS resource
- Route 53 can use:
  - public domain names you own
  - private domain names resolved by your instances in your VPCs
- Features:
  - Load balancing via DNS (client load balancing)
  - Health checks (limited)
  - Routing policies (e.g. simple, failover, weighted)
  - Pay per month, per hosted zone
- Time-to-live (TTL)
  - Caches DNS response in browser to reduce load on DNS
  - High TTL (e.g. 24hrs) reduces load, but can have stale data
  - Low TTL (e.g. 60secs) increases load, but data is fresher
  - TTL required for each DNS record
- CNAME can only point to non-root domains (i.e. _not_ `mydomain.com`)
- Alias works for root and non-root domains
- Alias records are free and have native health checks
- Routing Policy
  - Simple
    - When you need to redirect to a single resource
    - Doesn't have health checks
    - If multiple values _are_ returned, the client chooses one at random
      - Can be used for client-side load balancing
      - Values can be from different regions
  - Weighted
    - Sends x% of requests to each possible server (calculated as % given weight number is of total weight)
  - Latency
    - Redirects to server with lowest latency for the client (i.e. the closest)
  - Health checks
    - Unhealthy if fails 3 times in a row (default)
    - Healthy if it succeeds 3 times in a row (default)
    - Default interval = 30 secs
    - Fast health check = 10 sec interval, but costs more
    - Health checks can be linked to DNS queries (changing w/ Route53 DNS changes)
  - Failover routing policy can be used for a primary and secondary DNS record to have a backup if health check fails.
  - Geo-location routing policy: if user is located in X, route them to server Y (with default for locations without a policy)
  - Multi-value routing policy
    - Used for routing traffic to multiple resources (same domain, different IPs & health checks)
    - Up to 8 healthy records returned per query (even if more exist)
    - Not replacement of ELB, but contributes client-side load balancing


## VI. Virtual Private Cloud (VPC) Fundamentals

- Virtual network associated with specific region
- Subnets partition the network inside the VPC (associated with specific AZ)
  - Subnets can be public or private
  - Route tables define access to, from, and among, subnets
- Has a range if IPs (CIDR range)
- Internet gateway (IGW) connects VPC to public internet
  - Public subnet has route to the internet gateway
- NAT gateway (AWS managed) or NAT instance (self-managed) give private subnets access to internet while still being private (i.e. unaccessible themselves)
  - NAT goes inside public subnet, private subnet connects to the NAT, the NAT connects to public subnet's IGW
- Network ACL (NACL) is a firewall to control traffic to/from subnet
  - Attached to subnets
  - Allow/Deny rules apply to specific IPs
  - Stateless
  - Like security groups, controls what traffic can reach resources
    - SGs are stateful and operate at EC2 instance or ENI level
- VPC flow logs include logs of all traffic flowing in/out of the VPC, subnets and Elastic Network Interface (ENI)
  - For debugging network issues
- VPC peering connects to VPCs as if they were part of same network
  - CIDR ranges for VPCs must not have overlapping IP ranges
  - Not transitive: A is connected to B is connected to C, but A cannot communicate with C unless they are directly connected as well
- VPC endpoints allow you to connect to AWS services via private network when those services are outside the VPC
  - VPC endpoint gateway for S3 & DynamoDB
  - VPC endpoint interface for all other AWS services
- Site-to-Site VPN
  - Connects an on-premises VPN with an AWS VPC
  - Automatically encrypted
  - Goes over public internet
- Direct Connect (DX)
  - Physical connection from on-premises VPN to AWS VPC
  - Over private network
  - Requires a month to set up
- Nether above can use VPC endpoints
- 3-tier architecture: Route 53 DNS -> Public subnet Elastic Load Balancer -> Private subnet EC2 instances -> Data subnet DB/Cache
- LAMP stack: Linux, Apache (web server), MySQL, PHP

## VII. Amazon S3 Introduction

- S3 bucket name must be globally unique
- S3 is globally available, but a regional resource
- Naming conventions
  - No uppercase
  - No underscore
  - 3-63 characters long
  - Not an IP address
  - Start with letter or number
- Object key is the full path to the file
  - Composed of any directories inside the bucket (the prefix) and the object name
- Max object size = 5TB
- If object > 5GB, must use multi-part upload
- Versioning
  - Enabled at the bucket level
  - Uploading file w/ same key doesn't overwrite the file, but creates a new version of it
  - When enabling versioning, existing objects have version = null
- S3 can host static websites

### A. Encryption

- SSE-S3: encryption w/ keys handled & managed by AWS S3
  - Encrypted server-side w/ AES-256
  - Use special header to set encryption when uploading file
- SSE-KMS: use AWS Key Management Service
  - KMS advantages: user control & audit trail
  - Server-side encryption
  - Also uses special header when uploading files
- SSE-C: manage your own encryption keys
  - Amazon does not have the key, you provide it yourself
  - Must use HTTPS when uploading the file
  - Must provide the encryption key in the headers for each HTTP request when uploading files
  - Can only be done via CLI/API call, not console UI
- Client-side encryption
  - You encrypt the file before uploading it
  - AWS has an encryption SDK for performing this
- S3 has HTTP endpoint, but using HTTPS endpoint is recommended
- Read after write consistency for PUT new objects (i.e. newly-uploaded objects are immediately available)
  - Exception is if you perform a GET, then PUT, then GET, which is eventually consistent
- DELETE and PUT for existing objects are eventually consistent (i.e. there's a delay between the change and GETting the updated object)

### B. Security

- User-based: IAM policies resctricting access
- Resource-based:
  - S3 bucket policies, apply cross-account
  - Object Access Control List (ACL): object-level policies
  - Bucket Access Control List (ACL): less-common form of bucket-level policies
- A user can access an S3 object if their IAM policies permit it OR the resource policies permit it AND there's no explicit denial
- Policies can grant access or enforce encryption of objects
- You can block public access to buckets through policies or ACLs
- Can use VPC endpoints to access privately
- Can save access logs in an S3 bucket or CloudWatch

### C. CORS

- Origin: schema (protocol), host (domain), and port
- CORS tells web browser to allow requests to other origins while visiting the main origin (doesn't allow by default)
- Uses special headers to allow CORS
- S3 bucket needs to enable CORS headers to enable cross-origin requests
- S3 buckets have CORS settings, where you can define allowed origins, response headers, etc.

## VIII. AWS CLI, SDK, IAM Roles & Policies

- There's a policy simulator for testing IAM permissions
- For AWS SDK, if you don't define a region, us-east-1 is used by default
- Credentials priority when using an SDK:
  1. Environment variables
  2. Java system properties
  3. CLI credentials file (`~/.aws/credentials`)
  4. CLI config file (`~/.aws/config`)
  5. Container credentials (for ECS tasks)
  6. Instance profile credentials (EC2 instance profiles)
- Different services have different rate limits (API calls per endpoint per second)
  - For intermittent errors, use exponential backoff (retry mechanism included in the SDK)
  - For consistent errors, request increase in limit from AWS
- Service limits are the max number of instances of a service you can have on an account (e.g. there's a limit to number of EC2 instances you can run)
- Calls to the AWS API need to be signed by credentials
  - CLI & SDK do this automatically
  - If making raw API calls, use Signature v4 to sign the requests
  - HTTP header option uses `Authorization` header with credentials info
  - Query string option adds query params with the same info as above

### A. AWS CLI

- If you get error "aws: command not found", it means `aws` is not in the PATH env var
- Don't use personal credentials for AWS CLI on EC2 instances, give it an IAM role, with access policies, and that will be enough to use AWS CLI
- It can take time for IAM role changes to propagate
- CLI has --dry-run to test out commands
- Decode error messages with STS command line (decode-authorization-message)
- Instance metadata is so EC2 instances can know about themselves
  - The URL for the info is http://169.254.169.254/latest/meta-data
  - Internal to AWS (only accessible from inside an EC2 instance)
  - Can get IAM role name, but not IAM policy
  - This is how EC2 instances get access to resources via IAM roles: fetch temporary credentials from metadata & use those to call resource APIs
- Use different profiles for different accounts, and `--profile` with CLI calls to use non-default profiles
- To use MFA, call STS GetSessionToken to create temporary session: `aws sts get-session-token --serial-number <ARN of MFA device> --token-code <MFA token> --duration-seconds <how long the temp creds last>`
  - Returns an access key, secret key, and session token
- Credentials priority when using CLI:
  1. Command line options
  2. Environment variables
  3. CLI credentials file (`~/.aws/credentials`)
  4. CLI config file (`~/.aws/config`)
  5. Container credentials (for ECS tasks)
  6. Instance profile credentials (EC2 instance profiles)


## IX. Advanced S3 & Athena

### A. Advanced S3

- MFA-Delete
  - Requires MFA codes to do anything major in S3
  - Requires object versioning to be enabled
  - When enabled, requires MFA to:
    - Delete object version
    - Disable versioning
  - Doesn't require MFA to:
    - Enable versioning
    - Listing deleted versions
  - Can only be toggled by the bucket owner (i.e. 'root' account)
  - Can only be enabled via CLI for now
- Access Logging
  - Can save S3 access logs in another S3 bucket
  - Never set the monitored & logging bucket to be the same (infinite loop)
- S3 Replication
  - Asynchronous replication
  - Requires versioning
  - Can use Cross-Region Replication (CRR) or Same-Region Replication (SRR)
  - Can be from different accounts
  - After enabling, only new objects are replicated (not retroactive)
  - Permanent DELETE operations are not replicated, but replication of delete marker operations ("soft delete") can be enabled in settings
  - No chaining (i.e. replication operations cannot trigger other replication operations, meaning bucket 1 replicating into bucket 2 won't replicate into bucket 3)
- Pre-signed URLs
  - CLI can do it for downloads, must use SDK for uploads
  - By default, valid for 1 hr (can change w/ arg)
- S3 scales to 3,500 PUT/COPY/POST/DELETE per sec and 5,500 GET/HEAD per sec per prefix
  - If you use SSE-KMS, its limits can affect S3 speed, because it encrypts/decrypts w/ each upload & download call
  - KMS limit is 5,500/10,000/30,000 calls per sec (depending on region)
- Multi-part upload: recommended for files > 100MB, required for > 5GB
- S3 transfer acceleration (upload only): uploads to an edge location, then forwarded to your S3 bucket
- Speed up downloads with S3 Byte-Range Fetches
  - Breaks up file's bytes into chunks, allowing retries for partial failure
- S3 Select: fetch data form CSV file with SQL, filtering by rows and/or columns
  - Less processing on client-side
- S3 can generate notifications for events (e.g. object upload, download)
  - Can send these to SNS, SQS, or Lambda functions
  - To ensure all events are sent, enable versioning (to avoid conflicts of multiple events on the same object)
- AWS Athena
  - Service to do analytics on S3 data files
  - Uses SQL to query file data
  - Charged per query + amount of data fetched
  - Supports CSV, JSON, Avro, Parquet, ORC
- Object Lock
  - Write once, read many (WORM)
  - Block object/archive change/deletion for a given amount of time
  - Useful for compliance, data retention, auditing purposes
- Glacier Vault Lock
  - Like above, but also lock the policy to prevent changes

### B. S3 storage classes

- Standard
  - General purpose
  - High durability, high availability across AZs (almost never lose an object)
- Standard Infrequent Access (IA)
  - Infrequently accessed, but needed immediately when accessed
  - Same durability as above, ever-so-slightly less availability
  - Cheaper than above (assuming infrequent access)
  - Has a retrieval fee
- One Zone Infrequent Access
  - Slightly lower availability, and not resisitant to an AZ failure
  - About 20% cheaper than above
- Amazon S3 Intelligent Tiering
  - Service that automatically moves objects among access tiers to save costs
  - Costs monthly fee
- Amazon Glacier
  - Low-cost for archiving/backup
  - Long-term storage (10s of years)
  - Very-low storage cost, but object retrieval costs money
  - Objects are 'Archives' (up to 40TB files) and are stored in 'Vaults'
  - 3 retrieval options:
    - Expedited: 1-5 mins
    - Standard: 3-5 hrs
    - Bulk (multiple files): 5-12 hrs
  - Archives stored for min of 90 days
- Amazon Glacier Deep Archive
  - Even cheaper than above
  - Retrieval options:
    - Standard: 12 hrs
    - Bulk: 48 hrs
  - Min storage duration = 180 days
- Amazon S3 Reduced Redundancy Storage (deprecated)
- Can create lifecycle rules to trigger actions on objects
  - Transition actions: move objects among storage classes
  - Expiration actions: delete objects
  - Rules can be created for prefixes (only apply to certain files) or tags

## X. CloudFront

- Is a Content Delivery Network (CDN)
- Caches content at edge locations
- Includes DDoS protection, integration with AWS Shield, and AWS Web Application Firewall
- Can expose external HTTPS and talk to internal HTTPS endpoints
- When used with S3
  - Can use Origin Access Identity for increased security
    - Bucket is private, and only CloudFront can access the contents of the S3 bucket via IAM policies
  - Can be used for uploads at the edge
- Can be used with a custom origin: load balancer, EC2, S3 website, etc.
  - Must be in a public security group
- Can configure geographic restrictions (i.e. users in country X can/cannot access this resource)
- Caching can be based on headers, session cookies, or query string params
- Good idea to have two CloudFront distributions:
  - One for static content with full caching
  - One for dynamic content with more-careful caching rules
- Can manually invalidate a cache by creating an 'invalidation'
- HTTPS
  - Viewer Control Policy (for connection between client & edge location)
    - Redirect from HTTP to HTTPS or allow HTTPS only
  - Origin Protocol Policy (for connection between edge location & origin)
    - HTTPS only or match viewer (client) protocol
  - Note: S3 websites don't support HTTPS (HTTP only)
- To restrict access to CloudFront to certain users, use a Signed URL or Cookie
  - Includes URL expiration, IP ranges that can access data, trusted signers (AWS accounts that can create signed URL/Cookie)
  - Signed URL for individual file; signed cookie for multiple files
- Signed URL
  - Allow access to a path for any type of origin
  - Account-wide key/pair, only root can manage it
  - Accesses S3 objects via CloudFront as normal
- Pre-signed URL
  - Makes request as the account that created it
  - Uses IAM key of the signing IAM principal
  - Bypasses CloudFront to access S3 objects directly

## XI. ECS, ECR, & Fargate (Docker in AWS)

### A. Elastic Container Service (ECS)

- ECS cluster is group of EC2 instances
  - EC2s run ECS agent (Docker container)
  - ECS agent registers its EC2 with the ECS cluster
  - EC2s run special ECS-specific AMI
- Creating an ECS cluster creates an Auto-Scaling Group that creates any associated EC2s
- Types of ECS:
  - ECS w/ self-managed EC2 instances
  - Fargate (serverless)
  - EKS (Kubernetes)
- Task Definitions
  - JSON file that tells ECS how to run a Docker container (basically, `docker run` args)
  - Tasks need IAM roles to have permission to interact w/ AWS resources
- ECS Service runs tasks across all EC2s in the cluster (balanced per Task Placement settings)
- Can run multiple tasks: ECS tries to run within same EC2 or scales up to multiple if permitted by ASG (e.g. if there's a port # conflict)
- ECS w/ load balancer
  - Use Dynamic Port Forwarding in an Application Load Balancer when containers use random host ports (to avoid conflicts when running tasks in parallel)
  - Can only add a load balancer on ECS creation
  - Set up security group for ECS instances that accept traffic on all ports from the load balancer security group
- Fargate
  - Runs tasks (Docker containers) serverlessly (no managing EC2 instances)
  - Must choose Fargate option when creating service and task(s)
- IAM roles
  - EC2 instance profile gives permissions to the ECS agent running inside the EC2 instance to perform ECS-related tasks (make API calls to ECS service, send logs to CloudWatch, pull images from ECR)
  - ECS task roles give tasks permissions to interact w/ AWS resources based on what the task is for
  - Define task roles in the Task Definition
  - Must enable ECS_ENABLE_TASK_IAM_ROLE in `/etc/ecs/ecs.config` to allow ECS tasks to endorse IAM roles
- Logging
  - Integrates w/ CloudWatch
  - Configure at task definition level
  - Each container has own log stream
  - IAM permissions set in the EC2 instance profile

#### 1. ECS task placement

- Task placement strategy & constraints determine where to place new tasks and where to remove them when scaling down
- Only applies to ECS w/ EC2, not Fargate
- Strategies are more guidelines, constraints are more-strictly followed
- Task placement steps
  1. Find EC2s that satisfy CPU, memory, and port requirements
  2. Narrow down to EC2s that satisfy task placement constraints
  3. Narrow down to EC2s that best satisfy task placement strategies
  4. Place task in an EC2
- Task placement strategies
  - Binpack: place in EC2s w/ least available CPU or memory (minimises # of EC2s needed to save costs)
  - Random: does what it says on the tin
  - Spread: distributes tasks evenly across EC2s with a given attribute (e.g. all available AZs)
  - Can mix/match strategies
- Task placement constraints
  - distinctInstance: place each task on a different EC2 (i.e. no more than one task per EC2)
  - memberOf: place tasks on EC2s that satisfy a condition (uses Cluster Query Language, CQL)

#### 2. Autoscaling
- Auto-scaling settings are the same as for ASGs
- ECS Service Scaling is at the task level, not the EC2 instance level (unlike ASGs)
- Fargate bypasses this because it's serverless
- Cluster Capacity Provider defines auto-scaling rules for EC2s within an ECS cluster
  - Is associated with an ASG
  - When you run a task/service, define a Capacity Provider Strategy (instead of 'Launch Type'), which allows the provider to provision infrastructure
  - Smarter, more-flexible way of auto-scaling ECS service than standard ECS settings

### B. Elastic Container Repository (ECR)

- Private Docker image repository
- Uses IAM for permissions
- Logging in via CLI:
  - v1: `$(aws ecr get-login ...)` (to execute the output of the command)
  - v2: `aws ecr get-login-password ... | docker login ...` (to pass the password to `docker login`)
  - Once logged in, use docker commands to push/pull

## XII. AWS Elastic Beanstalk

- Platform as a Service (PAAS) that bundles basic application infrastructure (load balancer, ASG, cache, etc.) and manages it for you
- Beanstalk is free, you just pay for underlying services/instances
- Instance configuration, deployment strategy are configurable, but Beanstalk manages them
- Architecture models:
  - Single instance deployment (dev environments)
  - ALB + ASG (prod environments)
  - ASG only (worker processes in prod environments)
- Has 3 components
  - Application
  - Application version (each deployment has a version)
  - Environment name
- Deploy versions to environments and/or promote versions to the next environment (e.g. from `staging` to `production`)
  - Application to environment is one-to-many
- Can rollback application versions
- Supports a variety of runtimes (e.g. Python, Go, Java, Docker container)
- When creating an environment, can pick and choose which extra services you want (logging, load balancer, etc.)
  - Cannot change load balancer type after environment creation
  - To change load balancer type:
    1. Create new environment w/ same options except LB (manually, can't use clone)
    2. Deploy new environment
    3. Switch traffic from old environment to new
- Lifecycle policy
  - Limited to 1,000 application versions
  - Based on time (delete versions after X days) or space (delete all but the X newest versions)
  - Current version will never be deleted
  - Option to not delete code bundles from S3 (just deletes old versions from EB interface)
- EB Extensions
  - Replace console configuration w/ files in `.ebextensions/` in code bundle
  - Files are YAML or JSON and end in `.config`
  - Allows adding extra resources (ElastiCache, DynamoDB) that aren't available in the console
    - These resources depend on the environment, so deleted environment means the resource gets deleted as well
  - Can use CloudFormation here to provision anything in AWS
- Can clone an EB environment (new env w/ same resources & configuration)
- Can provision RDS via EB, but not ideal for production, because ties RDS lifecycle to EB lifecycle
  - Better to create RDS separately and provide connection string to EB
  - To migrate from EB RDS to independent RDS:
    1. Create snapshot (precautionary measure)
    2. In RDS console, protect the EB RDS from deletion
    3. Create new EB environment without RDS, but using connection to existing RDS
    4. Switch traffic to new EB environment
    5. Terminate old EB environment
    6. Delete CloudFormation stack manually (because delete will fail when trying to delete RDS)
- Using Docker
  - Can be based on Dockerfile (to build image) or Dockerrun.aws.json (to fetch existing image)
  - Note: this does not use ECS (just Docker on EC2)
  - Can use Multi-Docker Container to use ECS (requires Dockerrun.aws.json v2)
- To use HTTPS
  - Load SSL certificate onto the load balancer via the console, or via `.ebextensions/securelistener-alb.config`
  - SSL certificates can be provisioned via AWS Certificate Manager (ACM) or CLI
  - Must configure a security group to permit port 443
  - Either configure instances to redirect to HTTPS or configure LB to redirect
- For long tasks, use a dedicated worker environment
  - Use `cron.yml` to set up scheduled tasks
- Custom Platform allows for advanced configuration (OS, additional software, scripts, etc.)
  - Generally for code languages not natively supported by EB
  - Define AMI with `Platform.yml`, which uses Packer to build the AMI
  - Difference from Custom Image (AMI), which customises existing EB platform, is this creates whole new platform

### A. Deployments

#### 1. Deployment modes

- Single instance
  - Includes one Elastic IP (mapping DNS to this IP), one EC2 instance security group, one ASG in one AZ, one RDS
- High availability
  - Includes load balancer (optional), one ASG across multiple AZs, multiple EC2 instance security groups (one per AZ), multi-AZ RDS

#### 2. Deployment options for updates
- All at once
  - Deploys to all EC2s at same time
  - Fastest deployment
  - Requires downtime
- Rolling
  - Updates a few EC2s (or 'bucket') at a time, updating the next bucket after the current is healthy
  - Bucket size (# of EC2s per bucket) is configurable
  - No downtime, but diminished capacity during update
- Rolling with additional batches
  - Creates new instances with the update, with existing instances still running previous version
  - Still updates a few EC2s at a time
  - Guarantees full capacity during update
  - Extra cost due to running extra EC2s than needed for full capacity during update
- Immutable
  - Creates new instances in a new ASG to deploy the update
  - Switches from old EC2s/ASG to new once all the new EC2s are finished updating
  - Highest cost (double capacity during update) and longest deployment (creates whole new ASG)
  - Quick rollback in case of failures (just terminate new ASG)
  - Process:
    1. Create temporary ASG with one EC2
    2. After health check passes, create rest of EC2s in temp ASG
    3. Move new EC2s to current ASG
    4. Once temp ASG is empty, terminate old EC2s from current ASG
    5. Terminate temp ASG
- Blue/Green Deployment
  - Not a direct feature of Elastic Beanstalk
  - Can be achieved manually via weighted traffic w/ Route53 to test new version, followed by switching 100% over to new version
  - In Elastic Beanstalk, you can manually swap environment URLs to change which is being served in prod
- Elastic Beanstalk can be managed with the EB CLI (simpler than the AWS CLI)
- Deployment process
  1. Need to zip code files & describe dependencies via relevant file (e.g. requirements.txt, package.json)
  2. Upload zip file via console or CLI
  3. Deploy
  4. EB will deploy updated code to EC2s

## XIII. AWS CI/CD

- Tech stack for CI/CD (all covered under Orchestrate: AWS CodePipeline)
  - Code (GitHub, AWS CodeCommit)
  - Build (Jenkins CI, AWS CodeBuild)
  - Test (Jenkins CI, AWS CodeBuild as well)
  - Deploy (ElasticBeanstalk, AWS CodeDeploy)
  - Provision (ElasticBeanstalk or user-managed EC2s)

### A. CodeCommit

- AWS version control repository (private)
- No size limit
- Fully managed, highly available
- Code always stays inside AWS (increased security/compliance)
- Authentication via Git
  - SSH keys
  - HTTPS through AWS CLI Authentication Helper or generating HTTPS credentials
  - MFA available
- Authorization: Uses IAM policies
- Encryption
  - Automatically encrypted at rest with KMS
  - Automatically encrypted in transit, because can only use HTTPS or SSH
- Cross-account access: you use AWS IAM Role and the other person uses AWS STS (with AssumeRole API)
- Trigger notifications with AWS SNS, AWS Lambda, or AWS CloudWatch Event Rules
  - SNS/Lambda good for deletion of branches, pushes to master, notify build system
  - CloudWatch good for PR updates, commit comments
  - Notification target can be an SNS topic or Slack bot
  - Define triggers based on repo actions that target SNS topic or Lambda

### B. CodePipeline

- Orchestrates all the CI/CD tools/services
- Organised into stages (e.g. build, test, deploy)
  - Stages are broken up into action groups, which are further broken into actions
- Optionally allow for manual approvals between stages
- State changes (e.g. passed, failed, cancelled) can triggger CloudWatch Events, which can trigger SNS Notifications

#### 1. Troubleshooting

- State changes trigger CloudWatch events, which can create SNS notifications
- On failure, you can see more info in the console
- Use AWS CloudTrail to audit API calls to AWS services
- When CodePipeline can't perform an action, make sure the IAM Service Role has necessary permissions (i.e. check the IAM Policies)

### C. CodeBuild

- Fully managed (no servers for you to manage)
- Billed only for build time used (i.e. no idle servers)
- Integrates with other AWS service:
  - KMS for encrypting artifacts
  - IAM for build permissions
  - VPC for network security
  - CloudTrail for API calls logging
- Can use Docker images to create custom build environments
- Define instructions in `buildspec.yml`
  - Env vars (secrets via SSM Parameter Store)
  - Phases:
    1. Install
    2. Pre-build
    3. Build
    4. Post-build
  - Artifacts
  - Cache
- Logs to S3 and AWS CloudWatch
- Has metrics & alarms
- Can reproduce CloudBuild locally
- VPC
  - By default, instances launched outside of VPC can't access it
  - Configure VPC ID, Subnet ID, Security Group IDs to access a VPC

### D. CodeDeploy

- For deploying to EC2s not managed by Elastic Beanstalk (or on-premises instances or Lambda or ECS)
- EC2s must run CodeDeploy Agent
  1. Agent polls CodeDeploy
  2. CodeDeploy sends appspec.yml file
  3. Agent pulls code from repo (S3 or GitHub)
  4. EC2 runs deployment instructions for running app
  5. Agent reports success/failure to CodeDeploy
- Can be incorporated into CodePipeline & use artifacts from builds
- Works with any application, auto-scaling, blue/green deployments (EC2s only), AWS Lambda
- Does _not_ provision EC2 instances (they must exist already)
- Components
  - Application (i.e. the app name)
  - Compute platform (e.g. on-premises instances, EC2, Lambda, ECS)
  - Deployment configuration (rules for success/failure)
  - Deployment group
  - Deployment type (e.g. blue/green)
  - IAM instance profile (IAM for the EC2s)
  - Application revision (app code & appspec.yml)
  - Service role
  - Target revision
- AppSpec
  - File section (how to copy from S3/GitHub)
  - Hooks (instructions for deploying the new version of the app):
    1. ApplicationStop
    2. DownloadBundle
    3. BeforeInstall
    4. AfterInstall
    5. ApplicationStart
    6. ValidateService (i.e. health check)
- Deployment config
  - One-at-a-time (if one instance fails, deployment stops)
  - Half-at-a-time
  - All-at-once
  - Custom (e.g. Minimum healthy host = 75%)
- Failures
  - Instances stay in failed state till new deployment
  - Failed instances are deployed to before healthy ones
- Deployment targets
  - Set of tagged EC2s (or ASG) to deploy to
- Rollbacks
  - Can do manual or automatic rollbacks
  - Auto-rollbacks triggered by alarms
  - Rollback deploys a new version (with new ID) of the last healthy deployment version

### E. CodeStar

- Integrated service that ties together all the CI/CD tools on AWS (along with GitHub)
- Can integrate with issue trackers
- Can integrate with Cloud9 for web IDE (or other IDEs)
- Dashboard for all CI/CD stuff
- Free service (just pay for resources used)

## XIV. AWS CloudFormation

- Use templates (infrastructure as code) to replace manual process of creating/configuring resources
- As usual, this is free, but you're charged for the resources that you use
  - Can estimate resource costs with CloudFormation templates
- Templates saved in S3, can't edit, must create new version to supersede previous
- Creates stacks, which are identified by name
  - Deleting a stack deletes all associated resources
- CloudFormation automatically runs commands in correct order withou their having to be defined in the template (i.e. it doesn't matter what order you put them in)
- Use ChangeSets to see how template changes will alter resources before deploying
- Cross stack vs Nested stack
  - Use cross stack (connected via exported/imported values) for resources with different lifecycles that need to reference each other
  - Nested stacks are components that belong to other stacks (good for reuse of smaller stack configs)
- StackSets are for creating/updating/deleting stacks across multiple accounts/regions w/ a single command

### A. Resources

- Required section of template
- Represent the AWs resources/components manipulated by the template
- Can reference other resources in the template
- Each resource definition needs a Type and a list of Properties
- No dynamic code-generation/definitions: everything hardcoded

### B. Parameters

- Define inputs for the template
- Good for reusing templates in different contexts (e.g. companies, regions)
  - If a value is likely to change in the future, make it a parameter to avoid having to re-upload entire template
- `!Ref` function can be used to refer to the value of parameters or other resources
- AWS has built-in pseudo-parameters (e.g. `AWS::AccountId`) that give general info about the account or stack

### C. Mappings

- Fixed, hard-coded variables that are useful for differentiating among environments, regions, etc.
- For when you know the values in advance (i.e. which AMI to use per environment)
- Use `!FindInMap` to get map values: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey, ... ]`


### D. Outputs

- Optional outputs that can be exported, then imported to other stacks
- Useful for linking stack resources (e.g. output network info, so other stacks can connect to it)
- Good for collaboration across teams
- Cannot delete a stack whose outputs are being used by other stacks
- Exported output name must be unique per region
- Export output with `Export: Name: <output name>`
- Import value with `!ImportValue` function in another template

### E. Conditions

- Control creation or outputs based on environment, region, parameter, etc.


### F. Intrinsic Functions

- Ref
  - Get reference to another value in template
  - For paramaters, returns param value
  - For resources, returns resource ID
- Fn::GetAtt
  - Get attribute value of a resource (`!GetAtt <resource name>.<attribute name>`)
  - Available attributes vary by resource
- Fn::FindInMap
  - Get value from a `Mapping` object (can drill down for nested values)
- Fn::ImportValue
  - Get exported value by name
- Fn::Join
  - Join list of values into string w/ given delimiter
- Fn::Sub
  - String substitution with defined map object
- Condition Functions (e.g. Fn::If, Fn::Not, Fn::Equals)
  - Use to conditionally run other commands or define values

### G. Rollbacks

- Default is to auto-rollback if anything fails during creation/update (can be disabled)

## XV. AWS Monitoring & Audit: CloudWatch, X-Ray and CloudTrail

### A. CloudWatch

#### 1. Metrics

- Belong to namespaces
- Dimension is an attribute of a metric (for filtering/aggregation)
  - Can have up to 10 per metric
- Have timestamps
- Default EC2 monitoring is metrics every 5 mins
  - Detailed Monitoring gets metrics every 1 min, but costs more
- EC2 memory usage is not monitored by default, requires custom metric
- Custom metrics can have dimensions
  - Record metrics every 1 min by default, but High Resolution (StorageResolution parameter) can record up to every second, but costs more
  - Use PutMetricData API call to record custom metric
  - Use exponential backoff for throttle errors
- Metric Filters
  - Used to filter/aggregate metrics, and can be used to trigger alarms
  - Don't work retroactively, only as logs come in
#### 2. Alarms

- Used to trigger notifications based on metrics
- Can send alarm triggers to Auto-Scaling, EC2 Actions, SNS Notifications, etc.
- Various calculations available (sample, max, min, etc.)
- Alarm states:
  - OK (not triggered)
  - INSUFFICIENT_DATA
  - ALARM (triggered)
- Monitoring period: length of time in seconds to evaluate metric (for calculations)
  - High-resolution custom metrics only have 10 secs or 30 secs

#### 3. Logs

- Can send logs to CloudWatch via SDK (many AWS services send logs automatically)
- CloudWatch logs can be archived in S3 buckets or streamed to data analytics services (e.g. ElasticSearch)
- Structure:
  - Log groups (usually represents an application)
  - Log streams (series of logged events with a group)
- Can define log expiration policies at Log Group level
- Uses IAM permissions for sending logs
- Can encrypt logs at rest with KMS at group level
- EC2 doesn't send logs/events to CloudWatch by default (requires an agent)
- CloudWatch Logs Agent is older CW Unified Agent is newer (logs & metrics)
  - Unified Agent has central configuration via SSM parameter store

#### 4. Events

- Can have scheduled events or reactive events (i.e. something happens, and an event is created) called Event Pattern
- Creates a small JSON w/ event info
- Can send events to different targets (SNS, AWS Lambda, etc.)

### B. EventBridge

- Next eveolution of CloudWatch events
- Multiple event buses:
  - Default: events from AWS Services (CloudWatch Events)
  - Partner: events from SaaS providers (e.g. DataDog, Auth0)
  - Custom: events from your own applications
- Analyses events to infer schema
  - Schema Registry generates code for application that follows inferred event schema
  - Versioned
  - There are default schemas to use for AWS services
- Extends CloudWatch events via its event buses

### C. X-Ray

- Uses tracing to track requests across services
  - Each component adds a 'trace'
  - Annotations can be added to traces for more info
- Segments are per-application/service info
- Sub-Segments are more-details groupings within Segments
- Trace is a sequence of Segments that form an end-to-end trace
- Sampling is collection a % of traces (to reduce costs)
- Annotations are key/value pairs for indexing & filtering traces
- Metadata are key/value pairs that aren't indexed/searchable
- X-Ray Daemon can send traces cross-account (with proper IAM setup)
- Sampling
  - By default, records first request / sec + 5% of all other requests
  - First request is the 'resevoir', % is the 'rate' at which other requests are recorded
  - Can configure quantity for resevoir (e.g. first 10 / sec) and rate %
  - Can create other filtering rules (e.g. only POST requests, or requests to certain URLs)
- Write APIs
  - PutTraceSegments: uploads segment documents
  - PutTelemetryRecords: uploads metric-related data
  - GetSamplingRules: fetch sampling rules from AWS
  - GetSamplingTargets & GetSamplingStatisticSummaries (advanced APIs related to sampling rules)
- Read APIs
  - GetServiceGraph: main trace graph
  - BatchGetTraces: fetch list of traces based on IDs param
  - GetTraceSummaries: fetch IDs & annotations for traces (can be filtered by timeframe)
  - GetTraceGraph: fetch graph for one or more trace IDs

#### 1. Enabling X-Ray:
  - Basic steps:
    1. Add X-Ray SDK to application code
    2. Run the X-Ray daemon (e.g. on EC2) or enable X-Ray integration (e.g. on Lambda)
  - Enable X-Ray Daemon:
    - EC2: install X-Ray daemon via bash script provided
    - ElasticBeanstalk:
      - Enable in `.ebextensions` file
      - Enable via console option
    - ECS:
      - Run an X-Ray Daemon Task (i.e. one X-Ray Docker container per EC2 instances)
      - Alternatively, run X-Ray Daemon container as "sidecar" to app containers (i.e. one X-Ray container per app container)
      - Fargate must use "sidecar" pattern

### D. CloudTrail

- Tracks all events, API calls, etc. made within the AWS account
- Enabled by default
- Good for tracking changes to AWS resources (e.g. when something gets deleted)

## XVI. AWS Integration & Messaging: SQS, SNS, & Kinesis

- Each service represents a different message architectural model:
  - SQS: queue model
  - SNS: pub/sub model
  - Kinesis: real-time streaming model

### A. SQS

- Managed service for decoupling applications (separates 'producers' and 'consumers')
- No limit for throughput or quantity of messages in the queue
- By default, messages last 4 days, with a max of 14 days
- Has very low latency (< 10 ms)
- Limit to 256KB per message
- Sometimes sends duplicate messages (at-least-once delivery)
- Messages can arrive out of order (best-effort ordering)
- Consumer can pull up to 10 messages at a time
- ASGs can scale based on queue length (CloudWatch metric that can trigger CloudWatch alarm) when consuming from an SQS queue
- Security
  - In-flight encryption with HTTPS
  - At-rest encryption with KMS
  - Uses IAM policies
  - Uses SQS Access Policies (similar to S3 Bucket Policies)
- Messages have a visibility timeout: after received, they are invisible to other services for an amount of time
  - If a consumer takes too much time, a message can be processed twice
  - A consumer can extend the timeout with ChangeMessageVisibility
- Can set retry limit for messages that result in errors
  - After limit, send messages to Dead-Letter Queue (DLQ) for debugging
- Can delay messages before they appear in the queue (default is 0 secs)
- Long polling can be used to wait for messages if the queue is empty
  - Reduces # of calls, increases efficiency, & decreases latency
  - Can wait between 1 and 20 secs
  - Generally preferable to use long polling for 20 secs
- SQS Extended Client
  - Java library for enabling larger message size
  - Uses S3 to store data & message to queue with reference to data
- API calls to know
  - CreateQueue (with MessageRetentionPeriod), DeleteQueue
  - PurgeQueue
  - SendMessage (with DelaySeconds), ReceiveMessage, DeleteMessage
  - ReceiveMessageWaitTimeSeconds (i.e. perform long polling)
  - ChangeMessageVisibility (i.e. change visibility timeout)
  - Batch APIs: SendMessage, DeleteMessage, ChangeMessageVisibility
#### 1. FIFO queue
- Guarantees ordering (first-in-first-out)
- Exactly once delivery of messages
- Limited throughput (300 msg/s, 3,000 msg/s with batching)
- Name must end in `.fifo`
- De-duplication window is 5 mins (rejects any duplicate messages sent within 5 mins of original)
- De-duplication methods
  - Compare SHA-256 hashes of message bodies
  - Provide a message ID (MessageDeduplicationId)
- Message grouping
  - Can assign a MessageGroupID to group related messages
  - With multiple groups, ordering within groups is guaranteed, across groups it's not
  - Idea is to have one consumer per group

### B. SNS

- Message producer "publishes" to an SNS topic, and consumers who "subscribe" to that topic will receive the message
- Producer sends to one SNS topic, and all subscribers to that topic receive the message
- Up to 10,000,000 subscribers per topic
- Up to 100,000 topics
- Subscribers can be:
  - SQS
  - HTTP endpoint
  - Lambda
  - Email service
  - SMS messages
  - Mobile notifications
- Publishing
  - Publish to SNS topic via SDK
  - Direct publish to mobile apps (i.e. app notifications)
- Same encryption and access policies as SQS
- Available protocols for subscriptions:
  - HTTP
  - HTTPS
  - Email
  - Email-JSON
  - SQS
  - Lambda

### C. Kinesis

- AWS managed version of Apache Kafka
- Useful for real-time (streaming) big data
- Data is automatically replicated to 3 AZs
- 3 services:
  - Kinesis Streams: low-latency streaming at scale
  - Kinesis Analytics: real-time analytics using SQL
  - Kinesis Firehose: load streams into other services (e.g. Redshift, ElasticSearch) for long-term storage
- Structure is that Streams feeds into Analytics and/or Firehose
- Streams data is separated into shards (increase # shards for higher throughput)
  - Write throughput: 1 MB/sec, or 1,000 messages/sec, per shard
  - Read throughput: 2 MB/sec per shard
  - Billing is per shard
  - Shard count can be changed according to needs
  - Records are ordered per shard (ordered within given shard, no guaranteed ordering across shards)
- Data retained for 1 day by default (configurable up to 7 days)
- Can reprocess/replay data if necessary
- Can have multiple consumers of data
- Data cannot be deleted from Kinesis (immutable)
- Data can be processed in batches
- Data gets processed with a message key
  - Same key always goes to same shard (for correct ordering of that key's messages)
  - Messages get an incrementing sequence number to ensure ordering
  - Choose a well-distributed message key (i.e. without majority of data coming from a few IDs/groups) to avoid "hot shard" (i.e. overwhelming one shard that gets data from a key/ID with a lot of activity)
- ProvisionedThroughputExceeded error means too much data
  - Retries with backoff
  - Increase number of shards
  - Check partition key used for volume imbalance
- Consumers can use CLI, SDK, but there's also Kinesis Client Library
  - KCL uses DynamoDB to manage work
  - KCL is for reading records from streams in distributed applications
  - Each shard can be read by only one KCL instance
- Data Analytics
  - Uses SQL for querying data
  - Managed, auto-scaling, real-time
  - Pay for actual consumptions (like serverless)
  - Can produce further streams from queried/transformed data
- Firehose
  - Fully managed, near-real-time (60 sec delay), auto-scaling
  - Can save to a variety of data stores
  - Supports many data formats, but conversion between formats costs extra
  - Pay for actual usage

## XVII. AWS Serverless: Lambda

- Pay per request + how long the function runs for
- Run up to 15 mins per request
- Use up to 3GB of memory
- Increasing RAM will also improve CPU & network
- Main services to use for triggering Lambdas
  - API Gateway (REST API to invoke functions via HTTP request)
  - Kinesis (to perform data transformations)
  - DynamoDB
  - S3
  - CloudFront (for Lambda@Edge)
  - CloudWatch Events/EventBridge
  - CloudWatch Logs
  - SNS
  - SQS
  - Cognito (e.g. when a user logs in)
- Various built-in Lambda IAM roles to allow interaction with other services
  - Best practice: create one, specific execution role per Lambda function
- Use resource-based policies to allow other services to invoke Lambdas (similar to S3 bucket policies)
- Uses CloudWatch logs & metrics by default (assuming correct IAM role)
  - Can optionally add X-Ray integration via config & use SDK to add tracing
    - _X_AMZN_TRACE_ID: tracing header
    - AWS_XRAY_CONTEXT_MISSING: LOG_ERROR by default
    - AWS_XRAY_DAEMON_ADDRESS: <IP address>:<port> for X-Ray daemon
- Can have 123MB to 3,008MB of RAM
  - Increasing RAM limit increases vCPUs available
- Default timeout is 3 secs, but can increase to 900 secs
- Execution context is temp runtime environment for Lambdas
  - Is maintained for a short time between invokations for reuse
  - Includes `/tmp` directory (max size is 512MB) for transient file storage
  - Best practice: initialise reusable objects  (e.g. DB or SDK client) outside handler function to improve performance
- Can run up to 1,000 functions in parallel
- Can configure "reserved concurrency" to limit number of invocations
  - Synchronous: above limit returns 429 response
  - Asynchronous: above limit retries, then sends to DLQ
  - Limit applies to all Lambdas across the account (i.e. can run 1,000 total, not 1,000 per function)
- Provisioned Concurrency can allocate concurrency to avoid "cold start"
  - Can use Application Auto Scaling to manage concurrency
- Must include dependencies w/ code bundle
  - Up to 50MB directly to Lambda, larger saved in S3
- Possible to define simple function code inside CloudFormation
  - More common to save zip file in S3 and include reference in CF
  - Need to include version ID in reference to S3 file
- Each publish of a Lambda creates a new, immutable version (code + configuration)
  - Aliases are mutable pointers to immutable versions
  - Aliases allow blue/green by directing % of traffic to different versions
  - Aliases can't refer to aliases, just versions
- CodeDeploy can shift traffic from one version to another gradually over time (or all at once)
  - Can also handle rollbacks
- Limits (all are per-region):
  - Memory: 138 - 3,008 MB (64MB increments)
  - Max timeout: 900 secs (15 mins)
  - Env vars: 4KB
  - /tmp folder disk space: 512MB
  - Concurrent executions: 1,000 (can be increased upon request)
  - Max code bundle (compressed): 50MB (uncompressed = 250MB)

### A. Synchronous invocations

- When called via CLI, SDK, API Gateway, or Application Load Balancer

#### 1. Application Load Balancer

- Put Lambda in a target group
- ALB converts request into JSON object to pass to Lambda
- If you enable multi-header values in ALB, converts multiple values for same param key into an array of those values

#### 2. Lambda@Edge

- Deploy Lambdas with CloudFront CDN assets
- Used to change CloudFront requests/responses:
  - After CloudFront receives request from viewer (viewer request)
  - Before CloudFront forwards request to origin (origin request)
  - After CloudFront receives origin response
  - Before CloudFront forwards response to viewer (viewer response)
- Use cases
  - Security & privacy
  - Dynamic web application at the Edge
  - SEO
  - Route across origins & data centers
  - Bot mitigation at the Edge
  - Real-time image transformation
  - A/B testing
  - User authentication/authorization
  - User prioritization
  - User tracking/analytics

### B. Asynchronous invocations

- Event-based invocations (S3, SNS, CloudWatch)
- These events are placed in an internal Lambda event queue
- Lambda retries 3 times on errors:
  - 1 min after first retry
  - 2 min after second retry
- Lambdas should be idempotent for this reason
- Can create DLQ for Lambda events that error out
- Can create EventBridge or CodePipeline events to invoke Lambdas
- S3 events
  - Can take seconds up to a few minutes to arrive
  - Use versioning to guarantee each event gets a notification (unversioned object writes in quick succession can overwrite eachother and produce just one notification)

### C. Event Source Mapping

- Sources:
  - Kinesis
  - SQS
  - DynamoDB
- Records need to be fetched/polled by Lambda (i.e. Lambda is invoked synchronously)

#### 1. Streams (Kinesis or DynamoDB)

- Creates iterator for each shard (i.e. one Lambda per shard)
- Can start with new items or from a timestamp or from beginning
- Processed items aren't removed from stream
- Batch processing to group data before passing to Lambda
  - Can process batches in parallel (up to 10 per shard)
- On error, re-run Lambda until it succeeds or items expire
  - To guarantee in-order processing, all batches in given shard are paused until resolved

### 2. SQS

- Uses polling
- Can process 1-10 messages per batch
- Recommendation: set message visibility to 6x the Lambda timeout limit
- Configure DLQ on the SQS (not Lambda)
- On error, batches are returned to queue as individual items (can't guarantee same grouping for re-processing)
- Occasionally receive duplicate items (even without errors)
- Deletes item after processing
- Lambda scales up by 60 instances/min
- Can process up to 1,000 batches in parallel
- SQS FIFO:
  - Messages with same GroupID processed in order
  - Lambda scales up to number of message groups

### D. Destinations

- Send Lambda results to another service
- Can have different destinations for successful and failed invocations
- Asynchronous:
  - SQS
  - SNS
  - Another Lambda
  - EventBridge bus
- Event Source Mapping:
  - SQS
  - SNS
- Recommendation: use Destinations instead of DLQs because more flexible

### E. VPC

- By default Lambda is not launched in your VPC
- Can create Lambda inside VPC
  - Lambda creates an ENI to access VPC resources (requires AWSLambdaVPCAccessExecutorRole)
- Lambda in public subnet does not have access to public internet or public IP (unlike other resources)
  - To give it public access, use NAT Gateway or NAT Instance

### F. Lambda Layers
  - Can be used for custom runtimes (e.g. C++, Rust)
  - Can be used to store dependencies for re-use

## XVIII. AWS Serverless: DynamoDB

- NoSQL serverless DB
- Highly-available with replication across 3 AZs
- Made up of tables, each of which has a primary key
- Each table can have infinite items (i.e. rows)
- Items have attributes (i.e. column values)
- Max item size is 400KB


### A. Primary keys

- Partition key:
  - UUID per item
  - must be 'diverse' to distribute across instances
- Partition key + sort key:
  - combination must be unique
  - partition key groups data
  - 'sort key' is also called 'range key'
  - funtions similaraly to join tables
- Any field can be null except partition and sort keys

### B. Throughput

- Have to provision Capacity Units to configure throughput
- Separate CUs for read (RCUs) and write (WCUs)
- Can set up auto-scaling and/or use "burst credit" for scaling
- Raises ProvisionedThroughputException if you exceed capacity
- 1 WCU = one write per second for item up to 1KB
  - For fractions of KB, they get rounded up to calculate WCU (e.g. 6 items per sec at 4.3KB = 30 WCU)
- Reads can be "eventually consistent" or "strongly consistent"
  - Read type can be set in a parameter in the query
- 1 RCU = one strongly-consistent read (or two eventually-consistent reads) per second for an item up to 4KB
  - Like with WCUs, partials of 4KB get rounded up
- WCUs & RCUs are split as evenly as possible among partitions
- Common causes for exceeding throughput:
  - hot keys
  - hot partitions
  - reading/writing larger items, etc.
- Solutions for exceeding throughput:
  - Exponential backoff
  - Better distributeion of partition keys (so one partition isn't overloaded with requests)
  - Caching with DynamoDB Accelerator (DAX)

### Basic APIs

- PutItem: Write data to DynamoDB (create  new or replace existing)
- UpdateItem: update attributes of existing item
- Supports conditional writes
  - Helps with concurrent reads/writes
  - No performance impact
- DeleteItem: delete an item
  - Can do conditional deletes
- DeleteTable: deleta a whole table & its items
- BatchWriteItem: up to 25 PutItem or DeleteItem in one call
  - Limit 16MB of total data & 400KB per item
  - DynamoDB automatically processes items in a batch in parallel
  - Must retry failed items in a batch yourself
- GetItem: fetch items based on primary key
- ProjectionExpression: defines which fields to fetch when using GetItem
- BatchGetItem: fetch items in batches
  - Up to 100 items per batch
  - Up to 16MB of total item memory
  - Items retrieved in parallel
- Query: fetch & filter items
  - Query by the following:
    - PartitionKey value equality (i.e. '=' only)
    - SortKey value equality or range (e.g. '=', '>', '<='), optional
    - FilterExpression for client-side filtering
  - Returns up to 1MB of data or requested # of items (whichever is lower), then use pagination for more
  - More efficient way of fetching data
- Scan: fetch entire table, then filter items (inefficient)
  - Returns up to 1MB of data or requested # of items (whichever is lower), then use pagination for more
  - Consumes a lot of RCU
  - Use parallel scans to process multiple partitions for higher throughput (but more RCUs)