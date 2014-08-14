---
title: Using Boto to associate a domain to an ELB
layout: post
author: Micah Hausler
comments: true
---

The last few months at Ambition, we've been busy working on automated customer signup and setup. We currently set up a separate subdomain for each customer that points to an Elastic Load Balancer. Up to this point, activation of each new customer has been a manual process: Initializing servers, setting up an ELB, and pointing a new subdomain to that ELB through the AWS web console.

In our preparation for automated signups, we needed a solution that would scale.The greatest benefit for us to using Amazon Web Services is that an API endpoint exists for every operation. For most services, [boto](http://boto.readthedocs.org/en/latest/) (the AWS python library) has excellent walkthroughs, however for Route 53, only simple [API documentation](http://boto.readthedocs.org/en/latest/ref/route53.html) is currently available. After searching through Amazon's own Route 53 developer guide, we came across [a tutorial for setting up a domain to an ELB](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/HowToAliasRRS.html).
Since I could not find a tutorial elsewhere, below is a summary of how to associate a domain to an ELB.

>Note: This example is for simple routing only. If you want to get into latency or weighted policies, check out boto's documentation on [ResourceRecordSets](http://boto.readthedocs.org/en/latest/ref/route53.html#boto.route53.record.ResourceRecordSets).

<script src="https://gist.github.com/micahhausler/7662214.js"></script>