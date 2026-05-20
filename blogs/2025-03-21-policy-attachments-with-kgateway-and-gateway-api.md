---
title: "Policy attachments with kgateway and Gateway API"
url: "/blog/policy-attachments/"
date: "Fri, 21 Mar 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
One of the major shortcomings of the venerable Kubernetes Ingress API was in the area of extensibility. The API specification did not address how implementers should specify features that were outside the scope of the ingress scenarios covered by the Ingress API. Implementers had to resort to Kubernetes resource annotations to expose additional features or capabilities such as timeouts, retries, or rate limiting.
