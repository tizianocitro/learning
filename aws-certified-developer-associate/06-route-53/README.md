# 6 Route 53

## 6.1 Domain Name System (DNS)

DNS translates the human friendly hostnames into the machine IP addresses. For example, www.google.com -> [172.217.18.36](172.217.18.36). DNS is the backbone of the internet and uses a hierarchical naming structure: [.com](.com), example.com, www.example.com, docs.example.com.

Some **terminologies**:
- **Domain Registrar**: Amazon Route 53, GoDaddy, etc.
- **DNS Records**: A, AAAA, CNAME, NS, etc.
- **Zone File**: contains DNS records to match hostnames to IP addresses.
- **Name Server**: resolves DNS queries (Authoritative or Non-Authoritative).
- **Top Level Domain (TLD)**: .com, .us, .in, .gov, .org, etc.
- **Second Level Domain (SLD)**: it is the part of the domain name that is directly to the left of the TLD. For example, in amazon.com, the SLD is amazon, and in google.com, the SLD is google.

The following is a **Fully Qualified Domain Name (FQDN)**:

![FQDN](/assets/aws-certified-developer-associate/fqdn.png "FQDN")

The following image explains how DNS works:

![DNS](/assets/aws-certified-developer-associate/dns.png "DNS")

## 6.2 Amazon Route 53

A **highly available, scalable, fully managed, and authoritative DNS**. Where authoritative means that the customer (you) can update the DNS records. 53 is a reference to the traditional DNS port.

![Route 53](/assets/aws-certified-developer-associate/route53.png "Route 53")

Route 53 is **also a domain registrar**, which means you can register domain names using it.

You have the ability to check the health of your resources.

The only AWS service which **provides a 100% availability SLA**.

### 6.2.1 Route 53 Records

Records define how you want to route traffic for a domain. Each record contains:
- **Domain/subdomain name**: e.g., example.com.
- **Record type**: e.g., A or AAAA.
- **Value**: e.g., [12.34.56.78](12.34.56.78).
- **Routing policy**: how Route 53 responds to queries.
- **TTL**: amount of time the record cached at DNS resolvers.

Route 53 supports the following DNS record types:
    - **A, AAAA, CNAME, NS (must know for the exam)**.
    - CAA, DS, MX, NAPTR, PTR, SOA, TXT, SPF, SRV.

### 6.2.2 Route 53 Record Types

- **A**: maps a hostname to IPv4.
- **AAAA**: maps a hostname to IPv6.
- **CNAME**: maps a hostname to another hostname.
    - The target is a domain name which must have an A or AAAA record.
    - You cannot create a CNAME record for the top node of a DNS namespace (**Zone Apex**).
    - For example, you cannot create for it for example.com, but you can create it for www.example.com.
- **NS (Name Servers for the Hosted Zone)**: DNS names or IP addresses of the servers that can respond to DNS queries for your **hosted zone**.
    - They control how traffic is routed for a domain.

A **hosted zone** is a container for records that define how to route traffic to a domain and its subdomains. In AWS, you pay 0.50$ per month per hosted zone.

Two types of hosted zones:
- **Public Hosted Zones**: are for domains that are public and available over the internet. They contain records that specify how to route traffic on the internet (public domain names). For example, application1.mypublicdomain.com.

![Public Hosted Zone](/assets/aws-certified-developer-associate/public_hosted_zone.png "Public Hosted Zone")

- **Private Hosted Zones**: are for domains that are private and available only within your VPC. They contain records that specify how you route traffic within one or more VPCs (private domain names). For example, [application1.company.internal](application1.company.internal).

![Private Hosted Zone](/assets/aws-certified-developer-associate/private_hosted_zone.png "Private Hosted Zone")

## 6.3 Registering a Domain Name with Route 53

To register a domain name with Route 53, access the Route 53 console, and choose *Registered Domains*. Then, click **Register Domains**:

![Register Domain](/assets/aws-certified-developer-associate/route53_register_domain.png "Register Domain")

Next, enter the **domain name** you want to register:

![Enter Domain Name](/assets/aws-certified-developer-associate/route53_enter_domain_name.png "Enter Domain Name")

An then, select it and check out:

![Select Domain](/assets/aws-certified-developer-associate/route53_select_domain.png "Select Domain")

Configure **duration** for pricing and **auto-renew**:

