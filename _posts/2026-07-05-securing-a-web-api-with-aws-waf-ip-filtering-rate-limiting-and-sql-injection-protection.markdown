---
title: 'Securing a Web API with AWS WAF: IP Filtering, Rate Limiting and SQL Injection
  Protection'
date: 2026-07-05 16:26:00 +02:00
license: false
show_subscribe: false
show_author_profile: false
mermaid: true
---

Modern web applications expose APIs that are often directly connected to sensitive backend systems. In this project, I worked on securing a REST API hosted behind Amazon API Gateway and connected to a backend Lambda function and an Amazon RDS MySQL database.

The goal was to design and validate a defensive layer using AWS WAF to protect the API from unauthorized access, excessive request volume, and common web application attacks such as SQL injection.

The architecture follows a typical serverless web application pattern:

<pre class="mermaid">
graph TD
  Client["Client"] --> WAF["AWS WAF Web ACL"] --> Gateway["Amazon API Gateway"] --> Lambda["AWS Lambda"] --> RDS["Amazon RDS MySQL"]
</pre>
<script src="https://cdn.jsdelivr.net/npm/mermaid@10.9.1/dist/mermaid.min.js"></script>

--------------
-----------------
---------------

<style>
  .mermaid-wrapper {
    width: 100%;
    display: flex;
    justify-content: center;
    margin: 24px 0;
  }

  .mermaid-wrapper svg {
    max-width: 100%;
    height: auto;
  }
</style>

<div class="mermaid-wrapper">
  <pre class="mermaid">
graph TD
  Client["Client"] --> WAF["AWS WAF Web ACL"] --> Gateway["Amazon API Gateway"] --> Lambda["AWS Lambda"] --> RDS["Amazon RDS MySQL"]
  </pre>
</div>

<script src="https://cdn.jsdelivr.net/npm/mermaid@10.9.1/dist/mermaid.min.js"></script>


The API exposes inventory data, which should only be consumed by trusted partners. Because the backend database contains sensitive business information, the API needed stronger controls at the edge before requests reached the application logic.

## What I implemented

The technical work focused on building a layered AWS WAF policy.

First, I created a Web ACL and associated it with the API Gateway endpoint. The default action was configured to block requests, forcing all allowed traffic to explicitly match security rules.

Then I implemented an IP allowlist using an AWS WAF IP set. This allowed only trusted partner IP addresses to access the API while blocking non-privileged sources.

To reduce the risk of abuse and volumetric attacks, I added a rate-based rule. This blocks clients that exceed the expected request threshold, helping protect the Lambda backend from unnecessary load and potential denial-of-service behavior.

Finally, I enabled an AWS managed SQL injection rule group to detect and block malicious query strings attempting to manipulate backend SQL queries.

## Attack scenarios tested

The project validates several real-world attack and abuse cases:

Requests from non-authorized IP addresses
Excessive API requests from the same source
SQL injection payloads in query parameters
Geo-based filtering as an additional access control

Each control was tested against the API to confirm whether AWS WAF allowed, blocked, counted, or labeled requests as expected.

## Monitoring and visibility

A key part of the project was not only blocking traffic, but also understanding what AWS WAF was detecting.

I used AWS WAF metrics and Amazon CloudWatch to review:

* allowed requests
* blocked requests
* rate-limited traffic
* SQL injection detections
* rule matches and labels

This visibility is important because web application security should not rely only on prevention. Monitoring allows security teams to validate assumptions, detect abuse patterns, and tune rules based on real traffic.


-----

### Key takeaway

This project shows how AWS WAF can be used as a practical protection layer for API Gateway workloads. By combining default-deny logic, IP filtering, rate limiting, managed rule groups, and CloudWatch metrics, it is possible to reduce exposure to common web threats before requests reach the application backend.

The full technical write-up includes the complete architecture, AWS WAF configuration, rule explanations, test commands, screenshots, and validation results.

Full Technical Report (PDF):

[https://github.com/SergioValverde/AWS-Labs/blob/main/secure_web_app_aws_waf.pdf](https://github.com/SergioValverde/AWS-Labs/blob/main/secure_web_app_aws_waf.pdf)

---
