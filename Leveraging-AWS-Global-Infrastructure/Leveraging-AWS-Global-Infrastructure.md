# Leveraging the AWS Global Infrastructure

1. [Why Global Applications?](#why-global-application)
    - Benefits of a Global Application
    - Global AWS Infrastructure
    - Global Applications in AWS
2. [Route 53 Overview](#route-53-overview)
    - What is DNS?
    - Common DNS Record Types in AWS
    - Route 53 – Diagram for A Record
    - Route 53 Routing Policies
    - Summary of Routing Policies
3. [CloudFront Overview](#cloudfront-overview)
    - Key Benefits of CloudFront
    - CloudFront Origins
    - How Does CloudFront Work? - High-Level Overview
    - CloudFront – S3 as an Origin
    - Summary
    - CloudFront vs S3 Cross Region Replication
4. S3 Transfer Acceleration
5. AWS Global Acceleration
6. AWS Outposts
7. AWS WaveLength
8. AWS Local Zones
9. Global Application Architecture
10. Leveraging the AWS Global Infrastructure Summary


## <a id="why-global-application" href="#why-global-application">Global Infrastructure in AWS</a>

**Why would you build a global application?**

- A global application is one that is deployed across multiple geographic locations.
- When applied to AWS, this means deploying your application across different AWS Regions or Edge Locations.

### Benefits of a Global Application

**1\. Reduced Latency**

- Deploying your application globally helps reduce latency for users around the world.
- Latency refers to the time it takes for a network packet to travel from the user's device to a server.
- Since the Earth is large, the farther the server is from the user, the longer it takes for the data to travel.
- For example, 
    - if a user is in India and the server is located in the US, there will be increased latency---meaning more lag and slower response times.
    - However, if you deploy your application closer to your users (for instance, in Asia for your users in India), they will experience lower latency and a better overall experience.
    - By deploying your application in both the US and Asia, users in each region can access servers that are geographically closer, resulting in faster performance.

**2\. Disaster Recovery**

- Another important reason to build a global application is to enable disaster recovery.
- You don't want to rely on a single data center or region. If an entire AWS region were to go down---due to an earthquake, storm, power outage, political issue, or other unexpected event---your application could become unavailable.
- Although a complete region failure is rare, it's not impossible.
    - By having a disaster recovery (DR) strategy in place---such as replicating your application to another region---you can ensure that your application continues to function even in the face of regional failures.
    - This increases the availability and resilience of your application.

**3\. Improved Security Against Attacks**

- Lastly, having a global application helps protect against cyberattacks.
- It's common for hackers to target applications and attempt to bring them down for various reasons.
    - If your application is deployed in multiple regions and distributed globally, it becomes significantly more difficult for an attacker to take down all instances at once.
- This distribution enhances your application's security and resistance to such attacks.


### Global AWS Infrastructure
- Now, AWS has a really cool website--- https://infrastructure.aws/ ---that displays all the AWS Regions, Availability Zones, and more.
- This site shows the Regions where we deploy our applications and infrastructure.
    - As we know, **Regions** are made up of multiple **Availability Zones**, and each Availability Zone consists of one or more **data centers**.
    - In addition to that, there are **Edge Locations**, which we haven't discussed yet. These are also known as **Points of Presence**.
    - Edge Locations are used for **content delivery** and are strategically placed to be as close as possible to users, helping reduce latency and improve performance.

### Global Applications in AWS

**Global DNS**, we'll be learning about **Amazon Route 53**.

-   It helps route users to the closest deployment with the least latency, improving performance.
-   It's also a great tool for implementing disaster recovery strategies.

For the **Global Content Delivery Network (CDN)**, we'll be using **Amazon CloudFront**.

-   CloudFront replicates parts of your application to Edge Locations to reduce latency and improve the user experience.
-   It also caches frequently accessed content, speeding up responses and reducing the load on your backend systems.

**S3 Transfer Acceleration** 

-   S3 Transfer Acceleration helps accelerate uploads and downloads to and from Amazon S3, especially for users across the globe.

**AWS Global Accelerator**

- AWS Global Accelerator enhances the availability and performance of global applications by leveraging the AWS global network infrastructure.

* * * * *

## <a id="route-53-overview" href="#route-53-overview">Amazon Route 53 Overview</a>

The first service that's essential for deploying a global application is **Amazon Route 53**.

**Route 53** is a **managed DNS** (Domain Name System) service.

### What is DNS?

- Think of DNS as the **phone book of the internet.**
- It contains a set of rules and records that help clients (like web browsers) **find the correct servers using URLs.**

### Common DNS Record Types in AWS

Let's go over the most common DNS record types:

- **A Record:**
    - Maps a domain name (e.g., www.google.com) to an **IPv4 address.**
    - www.google.com => 12.34.56.78 == A record (IPv4)

- **AAAA Record (also called "quad-A" record):**
    - Maps a domain name to an **IPv6 address.**
    - www.google.com => 2001:0db8:85a3:0000:0000:8a2e:0370:7334 == AAAA IPv6

- **CNAME Record:**
    - Maps **one hostname to another hostname.**
    - For example, mapping search.google.com to www.google.com.
    - search.google.com => www.google.com == CNAME: hostname to hostname

- **Alias Record (specific to AWS):**
    - Maps a hostname directly to an **AWS resource**, such as:
        -   Elastic Load Balancer (ELB)
        -   CloudFront distribution
        -   S3 static website
        -   RDS instance
    - example.com => AWS resource == Alias (ex: ELB, CloudFront, S3, RDS, etc...)
    - Alias records behave like CNAMEs but are more flexible and optimized for AWS services.

> ### Exam Tip
> For the exam, you **don't need to memorize all the record types**, but it's important to understand the basic ones and know that **Route 53 is a managed DNS service** in AWS.

### <u>Understanding How DNS Works with Route 53 (A Record Example)</u>

Let's go one step further to better understand how DNS works.

#### Route 53 – Diagram for A Record

![](Route-53-Diagram-for-A-Record.png)

For example, consider **Amazon Route 53**.
- Suppose we want to see what happens when we use an **A record**.
- We have a **web browser** on the client side, and an **application server** that we've deployed with a **public IPv4 address**.
- We want to access this application server using a human-readable URL instead of an IP address.
- To achieve this, we configure a DNS record in **Route 53**:
    1.  **Create an A record** in Route 53 that maps the domain name (e.g., `myapp.mydomain.com`) to the server's public IP address.
    2.  When the web browser attempts to access `myapp.mydomain.com`, it makes a **DNS request**.
    3.  The DNS service (Route 53) responds with the associated IP address.
    4.  The browser then uses this IP address to connect to the application server.
    5.  Finally, the server returns an **HTTP response** to the browser.
- This is the basic process of how DNS resolution works using an A record --- at a high level.

### Route 53 Routing Policies

> From an exam perspective, it's important to understand the different Route 53 routing policies. You should be familiar with them at a high level and be able to choose the right one based on the use case. Fortunately, the core concepts are quite straightforward.

Here are the four main routing policies you need to know:

#### 1\. Simple Routing Policy

- **Description:** This is the most basic routing policy.
- **Behavior:** When a user (via a web browser) performs a DNS query, Route 53 returns a single IPv4 address (or whatever is configured).
- **Health Checks:**  Not supported.
- **Use Case:** Simple, single-instance applications where health checks are not required.

![](Simple-Routing-Policy.png)

#### 2\. Weighted Routing Policy

- **Description:** Distributes traffic across multiple resources by assigning weights.
- **Behavior:** Each resource (e.g., EC2 instances) is given a weight. Route 53 uses these weights to determine how much traffic should be routed to each resource.
    - Example: If we assign weights of 70, 20, and 10 to three instances, then approximately:
    - 70% of the traffic goes to the first instance,
    - 20% to the second,
    - 10% to the third.
- **Health Checks:**  Supported --- Route 53 will only route traffic to healthy instances.
- **Use Case:** Basic load balancing across multiple endpoints.

![](Weighted-Routing-Policy.png)

#### 3\. Latency Routing Policy

- **Description:** Routes users to the region that provides the lowest latency.
- **Behavior:** If your application is deployed in multiple regions (e.g., California and Australia), Route 53 will determine the region that offers the best performance based on the user's location.
    -   A user closer to California will be directed to the California instance.
    -   A user closer to Australia will be directed to the Australian instance.
- **Health Checks:**  Supported.
- **Use Case:** Global applications that require low-latency access for users across different geographical regions.

![](Latency-Routing-Policy.png)

#### 4\. Failover Routing Policy

- **Description:** Provides high availability and disaster recovery by defining primary and secondary (failover) resources.
- **Behavior:**
    -   Route 53 continuously checks the health of the primary instance.
    -   If the primary becomes unhealthy, traffic is automatically routed to the failover instance.
- **Health Checks:**  Required on the primary (and optionally on the secondary).
- **Use Case:** Applications requiring failover mechanisms to ensure uptime during outages or failures.

![](Failover-Routing-Policy.png)

#### Summary of Key Points

| Routing Policy | Supports Health Checks | Primary Use Case |
| --- | --- | --- |
| Simple | ❌ No | Single resource, basic routing |
| Weighted | ✅ Yes | Load balancing based on traffic share |
| Latency | ✅ Yes | Route users to nearest region |
| Failover | ✅ Yes (required) | Disaster recovery / high availability |


> By understanding these four routing policies, you’ll be well-prepared for questions related to DNS and Route 53 on the exam.

* * * * *

## <a id="cloudfront-overview" href="#cloudfront-overview">Amazon CloudFront Overview</a>

**Amazon CloudFront** is a **Content Delivery Network (CDN)** service that improves the performance, scalability, and security of your applications by caching content at globally distributed **edge locations**.

> ✅ **Exam Tip**: When you see **CDN** in the exam, think **CloudFront**.

### Key Benefits of CloudFront

#### 1\. **Improved Performance via Global Caching**

-   CloudFront caches your website content (e.g., images, videos, static files) at edge locations around the world.
-   This means users will receive content from the **closest** edge location, reducing latency and improving load times.
-   Example:
    -   A user in the U.S. accessing a website hosted in Australia will get the content from a **U.S. edge location** (after initial fetch).
    -   The next U.S. user accessing the same content will get it directly from the cache, without needing to go to Australia.

#### 2\. **Global Reach**

-   CloudFront operates **hundreds of Points of Presence (PoPs)** globally, including:
    -   **Edge locations** (where content is served)
    -   **Regional edge caches** (used for more efficient caching and performance)

#### 3\. **Built-in DDoS Protection**

-   CloudFront helps protect your application from **Distributed Denial of Service (DDoS)** attacks.
-   Since content is distributed globally, your infrastructure is less vulnerable to large-scale, centralized attacks.
-   CloudFront integrates with:
    -   **AWS Shield** -- for automatic DDoS protection
    -   **AWS WAF (Web Application Firewall)** -- for filtering and blocking malicious web requests

### Example Scenario

Let's say:

-   You have a **static website hosted in an S3 bucket in Australia**.
-   A user in the **U.S.** requests that content.

#### First Request:

-   The U.S. user is routed to the **nearest CloudFront edge location** in the U.S.
-   CloudFront fetches the content from the **S3 bucket in Australia** and caches it at the edge.

#### Subsequent Requests:

-   The same content is served **directly from the U.S. edge location**, significantly reducing latency.

#### Another User in China:

-   A user in China will be served via a **Chinese edge location**.
-   CloudFront will fetch the content (if not cached) from the S3 bucket and then **cache it locally** for future requests.

Source: https://aws.amazon.com/cloudfront/features/?nc=sn&loc=2

![](cloudFront.png)

### CloudFront Origins -- Connecting to Backends

**CloudFront** supports multiple types of **origins**, which are the **backends** that CloudFront fetches content from. You can configure different types of origins based on your application architecture and performance/security needs.

#### ✅ 1. **Amazon S3 Bucket (Standard Origin)**

-   Commonly used for distributing static files (e.g., HTML, CSS, JavaScript, images).
-   CloudFront caches this content at edge locations for faster access globally.
-   **Secure Access**:
    -   CloudFront connects to S3 using **Origin Access Control (OAC)**.
    -   OAC ensures that only CloudFront can access the S3 bucket (no public access required).
-   **Use Case**: Delivering static content securely and efficiently.

#### Optional:
-   You can also **upload files to S3 through CloudFront**, enabling reverse proxy use cases (e.g., pre-signed POST URLs).

#### ✅ 2. **VPC-Based Origins (Private Applications)**

If your application backend is hosted **within a VPC** (e.g., private subnets), you can use CloudFront to connect to these resources **privately**, typically via an **AWS service** like:
-   **Application Load Balancer (ALB)**
-   **Network Load Balancer (NLB)**
-   **Amazon EC2 Instances**

This setup allows you to use CloudFront as a secure, scalable, and globally distributed entry point to your internal applications.

#### ✅ 3. **Custom HTTP Origins**

-   CloudFront also supports **custom origins**, which are any backends that respond over **HTTP or HTTPS**.
-   Examples:
    -   A website hosted on Amazon S3 **(must be configured for static website hosting)**.
    -   Any **public HTTP backend**, either inside or outside of AWS.
-   Ensure that the origin is properly configured to handle requests from CloudFront (e.g., CORS, caching headers, etc.).


### How Does CloudFront Work? -- High-Level Overview

CloudFront uses **edge locations** distributed all around the world. These edge locations are responsible for handling requests from users and connecting to your **origin**, which could be an **Amazon S3 bucket** or an **HTTP server**.

![](CloudFront-High-Level.png)

- When a client sends an **HTTP request** to the nearest edge location, CloudFront first checks whether the requested content is already stored in the **cache** at that location.
    -   If the content **is cached**, it is returned immediately to the client.
    -   If the content **is not cached** (a cache miss), the edge location fetches the content from the **origin**.
- Once the content is retrieved from the origin, it is **cached at the edge location**. This means that if another client requests the same content from the same edge location, CloudFront can serve it **directly from the cache**, without needing to contact the origin again.

### CloudFront – S3 as an Origin

![](CloudFront-S3-as-a-origin.png)

#### Example: Using S3 as the Origin

Let's say your origin is an **Amazon S3 bucket** hosted in a specific region (e.g., Sydney). That bucket is your origin in the cloud. You have **edge locations** around the world.

For example, 
- One in **Los Angeles**.
    -   When a user in Los Angeles accesses your content, CloudFront routes the request to the **Los Angeles edge location**.
    -   If the content is not yet cached there, the edge location **retrieves it from the S3 bucket** over AWS's **private network**.
    -   The content is then **cached at the Los Angeles edge location**, and served to the user.
    -   For future requests, that content is served **directly from the cache**, reducing latency.

The connection between CloudFront and the S3 bucket is secured using **Origin Access Control (OAC)**, and access is managed through appropriate **S3 bucket policies**.

- Another User in **São Paulo**
    -   Now, let's say a user in **São Paulo, Brazil** makes a similar request.
    -   The request is routed to the **nearest edge location** in Brazil.
    -   If the content isn't cached, it is fetched from the origin (S3 bucket), and then **cached at the São Paulo edge location** for future requests.

This same pattern repeats globally, with edge locations **serving nearby users**, reducing the need to constantly contact the origin.

### Summary

By using CloudFront:
-   Content stored in an S3 bucket (or other origin) in **one region** can be **distributed globally** via edge locations.
-   Users receive content **faster and more reliably**, thanks to **local caching**.
-   Secure connections between CloudFront and your origin are maintained using **OAC** and proper **bucket policies**.
-   This architecture improves performance, reduces load on your origin, and enhances user experience worldwide.

### What's the Difference Between CloudFront and S3 Cross-Region Replication (CRR)?

While both services help with **data distribution**, they serve **very different purposes**:

#### **Amazon CloudFront** -- Global Content Caching (CDN)

-   **CloudFront** is a **Content Delivery Network (CDN)**.
-   It uses a **global edge network** with over **216 Points of Presence (PoPs)**.
-   Content is **cached temporarily** at edge locations---usually for a few minutes to several days (based on TTL settings).
-   Ideal for **static content** (images, videos, HTML, etc.) that needs to be delivered quickly to users **anywhere in the world**.
-   CloudFront **does not store content permanently**---it retrieves and caches it from an **origin** (like an S3 bucket or HTTP server) on demand.

> ✅ **Use Case:** Serve static content globally with low latency and reduced load on the origin.

#### **S3 Cross-Region Replication (CRR)** -- Permanent Multi-Region Storage

-   S3 **Cross-Region Replication** is a feature of Amazon S3 that **automatically replicates objects** from one bucket to another **in a different AWS region**.
-   You must configure replication **for each target region**---this is not a global setup.
-   It **does not cache content**. Instead, it **permanently replicates** and stores objects in the destination bucket.
-   Updates happen in **near real-time**, maintaining consistency between buckets.
-   Replicated data is usually **read-only** in the target bucket unless configured otherwise.

> ✅ **Use Case:** Maintain **durable, real-time copies** of data in multiple AWS regions for **disaster recovery, compliance, or low-latency access** in specific regions.


#### Key Differences

| **Feature** | **CloudFront** | **S3 Cross-Region Replication** |
| --- | --- | --- |
| **Purpose** | Global caching and content delivery (CDN) | Permanent data replication between regions |
| **Global Coverage** | Yes -- 216+ edge locations worldwide | No -- must be configured for specific regions |
| **Caching** | Yes (temporary) | No (permanent replication) |
| **Storage** | No (pulls from origin) | Yes (replicates to S3 buckets) |
| **Best for** | Static content delivery | Dynamic content replication and regional access |
| **Latency Benefits** | Global low latency | Regional low latency |
| **Content Type** | Static (e.g., images, videos) | Dynamic or critical data (e.g., backups, logs) |


### Summary

-   **CloudFront** is used for **performance optimization** and **global distribution** of content through caching.
-   **S3 Cross-Region Replication** is used for **data redundancy**, **resilience**, and **regional availability**.

These services are **complementary**, not interchangeable. In many architectures, you might use both together---for example, replicate S3 data across regions and serve it globally via CloudFront.

* * * * *

## S3 Transfer Acceleration -- Overview

As we know, **S3 buckets are tied to a single AWS region**.\
Sometimes, you may need to **transfer files from various locations around the world** into **one specific S3 bucket**.

To speed up these long-distance transfers, AWS offers a feature called **S3 Transfer Acceleration**.

### How It Works

Let's say a file is being uploaded **from the United States** to an **S3 bucket located in Australia**.

![](S3-Transfer-Accelaration.png)

Here's what happens:

1.  Instead of sending the file directly to the S3 bucket over the public internet, the file is first uploaded to a **nearby CloudFront edge location**---for example, one close to the user in the U.S.
2.  From there, the file is transferred to the S3 bucket in Australia using **AWS's high-speed internal network**.
3.  This results in a **faster and more reliable** transfer compared to using the public internet end-to-end.

This is the basic concept behind how **S3 Transfer Acceleration** works.

### When to Use It

S3 Transfer Acceleration is most useful when:

-   You or your users are located **far from the region** where the S3 bucket is hosted.
-   You are **uploading or downloading large files** and want to reduce latency and improve performance.

Test the tool at: https://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html

* * * * *

## AWS Global Accelerator

- AWS Global Accelerator is used to **improve the availability and performance of global applications** by leveraging the AWS global network.
- The core idea is that your requests are routed through AWS's internal network, allowing you to optimize the route to your application by up to 60%.

Let's look at an example.

![](Global-Acceleration.png)

Suppose we have deployed an application hosted in India, and users from around the world want to access it. With AWS Global Accelerator, users connect to the nearest AWS edge location. From there, the traffic is routed through AWS's private global network directly to the application in India.

The key benefit is that traffic only travels over the public internet for a short distance---between the user and the nearest edge location. After that, it uses AWS's high-speed private network to reach the application, which significantly improves performance and reliability.

The same process applies for users in Europe, Australia, and other regions.

Additionally, you only need to expose your application through two static IP addresses, known as **Anycast IPs**. These static IPs automatically route traffic to the optimal edge location based on the user's geographic location.

From the edge location, the traffic is forwarded directly to your application.

### **Understanding AWS Global Accelerator with a Diagram**

There's a useful diagram on the AWS website that clearly shows the difference between using AWS Global Accelerator and not using it.

![](Global-Acceleration2.png)

https://aws.amazon.com/global-accelerator/


#### ❌ **Without Global Accelerator:**

When clients try to connect to your application hosted in a specific AWS region, their requests travel across the public internet. This means:

-   The traffic may pass through **many network hops**.
-   It is subject to **variable latency**, **packet loss**, and **network congestion**.
-   There is no guarantee of an optimized or reliable route.

As a result, performance can suffer---especially for users located far from the AWS region where your application is hosted.

#### ✅ **With Global Accelerator:**

When you use AWS Global Accelerator, here's what changes:

-   Clients connect to the **nearest AWS edge location**.
-   From the edge location, traffic is routed through **AWS's private global network** directly to your application's region.
-   This reduces the number of hops and avoids unpredictable public internet routes.

Because the AWS network is **highly optimized**, **low-latency**, and **secure**, the result is:

-   Faster connections
-   Improved reliability
-   More consistent user experience across the globe

This is especially valuable for global applications with users in different continents.
