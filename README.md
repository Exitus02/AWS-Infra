# AWS-Infra
Web frontend served via CloudFront (in front of S3) that talks to a Python backend running on Lambda which, too, is served via CloudFront (in front of API Gateway), with data written to and read from a DynamoDB table.

![image](https://user-images.githubusercontent.com/71135302/169729103-5043f28c-270c-4f41-be79-d7fe1dba64f1.png)

•	The client intends to migrate more applications in the future. How will the solution remain intuitive to new developers and scale to their needs?

With this setup all they have to do is create more S3 Buckets for their needs , Amazon Lambda can autoscale based on the database size and traffic.
CloudFront speeds up the distribution of your content by routing each user request through the AWS backbone network to the edge location that can best serve your content. Typically, this is a CloudFront edge server that provides the fastest delivery to the viewer. Using the AWS network dramatically reduces the number of networks that your users' requests must pass through, which improves performance. Users get lower latency—the time it takes to load the first byte of the file—and higher data transfer rates.

You also get increased reliability and availability because copies of your files (also known as objects) are now held (or cached) in multiple edge locations around the world.

•	How can the application be deployed in such a way to scale with demand?

Amazon Lambda
The function continues to scale until the account's concurrency limit for the function's Region is reached. The function catches up to demand, requests subside, and unused instances of the function are stopped after being idle for some time. Unused instances are frozen while they're waiting for requests and don't incur any charges.

When your function scales up, the first request served by each instance is impacted by the time it takes to load and initialize your code. If your initialization code takes a long time, the impact on average and percentile latency can be significant. To enable your function to scale without fluctuations in latency, use provisioned concurrency. The following example shows a function with provisioned concurrency processing a spike in traffic.

•	Encrypting resources at rest and in transit will help the client improve their security position.

The CloudFront API endpoints, cloudfront.amazonaws.com and cloudfront-fips.amazonaws.com, only accept HTTPS traffic. This means that when you send and receive information using the CloudFront API, your data—including distribution configurations, cache policies and origin request policies, key groups and public keys, and function code in CloudFront Functions—is always encrypted in transit. In addition, all requests sent to the CloudFront API endpoints are signed with AWS credentials and logged in AWS CloudTrail.

Encryption at rest
CloudFront uses SSDs which are encrypted for edge location points of presence (POPs), and encrypted EBS volumes for Regional Edge Caches (RECs).

Function code and configuration in CloudFront Functions is always stored in an encrypted format on the encrypted SSDs on the edge location POPs, and in other storage locations used by CloudFront.

•	Enabling services to be self-recovering will reduce staff time spent on-call.

I did not put any self recovery in place with cloudfront and S3 bucket , Amazon Cloudwatch policy implementation should handle that in the future.

Some considerations about hosting a static website with AWS S3 bucket and Cloudfront are below:

Upper cost bound
In the most expensive country - India - CloudFront charges you $0.170 per GB of data transferred. This is huge!

Let's say you have a popular (mainly) Indian website with about 50,000 users visiting your site daily. Also, let's say you make some design changes every week on your site (pretty common for fast iterating products) so you have to invalidate the browser and CloudFront cache.

Also, let's assume on average, a single user downloads about 10MB of the static asset from your site (includes CSS/JS/images/fonts) hosted on S3 proxied through CloudFront.

Let's calculate the cost:

50K Indian users
0.17 USD per GB
10MB per user
Every user retrieves this 4 times a month (you flush your cache 4 times - once every week)
Cost = 50000 * 0.17 * (10/1024) * 4 = 332 USD. That is your COST of just data transfer! I did not calculate the S3 storage cost and the hosting site cost. (I also did not include lambda pricing because it's not much => $(0.20 * (50,000 * 4))/1 million = 4 cents.)

Lower cost bound
In this case, let's assume a US based traffic site. The parameters now would be:

50K USA users
0.085 USD per GB
3 MB per user
Every user retrieves this 4 times a month (you flush your cache 4 times - once every week)
The cost = 50000*0.085*3*4/1024 = 50 USD. That is the lowest you'll pay when using CloudFront with the mentioned traffic (given that all of your 50K users are from the USA only). And remember, that is the cost only for the data transfers! (Not including server costs for hosting your website.)

Alternative
Let's say now you host all these static assets on your main server only - reverse proxied by NGiNX and say, running on a $60 DigitalOcean instance.

Your data transfer per month = 50000 * (10/1024) * 4 = 1952 GB approximately 2TB - DigitalOcean covers your 1TB of transfer per droplet for free. And it is $10 per 1TB from then, so it'll be $70 net for running the server.

Sure, you'll get some latency now - because you're hosting it yourself (we'll even fix this later). NGiNX is a high performing web server and you can rely on it not to be a bottleneck in your static asset delivery.

So you just dropped the cost of "only asset transfer" from $332 to $70 for running the whole server! Bonus tip? We were focusing on running this only in India, so use a DigitalOcean server from India. This would mean less latency.

Not only this, but you can also opt for Cloudflare CDN too - which is FREE. Cloudflare won't respect your files to keep in the CDN if they're too big or too infrequently accessed. But we're assuming a hell of a popular site here, so we should be fine. If not, opt for any other CDN service, and I guarantee you it will be less than $332 a month.

TL;DR - If you're hosting a website with medium-large amounts of traffic with regularly scheduled updates, it is much more cost efficient to host assets yourself and use external CDNs (or even things like DigitalOcean CDN) instead of using S3 and CloudFront (where data traffic rates are through the roof).
