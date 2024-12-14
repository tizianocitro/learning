# 12 CloudFront

It is a **Content Delivery Network (CDN)** that improves read performance because content is cached at the edge locations, closer to the users. As a resulrt, it improves users experience.

CloudFront consists of **216 Point of Presence (or edge locations)** gloabally but they keep increasing.

By having the **content distributed globally, it provides DDoS protection**, also thanks with integration with AWS Shield and AWS Web Application Firewall.

How CloudFront works at an high level:

![CloudFront](/assets/aws-certified-developer-associate/cloudfront.png "CloudFront")

## 12.1 Origins

The **origins of all the files that the CDN will distribute**.

Origins can be:
1. **S3 bucket**:
    - For distributing files and caching them at the edge.
    - Enhanced security with **CloudFront Origin Access Control (OAC)**, which guarantees that only CloudFront can access the files in the S3 bucket.
    - OAC is replacing Origin Access Identity (OAI).
    - **CloudFront can be used as an ingress (to upload files to S3)**.
2. **Custom Origin (HTTP)**:
    - Application Load Balancer.
    - EC2 instance.
    - S3 website (must first enable the bucket as a static website).
    - Any HTTP backend you want.

**CloudFront with S3 as origin**:

![CloudFront-S3](/assets/aws-certified-developer-associate/cloudfront_s3.png "CloudFront-S3")

## 12.2 CloudFront vs S3 Cross Region Replication

**This can be a common question in the exam**.

With **CloudFront** (more static content):
- Leverages a global edge network (216+ locations).
- Files are cached for a TTL (maybe a day).
- **Great for static content that must be available everywhere**.

With **S3 Cross Region Replication** (more dynamic content):
- Must be setup for each region you want replication to happen.
- Files are updated in near real-time.
- It is for read only access.
- **Great for dynamic content that needs to be available at low-latency in few regions**.

## 12.3 Setting Up a CloudFront Distribution

We will create the distribution and configure it to use an S3 bucket as the origin. We will **use CloudFront to distribute the content of the S3 bucket without making the bucket public**.

To createt the distribution, go to the **CloudFront** service and click on **Create a CloudFront Distribution**. No need to select a region because it is a global service.

First, you need to select the **origin** settings:

![CloudFront Origin](/assets/aws-certified-developer-associate/cloudfront_origin.png "CloudFront Origin")

The **origin domain** will be the S3 bucket (but it can be any of the other origins). Then, you need to set the **origin path** to the folder you want to distributea nd the **origin access**, which in the case of S3 can be:

![CloudFront Origin Access](/assets/aws-certified-developer-associate/cloudfront_origin_access.png "CloudFront Origin Access")

As you can see, we are using the **OAC to restrict access to the bucket only to CloudFront (recommended option)**. You can just click on *Create Control Setting* to create the OAC:

![CloudFront OAC](/assets/aws-certified-developer-associate/cloudfront_oac.png "CloudFront OAC")

After creating the OAC, we need to **update the bucket policy to allow CloudFront to access the bucket**. You can do this by going to the bucket, clicking on *Permissions* and then working on the *Bucket Policy*. However, you can also use the one shown in the console (which you can access from the distribution, then *Origins* tab, edit the origin and there you will find the policy):

![CloudFront Bucket Policy](/assets/aws-certified-developer-associate/cloudfront_bucket_policy.png "CloudFront Bucket Policy")

The above policy allows CloudFront to invoke the `GetObject` action on all the objects in the bucket.

You also have some more options to configure the distribution, such as the security protection via **AWS Web Application Firewall (WAF)**, the **price class** (use all edge locations or just the cheaper ones), etc.

![CloudFront Other Options](/assets/aws-certified-developer-associate/cloudfront_other_options.png "CloudFront Other Options")

Finally, set the **default root object** (in this case index.html) and create the distribution:

![CloudFront Root Object](/assets/aws-certified-developer-associate/cloudfront_root_object.png "CloudFront Root Object")

It may take some time to deploy the distribution. Once it is deployed, you can **use the distribution URL (domain name)** and see the content of the S3 bucket. For example, the URL of the distribution in the image below is `d3oijl70vpeijf.cloudfront.net`:

![CloudFront URL](/assets/aws-certified-developer-associate/cloudfront_url.png "CloudFront URL")

To use it to access the `index.html` file in the S3 bucket, you can use the URL `d3oijl70vpeijf.cloudfront.net/index.html`.

One last thing, to manage your origin access, go to the *Origin Access* entry in the *Security* section of the CloudFront dashboard. For example, this is the one associated with the distribution we created:

![CloudFront OAC Console](/assets/aws-certified-developer-associate/cloudfront_oac_console.png "CloudFront OAC Console")