![Configure Pricing](/assets/aws-certified-developer-associate/route53_configure_pricing.png "Configure Pricing")

Enter the **contact information** for registrant, administrative, and technical contacts with also the option to enable **privacy protection** to hide contact information from WHOIS queries:

![Contact Information](/assets/aws-certified-developer-associate/route53_contact_information.png "Contact Information")

Finally, create the domain and it can be accessed ion the *Registered Domains* section. If you click on it, you can see its details and the **records** associated with it. Upon creation, you will see a *NS* record which is the name servers (AWS servers) for the hosted zone and a *SOA* record.

## 6.4 Creating Records in Route 53

To create a record in Route 53, access the Route 53 console, and choose *Hosted Zones*. Then, click on the hosted zone you want to create a record for:

![Hosted Zone](/assets/aws-certified-developer-associate/route53_hosted_zone.png "Hosted Zone")

And click on **Create Record**:

![Create Record](/assets/aws-certified-developer-associate/route53_create_record.png "Create Record")

Enter the **record name**, **record type** (A, AAAA, ...), **value** (the IP address), **routing policy** (will see later), and **TTL**:

![Enter Record](/assets/aws-certified-developer-associate/route53_enter_record.png "Enter Record")

Finally, create the record and it will be listed in the hosted zone.

You can test the created record on your terminal or AWS CloudShell. For example, to test the record, you can use either of the following commands from the `bind-utils` package:

```bash
nslookup example.com

dig example.com
```

## 6.5 Route 53 Records TTL

The **TTL allows you to control how long the DNS resolvers cache the records provided in response to DNS queries**. The TTL is set in seconds and the default value is 300 seconds. This is useful because we do not expect records to change a lot, so we do not want to query the DNS server too often.

![TTL](/assets/aws-certified-developer-associate/route53_ttl.png "TTL")

- High TTL – e.g., 24 hours.
    - Less traffic on Route 53 because less clients are making queries as results are cached.
    - Possibly outdated records as you have to wait for the TTL to expire.

- Low TTL – e.g., 60 seconds.
    - More traffic on Route 53, so it will cost you more.
    - Records are outdated for less time, so it is easier to change records.

A strategy is to **lower the TTL before a change** and then increase it again after the record change to reduce the traffic on Route 53.

**TTL is mandatory for each DNS record, except for the Alias record**.

You can use the `nslookup` or `dig` command to check the TTL of a record, as well as checking if the record is up to date or not.

## 6.6 Route 53 CNAME vs Alias Records

You will use these records to **map a hostname to another hostname**. For example, you have an AWS resource (e.g., ELB, CloudFront, ...) that exposes an AWS hostname like lb1-1234.us-east-2.elb.amazonaws.com but you want myapp.mydomain.com.

A **CNAME** record:
- Points a hostname to any other hostname. For example, app.mydomain.com to blabla.anything.com.
- Works **only for non-root domain**. For example, it works for www.mydomain.com or demo.mydomain.com but not for mydomain.com.

An **Alias** record:
- Points a hostname to an AWS resource. For example, app.mydomain.com to resource.amazonaws.com.
- Works for both root and non-root domains. For example, it works for www.mydomain.com and demo.mydomain.com, and also for mydomain.com.
- Is free of charge.
- Offers native health checks.

### 6.6.1 Alias Records

They are an extension to DNS functionality to **map a hostname to an AWS resource**.

![Alias Record](/assets/aws-certified-developer-associate/route53_alias_record.png "Alias Record")

Alias records **automatically recognize changes in the resource's IP addresses**.

Alias records are **always of type A/AAAA for AWS resources (IPv4/IPv6)** and you cannot set a TTL for them (it will be assigned by Route 53).

Unlike CNAME, it **can be used for the top node of a DNS namespace (called the Zone Apex)**, e.g.: example.com.

A target for an alias record can be:
- Elastic Load Balancers.
- CloudFront distributions.
- API Gateway.
- Elastic Beanstalk environments.
- S3 Websites.
- VPC Interface Endpoints.
- Global Accelerator accelerator.
- Route 53 record in the same hosted zone.

**A target for an alias record cannot be an EC2 DNS name**.

## 6.7 Creating CNAME and Alias Records

To create a CNAME or Alias record in Route 53, access the Route 53 console, and choose *Hosted Zones*. Then, click on the hosted zone you want to create a record.

To **create a CNAME record**, click on *Create Record* and enter **record type as CNAME**.

