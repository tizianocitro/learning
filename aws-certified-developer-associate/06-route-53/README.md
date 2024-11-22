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
