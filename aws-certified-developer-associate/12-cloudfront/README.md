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

## 12.4 CloudFront Caching

The **cache lives at each CloudFront Edge Location, so you have as many caches as edge locations**.

![CloudFront Cache Key](/assets/aws-certified-developer-associate/cloudfront_caching.png "CloudFront Cache Key")

You want to **maximize the cache hit ratio to minimize requests to the origin**.

You can **invalidate part of the cache using the CreateInvalidation API to avoid waiting for the cache to expire based on the TTL**.

### 12.4.1 Cache Key

**CloudFront identifies each object in the cache using a cache key**: a unique identifier for every object in the cache. By default, consists of *hostname + resource portion of the URL*.
- For example, if you request http://www.example.com/image.jpg, the cache key would be `www.example.com/image.jpg` because the hostname is `www.example.com` and the resource is `/image.jpg`.

![CloudFront Cache Key](/assets/aws-certified-developer-associate/cloudfront_cache_key.png "CloudFront Cache Key")

If you have an application that serves up content that varies based on parameters (e.g., user, device, language, location, ...), you can **add other elements (HTTP headers, cookies, query strings) to the cache key using CloudFront Cache Policies**.

## 12.5 Cache Policies

Cache policies allow you to **customize the cache key**. You can **add additional elements to the cache key** to serve up different versions of the same object based on the request.

You can **cache based on**:
- **HTTP headers**: None | Whitelist.
- **Cookies**: None | Whitelist | Include All-Except | All.
- **Query strings**: None | Whitelist | Include All-Except | All.

Note: **all HTTP headers, cookies, and query strings that you include in the cache key are automatically included in origin requests**.

**Control the TTL (from 0 seconds to 1 year)**. The TTL can also be set by the origin using specific headers like `Cache-Control` header, `Expires` header, etc.

You can **create your own policy or use Predefined Managed Policies**, which are offered by AWS.

### 12.5.1 Caching with HTTP Headers

Caching with HTTP headers example:
- **None**:
    - None of the headers are included in the cache key (except default).
    - Headers are not forwarded (except default).
    - **Best caching performance**.
- **Whitelist**:
    - **Only specified headers are included in the cache key**.
    - Specified headers are also forwarded to the origin.

![CloudFront HTTP Headers](/assets/aws-certified-developer-associate/cloudfront_http_headers.png "CloudFront HTTP Headers")

### 12.5.2 Caching with Query Strings

Caching with query strings example:
- **None**:
    - None of the query strings are included in the cache key.
    - Query strings are not forwarded.
- **Whitelist**:
    - Only specified query strings are included in the cache key.
    - Only specified query strings are forwarded.
- **Include All-Except**:
    - All query strings are included in the cache key except the ones specified in the list.
    - All query strings are forwarded except the ones specified in the list.
- **All**:
    - All query strings are included in the cache key.
    - All query strings are forwarded.
    - **Worst caching performance** (e.g., when you have many query strings).

![CloudFront Query Strings](/assets/aws-certified-developer-associate/cloudfront_query_strings.png "CloudFront Query Strings")

## 12.6 Origin Request Policies

**Specify values that you want to include in origin requests without including them in the cache key** (no duplicated cached content). With origin request, we mean the request that CloudFront makes to the origin to get the content in case of a cache miss.

This feature provides the ability to add CloudFront HTTP headers and custom headers to an origin request, even though they were not included in the viewer request. For example, API keys.

You **can include**:
- **HTTP headers**: None | Whitelist | All viewer headers options.
- **Cookies**: None | Whitelist | All.
- **Query strings**: None | Whitelist | All.

You can create your own policy or use Predefined Managed Policies, which are offered by AWS.

The **benefit of this feature** is that, if you have a cache policy that includes only some HTTP headers, you can use the origin request policy to include the missing headers in the origin request. For example, the image below shows a *cache policy* that includes only the `Authorization` header in the cache key, but the origin needs also a `session_id` cookie, a `User-Agent` header (alongside the `Authorization` header), and a `ref` query param to handle the request. So, you can use the *origin request policy* to include these values in the request to the origin:

![CloudFront Origin Request Policies](/assets/aws-certified-developer-associate/cloudfront_origin_request_policies.png "CloudFront Origin Request Policies")

## 12.7 Cache Invalidation