![Create CNAME](/assets/aws-certified-developer-associate/route53_create_cname.png "Create CNAME")

To **create an Alias record**, click on *Create Record* and enter **record type** as A or AAAA. Then, **enable the Alias attribute** and select the type of **Route traffic to** endpoint in the list:

![Create Alias](/assets/aws-certified-developer-associate/route53_create_alias.png "Create Alias")

Then select the region for it and **select the resource you want to route traffic to**, a load balancer in the image below:

![Select Resource](/assets/aws-certified-developer-associate/route53_select_resource.png "Select Resource")

You can also configure a **routing policy** for the record and enable/disable **health checks** (because this is an Alias record):

![Health Checks](/assets/aws-certified-developer-associate/route53_health_checks.png "Health Checks")

You can follow this exact process to create an Alias record to route traffic to a desired AWS resource when users hit the Zone Apex.

## 6.8 Route 53 Routing Policies

Routing policies define how Route 53 responds to DNS queries.

Route 53 Supports the following Routing Policies
- Simple.
- Weighted.
- Latency-based.
- Failover.
- Geolocation.
- Geoproximity (using Route 53 Traffic Flow feature).
- IP-based.
- Multi-Value Answer.

### 6.8.1 Simple Routing Policy

Typically used to **route traffic to a single resource** but you **can specify multiple values** in the same record. If multiple values are returned, a **random one is chosen by the client** (it is a client choice which one to use).

![Simple Routing](/assets/aws-certified-developer-associate/route53_simple_routing.png "Simple Routing")

When Alias is enabled, you can specify only one AWS resource. 

You cannot associate health checks to it.

**Configuring it** is really simple, you just need to create a record and leave the routing policy to the value **Simple routing**.

### 6.8.2 Weighted Routing Policy

They allow you to **control the percentage of the requests that go to each resource**.

You **assign each record a relative weight**, so that **the traffic received by each resource is proportional to the weight assigned to the record**. Weights do not need to sum up to 100.

![Weighted Routing](/assets/aws-certified-developer-associate/route53_weighted_routing.png "Weighted Routing")

DNS records must have the same name and type.

They can be associated with health checks.

Use cases: load balancing between regions, testing new versions of software, A/B testing, etc.

**Assign a weight of 0 to a record to stop sending traffic to a resource**.

**If all records have weight of 0, then all records will be returned equally**.

To **create a weighted routing policy**, create a record and set the routing policy to **Weighted routing**. Then, create multiple records and set the **weight** for them:

First record:

![Weighted Record 1](/assets/aws-certified-developer-associate/route53_first_weighted_record.png "Weighted Record 1")

To add the second record, click on *Add another record*:

![Weighted Record 2](/assets/aws-certified-developer-associate/route53_second_weighted_record.png "Weighted Record 2")

### 6.8.3 Latency-based Routing Policy

With latency-based routing, you can route your traffic based on the **lowest network latency for your end user**. Latency is the time taken for the DNS query to resolve to the IP address and is based on traffic between users and AWS regions.

So, you can redirect to the resource that has the least latency close to your users. This is helpful when latency for users is a priority/concern. For example, Germany users may be redirected to the North Virginia region if it has the lowest latency.

They can be associated with health checks and have a **failover capability**.

![Latency Routing](/assets/aws-certified-developer-associate/route53_latency_routing.png "Latency Routing")

To **create a latency-based routing policy**, create a record and set the routing policy to **Latency routing**. Then, create multiple records and set the **region** for them.

First record:

![Latency Record 1](/assets/aws-certified-developer-associate/route53_first_latency_record.png "Latency Record 1")

Then, click on *Add another record* to add the second record:

![Latency Record 2](/assets/aws-certified-developer-associate/route53_second_latency_record.png "Latency Record 2")

To test these records, you can use a VPN to change your location and check the resource that responds.

### 6.8.4 Failover Routing Policy (Active-Passive Setup)

These are used when you want to create an **active/passive setup**. For example, you have a primary instance in EU-West-1 and a secondary DR site in EU-Central-1. If the primary instance is healthy, Route 53 will route traffic to it. If the primary instance is unhealthy, Route 53 will perform an **automatic failover** and route traffic to the secondary instance.

![Failover Routing](/assets/aws-certified-developer-associate/route53_failover_routing.png "Failover Routing")

