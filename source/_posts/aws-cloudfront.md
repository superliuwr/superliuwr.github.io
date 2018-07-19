---
title: aws-cloudfront
date: 2018-07-08 21:41:35
categories:
- DevOps
tags:
- AWS
- Cloud
---
# AMAZON CLOUDFRONT

Amazon CloudFront is a global Content Delivery Network (CDN) service. It integrates with other AWS products to give developers and businesses an easy way to distribute content to end users with low latency, high data transfer speeds, and no minimum usage commitments.

<!-- more -->

Amazon CloudFront is AWS CDN. It can be used to deliver your web content using Amazon’s global network of *edge locations*.

Amazon CloudFront is optimized to work with other AWS cloud services as the origin server, including Amazon S3 buckets, Amazon S3 static websites, Amazon Elastic Compute Cloud (Amazon EC2), and Elastic Load Balancing. Amazon CloudFront also works seamlessly with any non-AWS origin server, such as an existing on-premises web server. Amazon CloudFront also integrates with Amazon Route 53.

Amazon CloudFront supports all content that can be served over HTTP or HTTPS. This includes any popular static files that are a part of your web application, such as HTML files, images, JavaScript, and CSS files, and also audio, video, media files, or software downloads. Amazon CloudFront also supports serving dynamic web pages, so it can actually be used to deliver your entire website. Finally, Amazon CloudFront supports media streaming, using both HTTP and RTMP.

## Distributions

To use Amazon CloudFront, you start by creating a distribution, which is identified by a DNS domain name such as d111111abcdef8.cloudfront.net.

You can use the Amazon CloudFront distribution domain name as-is, or you can create a user-friendly DNS name in your own domain by creating a CNAME record in Amazon Route 53 or another DNS service. The CNAME is automatically redirected to your Amazon CloudFront distribution domain name.

## Origins

When you create a distribution, you must specify the DNS domain name of the origin—the Amazon S3 bucket or HTTP server—from which you want Amazon CloudFront to get the definitive version of your objects (web files).

For example:
* Amazon S3 bucket: myawsbucket.s3.amazonaws.com
* Amazon EC2 instance: ec2–203–0–113–25.compute-1.amazonaws.com
* Elastic Load Balancing load balancer: my-load-balancer-1234567890.us-west-2.elb.amazonaws.com
* Website URL: mywebserver.mycompanydomain.com

## cache control

Once requested and served from an edge location, objects stay in the cache until they expire or are evicted to make room for more frequently requested content. By default, objects expire from the cache after 24 hours.

Optionally, you can control how long objects stay in an Amazon CloudFront cache before expiring. To do this, you can choose to use Cache-Control headers set by your origin server or you can set the minimum, maximum, and default Time to Live (TTL) for objects in your Amazon CloudFront distribution.

You can also remove copies of an object from all Amazon CloudFront edge locations at any time by calling the invalidation Application Program Interface (API).

Instead of invalidating objects manually or programmatically, it is a best practice to use a version identifier as part of the object (file) path name. For example:

* Old file: assets/v1/css/narrow.css
* New file: assets/v2/css/narrow.css

## Dynamic Content, Multiple Origins, and Cache Behaviors

Serving static assets, such as described previously, is a common way to use a CDN. An Amazon CloudFront distribution, however, can easily be set up to serve dynamic content in addition to static content and to use more than one origin server. You control which requests are served by which origin and how requests are cached using a feature called cache behaviors.

A cache behavior lets you configure a variety of Amazon CloudFront functionalities for a given URL path pattern for files on your website.

The functionality you can configure for each cache behavior includes the following:

* The path pattern
* Which origin to forward your requests to
* Whether to forward query strings to your origin
* Whether accessing the specified files requires signed URLs
* Whether to require HTTPS access
* The amount of time that those files stay in the Amazon CloudFront cache (regardless of the value of any Cache-Control headers that your origin adds to the files)

## Private Content 

In many cases, you may want to restrict access to content in Amazon CloudFront to only selected requestors, such as paid subscribers or to applications or users in your company network. Amazon CloudFront provides several mechanisms to allow you to serve private content. These include:

* Signed URLs Use URLs that are valid only between certain times and optionally from certain IP addresses.

* Signed Cookies Require authentication via public and private key pairs.

* Origin Access Identities (OAI) Restrict access to an Amazon S3 bucket only to a special Amazon CloudFront user associated with your distribution. This is the easiest way to ensure that content in a bucket is only accessed by Amazon CloudFront.