In case you update the back-end origin, CloudFront does not know about it and will only get the refreshed content after the TTL has expired. However, you can **force an entire or partial cache refresh** (thus bypassing the TTL) **by performing a CloudFront invalidation**.

You can invalidate all files (`*`) or a special path (e.g., `/images/*`).

The following image shows an example of invalidation, where we are invalidating the `index.html` file and the `/images/*` path coming from an S3 bucket (origin of the distribution):

![CloudFront Invalidation](/assets/aws-certified-developer-associate/cloudfront_invalidation.png "CloudFront Invalidation")

After an invalidation is triggered, the edge locations will get the new content from the origin upon the next request.

### 12.7.1 Using Cache Invalidation

To **perform a cache invalidation**, you need to go to the CloudFront service and select the distribution you want to do it for. Then, go to the *Invalidations* tab and click on the *Create Invalidation* button.

Enter the object paths you want to invalidate (e.g., `index.html` or `/images/*`) and confirm:

![CloudFront Invalidation Creation](/assets/aws-certified-developer-associate/cloudfront_invalidation_creation.png "CloudFront Invalidation Creation")

The invalidation in the image above has an object path of `/*`, which means that all the objects in the cache will be invalidated.

The invalidation process may take some time to complete.

## 12.8 Cache Behaviors

**Configure different settings for a given URL path pattern**. For example: one specific cache behavior to `images/*.jpg` files on your origin web server.

**Route to different kind of origins/origin groups based on the content type or path pattern**:
- `/images/*`.
- `/api/*`.
- `/*` (default cache behavior).

![CloudFront Cache Behaviors](/assets/aws-certified-developer-associate/cloudfront_cache_behaviors.png "CloudFront Cache Behaviors")

When adding additional cache behaviors, the **default cache behavior (which is always `/*`) is always the last to be processed and works as a fallback** if no other cache behavior matches.

### 12.8.1 Cache Behavior for Sign In Page

In this example, we want users hitting the `/login` URL to go to the origin (e.g., an EC2 instance) to be able to sign in and get a cookie in the response that they can use to access all other things in the application, which can be cached (default behavior).

![CloudFront Sign In Page](/assets/aws-certified-developer-associate/cloudfront_sign_in_page.png "CloudFront Sign In Page")

### 12.8.2 Cache Behavior to Maximize Cache Hits by Separating Static and Dynamic Distributions

In this example, we have a website with static content that can be cached for all users (no rule needed) and dynamic content that is user-specific, so needs to be cached based on the correct HTTP headers and cookies.

![CloudFront Maximize Cache Hits](/assets/aws-certified-developer-associate/cloudfront_maximize_cache_hits.png "CloudFront Maximize Cache Hits")

## 12.9 Access and Create Cache Behaviors

To access cache behaviors, you need to go to the **CloudFront** service and select the distribution you want to work with. Then, go to the *Behaviors* tab and you will get a list of the cache behaviors that are already configured for the distribution. In the image below, only the default cache behavior is configured:

![CloudFront Behaviors List](/assets/aws-certified-developer-associate/cloudfront_behaviors_list.png "CloudFront Behaviors List")

If you access its details, you can see the settings of the default cache behavior, among which the **cache policy** and the **origin request policy**:

![CloudFront Default Behavior Settings](/assets/aws-certified-developer-associate/cloudfront_default_behavior.png "CloudFront Default Behavior Settings")

### 12.9.1 Creating a Cache Behavior

To create a cache behavior, you need to click on the *Create Behavior* button. You will be redirected to the cache behavior creation page.

In the creation page, you need to add the **path pattern** (e.g., `/images/*`), **origin/origin group**, **compression**, a **viewer protocol policy**. and **allowed HTTP methoods** (*GET, HEAD* | *GET, HEAD, OPTIONS* | *GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE*).

![CloudFront Cache Behavior Creation](/assets/aws-certified-developer-associate/cloudfront_cache_behavior_creation.png "CloudFront Cache Behavior Creation")

Then, you can **add cache policies and origin request policies** to the cache behavior:

![CloudFront Behavior Add Policies](/assets/aws-certified-developer-associate/cloudfront_default_behavior.png "CloudFront Behavior Add Policies")

You can also create a cache policy or an origin request policy from this page using the dedicated buttons.