To **create a failover routing policy**, create a record and set the routing policy to **Failover routing**. Then, **create 2 records, one primary and one secondary, by setting the failover record type** for them:

![Failover Record](/assets/aws-certified-developer-associate/route53_failover_record.png "Failover Record")

To test these records, you can block the primary instance (for example by making it inaccessible to health checkers) to simulate a failure and check if the secondary instance responds.

### 6.8.5 Geolocation Routing Policy

This routing is **based on user location**, i.e., the location of the user making the DNS query. Specify location by Continent, Country or by US State (if there is overlapping, most precise location selected).

You should create a **Default record in case there is no match on location**.

For example, users in Germany will be sent to an instance with IP 11.22.33.44, and users in the France will be sent to an instance with IP 55.66.77.88. All other users will be sent to the default record at 99.11.22.33.

![Geolocation Routing](/assets/aws-certified-developer-associate/route53_geolocation_routing.png "Geolocation Routing")

They can be associated with health checks.

Use cases: website localization, restrict content distribution, load balancing, etc.

To **create a geolocation routing policy**, create a record and set the routing policy to **Geolocation routing**. Then, create multiple records and set the **location (or Default)** for them:

![Geolocation Record](/assets/aws-certified-developer-associate/route53_geolocation_record.png "Geolocation Record")

To test these records, you can use a VPN to change your location and check the resource that responds.

### 6.8.6 Geoproximity Routing Policy

Route traffic to your resources based on the geographic location of users and resources.

With this policy, you have the **ability to shift more traffic to resources in one region by setting the bias**.

To **change the size of the geographic region, specify bias values**:
- To expand (1 to 99): more traffic to the resource.
- To shrink (-1 to -99): less traffic to the resource.

Resources can be:
- AWS resources (specify AWS region).
- Non-AWS resources (specify Latitude and Longitude).

You **must use Route 53 Traffic Flow to use this feature and leverage the bias**.

Consider the **two following examples with two regions**. In the first example, the two regions both have bias set to 0:

![Geoproximity Routing Same Bias](/assets/aws-certified-developer-associate/route53_geoproximity_routing_same_bias.png "Geoproximity Routing Same Bias")

In the second example, the two regions have different bias values:

![Geoproximity Routing Different Bias](/assets/aws-certified-developer-associate/route53_geoproximity_routing_different_bias.png "Geoproximity Routing Different Bias")

### 6.8.7 IP-based Routing Policy

**Route traffic based on the IP address of the client making the request**. You can specify the IP address ranges (in CIDR format) and the location to route the traffic to.

Use cases: optimize performance, reduce network costs, etc.

![IP-based Routing](/assets/aws-certified-developer-associate/route53_ip_based_routing.png "IP-based Routing")

### 6.8.8 Multi-Value Answer Routing Policy

**Used to route traffic to multiple resources** because Route 53 return multiple values/resources.

They can be associated with health checks, so that they return only values for healthy resources.

**Up to 8 healthy records are returned for each multi-value query** and then the client will choose which one to use among them. Thanks to health checks, you can ensure that only healthy resources are returned, so **it is safer than the simple routing policy with multiple values because in simple routing, all values are returned as there is no health checks**.

Multi-value is not a substitute for having an ELB, it is **client-side load balancing**.

To **create a multi-value answer routing policy**, create a record and set the routing policy to **Multivalue answer**. Then, create multiple records and set the **value** for them:

![Multi-Value Record](/assets/aws-certified-developer-associate/route53_multi_value_record.png "Multi-Value Record")

To test these records, you can use the `nslookup` or `dig` command to check the values returned by the DNS query and see that there are multiple values returned, as many as the number of records you created. However, if you force the health check to fail for one of the records, you will see that it is not returned and will have less records because only the healthy ones are returned.

## 6.9 Route 53 Health Checks

HTTP Health Checks are a way to **check the status for public resources (only)**.

Health checks provide Automated DNS Failover. Three types of health checks:
1. Health checks that **monitor an endpoint** (application, server, other AWS resource)
2. Health checks that monitor other health checks (**Calculated Health Checks**)
3. Health checks that **monitor CloudWatch Alarms** (you get full control). For example, you can check throttles of DynamoDB, alarms on RDS, custom metrics. This is particularly helpful for private resources.

Health checks are integrated with CloudWatch Metrics because they have their own metrics that you can see there.

