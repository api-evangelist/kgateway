---
title: "Delegation in kgateway - scaling routing with multi-tenant ownership"
url: "/blog/delegation-kgateway/"
date: "Fri, 09 May 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
As environments scale, traffic routing through Kubernetes gateways naturally becomes increasingly complex. Microservices architecture adoption tends to amplify this challenge as what used to be a single route for a monolith becomes hundreds or even thousands of path-based matchers across services, often bundled under a shared hostname. While it’s technically possible to configure all routes in a single HTTPRoute resource, this monolithic approach doesn’t scale well in multi-team environments.