### 12.9.2 Creating a Cache Policy from the Cache Behavior

To create a cache policy, you need to click on the *Create Policy* button below the cache policy field. You will be redirected to the cache policy creation page.

For the **cache policy creation**, you need to add some information like **name** and description, and the settings for the **TTL**:

![CloudFront Cache Policy Creation](/assets/aws-certified-developer-associate/cloudfront_cache_policy_creation.png "CloudFront Cache Policy Creation")

Next are the **cache key settings**, where you can add elements to the cache key (headers, custom headers, etc.) with the options indicated previously:

![CloudFront Cache Policy Cache Key](/assets/aws-certified-developer-associate/cloudfront_cache_policy_cache_key.png "CloudFront Cache Policy Cache Key")

Finally, you can set the **compression support** for the cache policy:

![CloudFront Cache Policy Compression](/assets/aws-certified-developer-associate/cloudfront_cache_policy_compression.png "CloudFront Cache Policy Compression")

### 12.9.3 Creating an Origin Request Policy from the Cache Behavior

From the cache behavior, you can also create an origin request policy using the *Create Policy* button below the origin request policy field. The process is similar to the cache policy creation.

For the **origin request policy creation**, you have to add some information like **name** and description:

![CloudFront Origin Request Policy Creation](/assets/aws-certified-developer-associate/cloudfront_origin_request_policy_creation.png "CloudFront Origin Request Policy Creation")

And the set the **origin request settings**, where you can add elements to the origin request (headers, custom headers, etc.) with the options indicated previously:

![CloudFront Origin Request Policy Origin Request](/assets/aws-certified-developer-associate/cloudfront_origin_request_policy_origin_request.png "CloudFront Origin Request Policy Origin Request")

## 12.10 ALB or EC2 as Origins


### 12.10.1 EC2 as Origin

You can expose back-end applications running on EC2 instances through CloudFront, so that users can access them through edge locations.

To do so, your EC2 instances must be **publicly accessible** and the **security group must allow inbound traffic from CloudFront edge locations' IP addresses**.

![CloudFront EC2](/assets/aws-certified-developer-associate/cloudfront_ec2.png "CloudFront EC2")

### 12.10.2 ALB as Origin

You can also use an ALB as an origin for CloudFront. This is useful when you have an ALB with multiple EC2 instances behind it, which **remain private** because you can have private VPC connection between the ALB and EC2 instances.

To do so, the ALB must be **publicly accessible** and the **security group must allow inbound traffic from CloudFront edge locations' IP addresses**.

![CloudFront ALB](/assets/aws-certified-developer-associate/cloudfront_alb.png "CloudFront ALB")

## 12.11 CloudFront Geo Restriction

You can **restrict who can access your distribution** via:
- **Allow list**: Allow users to access your content only if they are re in one of the countries on a list of approved countries.
- **Block list**: Prevent users from accessing your content if they are in one of the countries on a list of banned countries.

The country is determined using a 3rd party Geo-IP database.

**Use case**: Copyright Laws to control access to content.

### 12.11.1 Turn on Geo Restriction

To turn on Geo Restriction, go to the CloudFront service and select the distribution you want to work with. Then, go to the *Security* tab and find the *CloudFront Geographic Restrictions* settings.

![CloudFront Geo Restriction](/assets/aws-certified-developer-associate/cloudfront_geo_restriction.png "CloudFront Geo Restriction")

Click on the *Edit* button to change the settings. You can choose between *No restrictions* (in case, no need to select countries), *Allow list*, and *Black list* and then select the countries you want to allow or block:

![CloudFront Geo Restriction Edit](/assets/aws-certified-developer-associate/cloudfront_geo_restriction_edit.png "CloudFront Geo Restriction Edit")

Then, save and see how it appears in the distribution settings:

![CloudFront Geo Restriction Enabled](/assets/aws-certified-developer-associate/cloudfront_geo_restriction_enabled.png "CloudFront Geo Restriction Enabled")

## 12.12 CloudFront Signed URLs and Signed Cookies

You can use CloudFront Signed URL/Cookie to distribute paid shared content to premium users over the world:

You attach a policy with:
- **URL expiration**.
- **IP ranges to access the data from**.
- **Trusted signers**: which AWS accounts can create signed URLs.