![Route53 Health Checks diagram](/assets/aws-certified-developer-associate/route53_health_checks_diagram.png "Route53 Health Checks diagram")

### 6.9.1 Monitor an Endpoint

About **15 global health checkers** will check the endpoint health.
- **Healthy/Unhealthy Threshold**, which is 3 by default, and indecates how many health checkers need to report the endpoint as unhealthy to consider it unhealthy.
- **Interval**: by default 30 seconds but you can set to 10 seconds (higher cost).
- Supported protocols: HTTP, HTTPS and TCP.
- If **>18%** of health checkers report the endpoint is healthy, Route 53 considers it Healthy. Otherwise, it is deemed Unhealthy.
- Ability to choose which locations you want Route 53 to use.

You need to configure your router/firewall to **allow incoming requests from Route 53 health checkers**. The IP ranges can be found at [https://ip-ranges.amazonaws.com/ip-ranges.json](https://ip-ranges.amazonaws.com/ip-ranges.json).

![Monitor an Endpoint](/assets/aws-certified-developer-associate/route53_monitor_endpoint.png "Monitor an Endpoint")

Health Checks pass only when the endpoint returns a 2xx or 3xx status code and can also be setup to pass/fail based on the text in the first 5120 bytes of the response.

### 6.9.2 Calculated Health Checks

**Combine the results of multiple health checks into a single health check**:

![Calculated Health Checks](/assets/aws-certified-developer-associate/route53_calculated_health_checks.png "Calculated Health Checks")

You can use OR, AND, or NOT to combine and monitor **up to 256 child health checks**. You need to specify how many of the health checks need to pass to make the parent health check pass.

Use case: perform maintenance to your website without causing all health checks to fail.

### 6.9.3 CloudWatch Alarm Health Checks

Route 53 health checkers are outside the VPC, so they cannot access private endpoints (private VPC or on-premises resource). 

**You can create a CloudWatch Metric and associate a CloudWatch Alarm, then create a health check that checks the alarm itself**. So, if the metric is breached, the alarm will go off and the health check will fail.

![CloudWatch Alarm Health Checks](/assets/aws-certified-developer-associate/route53_cloudwatch_alarm_health_checks.png "CloudWatch Alarm Health Checks")

## 6.10 Creating Route 53 Health Checks

To create a health check in Route 53, access the Route 53 console, and choose *Health checks*. Then, click on *Create health check* and choose the **type of health check** you want to create:

![Create Health Check](/assets/aws-certified-developer-associate/route53_create_health_check.png "Create Health Check")

### 6.10.1 Creating a Monitor an Endpoint Health Check

To create a monitor an endpoint health check, you need to select the type of health check as *Endpoint* (like in the previous image) and then configure the endpoint that you want to monitor:

![Health Check Configuration](/assets/aws-certified-developer-associate/route53_health_check_configuration.png "Health Check Configuration")

There are also some advanced options that you can configure for the health check (e.g., interval, failure threshold, look for string in the response's first 5120 bytes, regions, ...):

![Advanced Options](/assets/aws-certified-developer-associate/route53_advanced_options.png "Advanced Options")

Before creation, you can also set a CloudWatch alarm for the health check to get notified when it fails:

![CloudWatch Alarm](/assets/aws-certified-developer-associate/route53_cloudwatch_alarm.png "CloudWatch Alarm")

To test the health check, you can block inbound traffic to the port configured in the health check (in this case, port 80) to force the health check to fail. For example, this is how health checks look like in the dashboard:

![Health Check Status](/assets/aws-certified-developer-associate/route53_health_check_status.png "Health Check Status")

From the **Health Checkers** tab, you can see what health checkers are doing and also look for failed checks.

### 6.10.2 Creating a Calculated Health Check

To create a calculated health check, you need to select the type of health check as *Calculated health check* and then configure the child health checks that you want to combine:

![Create Calculated Health Check](/assets/aws-certified-developer-associate/route53_create_calculated_health_check.png "Create Calculated Health Check")

To configure the child health checks, you need to select the health checks that you want to combine and the **combination method** (AND, OR, or NOT):

![Configure Child Health Checks](/assets/aws-certified-developer-associate/route53_configure_child_health_checks.png "Configure Child Health Checks")

You can set the alarm to get the notification when the calculated health check fails, just like the previous health check.

### 6.10.3 Creating a CloudWatch Alarm Health Check

To create a CloudWatch alarm health check, you need to select the type of health check as *State of CloudWatch alarm* and then configure the CloudWatch alarm that you want to monitor:

![Create CloudWatch Alarm Health Check](/assets/aws-certified-developer-associate/route53_create_cloudwatch_alarm_health_check.png "Create CloudWatch Alarm Health Check")

## 6.11 Route 53 Traffic Flow

Route 53 Traffic Flow is a **visual editor for managing complex routing configurations**. It simplifies the process of creating and maintaining records in large and complex configurations.

**Configurations can be saved as Traffic Flow Policy** and can be applied to different Route 53 Hosted Zones (different domain names) with **support for versioning**.

![Route 53 Traffic Flow](/assets/aws-certified-developer-associate/route53_traffic_flow.png "Route 53 Traffic Flow")

To create a traffic flow policy, access the Route 53 console, and choose *Traffic Flow*. Then, click on *Create Traffic Flow* and give a name to the policy.

So, you get a **start point** where you specify the type of record you want to create (e.g., A, AAAA, CNAME, ...):

![Traffic Flow Start](/assets/aws-certified-developer-associate/route53_traffic_flow_start.png "Traffic Flow Start")

Then, you can connect it to rules or endpoint:

![Traffic Flow Rules](/assets/aws-certified-developer-associate/route53_traffic_flow_rules.png "Traffic Flow Rules")

For example, you can connect the record to an endpoint:

![Traffic Flow Endpoint](/assets/aws-certified-developer-associate/route53_traffic_flow_endpoint.png "Traffic Flow Endpoint")

Or you can connect it to a rule (in this case, a weighted rule):

![Traffic Flow Weighted Rule](/assets/aws-certified-developer-associate/route53_traffic_flow_weighted_rule.png "Traffic Flow Weighted Rule")

Which you can then connect to other rules or endpoints.

### 6.11.1 Traffic Flow for Geoproximity Rules

The **Traffic Flow UI makes it a lot easier to create and understand geoproximity routing policies**, in particular. For example, you can configure the regions and the biases and you will get a visual representation of the routing policy like the following:

![Traffic Flow Geoproximity](/assets/aws-certified-developer-associate/route53_traffic_flow_geoproximity.png "Traffic Flow Geoproximity")

And by changing the bias as you need, you can adjust the way the traffic is routed:

![Traffic Flow Geoproximity Bias](/assets/aws-certified-developer-associate/route53_traffic_flow_geoproximity_bias.png "Traffic Flow Geoproximity Bias")

You can add as many regions as you need and adjust the biases as you need:

![Traffic Flow Geoproximity Multiple Regions](/assets/aws-certified-developer-associate/route53_traffic_flow_geoproximity_multiple_regions.png "Traffic Flow Geoproximity Multiple Regions")

When you are done, you can create the traffic flow policy and **apply it to an hosted zone with a policy record DNS name** (you can also see the pricing for the policy):

![Traffic Flow Apply Policy](/assets/aws-certified-developer-associate/route53_traffic_flow_apply_policy.png "Traffic Flow Apply Policy")

## 6.12 Route 53 (External) Domain Registrar and DNS Service

So far, we have seen how to create records in Route 53 for domains that are registered with Route 53. However, you can also use Route 53 as a DNS service for domains that are registered with other domain registrars.

You buy or register your domain name with a domain registrar, typically by paying annual charges (e.g., GoDaddy, Amazon Registrar Inc., ...). The domain registrar usually also provides you with a DNS service to manage your DNS records but you can use another DNS service to manage your DNS records.

**Scenario**: you purchase the domain from GoDaddy and use Route 53 to manage your DNS records.

![Route 53 External Domain Registrar](/assets/aws-certified-developer-associate/route53_external_domain_registrar.png "Route 53 External Domain Registrar")

To **use Route 53 as a DNS service for a domain that is registered with another domain registrar**, you need to follow these steps:
1. **Create a Hosted Zone in Route 53**.
2. **Update the NS records in the domain registrar's console** to point to the Route 53 name servers.
3. **Manage records direclty in Route 53**.

![Route 53 External Domain Registrar Steps](/assets/aws-certified-developer-associate/route53_external_domain_registrar_steps.png "Route 53 External Domain Registrar Steps")

The crucial point to understand here is that **domain regitrar != DNS service**, even though every domain registrar usually comes with some DNS features.