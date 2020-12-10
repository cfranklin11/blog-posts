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

## II. AWS Fundamentals: ELB + ASG

### A. Scaling & Availability

- Vertical scaling: get a bigger instance/server
- Horizontal scaling: get more instances/servers
  - Auto-scaling group
  - Load balancing among instances
- High availability: run instances across multiple AZs

### B. Load balancing

- Load balancers forward traffic to one or more server instances
- Spread load across servers and give single point of access (DNS) to the application
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
- SNI:
  - Solves problem of using multiple certificates on single web server to serve multiple webistes (load balancer)
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