How long should the URL be valid for?
- Shared content (movie, music): make it short (a few minutes).
- Private content (private to the user): you can make it last for years.

**Signed URL vs Signed Cookies**:
- Signed URLs provide access to individual files: **one signed URL per file**.
- Signed Cookies provide access to multiple files: **one signed cookie for many files**.

### 12.12.1 CloudFront Signed URLs vs S3 Pre-Signed URLs

**CloudFront Signed URLs**:
- **Allow access to a path, no matter the origin**: it can be an S3 bucket, an EC2 instance, an ALB, etc.
- **Account wide key-pair**: any IAM user with sufficient permissions can create keys and key groups (signers of the URLs).
- Can **filter by IP, path, date, expiration**.
- Can **leverage caching features** of CloudFront.

**S3 Pre-Signed URLs**:
- Issue a **request as the person who pre-signed the URL**.
- Uses the **IAM key of the signing IAM principal**.
- **Limited lifetime**.

S3 pre-signed URLs are more specific because they are limited to S3, while CloudFront signed URLs are more flexible because they can be used to access content from any origin.

### 12.12.2 CloudFront Signed URL Workflow

We have an S3 bucket that is configured to be the origin of a CloudFront distribution. The content of the bucket is private and can be accessed only by CloudFront because of OAC. We want to provide access to a file in the bucket to a user through CloudFront.

The user authenticates with the application and requests access to a file. The application checks the user's credentials and generates a CloudFront signed URL for the file. The signed URL is sent to the user, who can use it to access the file through CloudFront.

![CloudFront Signed URL](/assets/aws-certified-developer-associate/cloudfront_signed_url.png "CloudFront Signed URL")

It **works in the same way for signed cookies**.

## 12.13 CloudFront Signed URL Process

**Two types of signers**:
- **Trusted key group (recommended)**: can leverage APIs to create and rotate keys, and IAM for API security.
- **AWS account that contains a CloudFront Key Pair (not recommended)**: need to manage keys using the root account and the AWS console, so it is **not recommended** because:
    - You should not use the root account for this.
    - Not possible to automate like you can with key groups, which can be done via APIs.

### 12.13.1 CloudFront Trusted Key Groups

In your CloudFront distribution **create one or more trusted key groups**.

The, **generate your own public and private key**:
- The private key is used by your applications (e.g., EC2) to sign URLs.
- The public key (uploaded) is used by CloudFront to verify URLs.

### 12.13.2 Adding a Key Pair to Use in a Key Group

Go to the CloudFront service and find the *Key Management* entry in the menu. Then, go into *Public Keys* to add a new key pair that you can generate separately (e.g., locally).

Following image is an example of a key pair (2048 bit key size):

![Generated Key Pair](/assets/aws-certified-developer-associate/generated_pey_pair.png "Generated Key Pair")

So, click on the *Create Public Key* button and add the generated public key:

![CloudFront Add Public Key](/assets/aws-certified-developer-associate/cloudfront_add_public_key.png "CloudFront Add Public Key")

### 12.13.3 Creating a Key Group

Go to the CloudFront service and find the *Key Management* entry in the menu. Then, go into *Key Groups* and click on the *Create Key Group* button.

![CloudFront Key Group Console](/assets/aws-certified-developer-associate/cloudfront_key_group_console.png "CloudFront Key Group Console")

Insert a **name**, optionally a **description**, and add the **public keys (currently, up to 5)** you want to use in the key group:

![CloudFront Create Key Group](/assets/aws-certified-developer-associate/cloudfront_create_key_group.png "CloudFront Create Key Group")

Finally, create it and you will see it in the list of key groups.

## 12.14 CloudFront Pricing and Price Classes

CloudFront **edge locations are all around the world and because of that the cost of data out per edge location varies**.

![CloudFront Pricing](/assets/aws-certified-developer-associate/cloudfront_pricing.png "CloudFront Pricing")

You can **reduce the number of edge locations for cost reduction using price classes**.

Three price classes:
1. **Price Class All**: all regions to offer the best performance but is the most expensive.
2. **Price Class 200**: most regions, but excludes the most expensive regions.
3. **Price Class 100**: only the least expensive regions.

![CloudFront Price Classes](/assets/aws-certified-developer-associate/cloudfront_price_classes.png "CloudFront Price Classes")