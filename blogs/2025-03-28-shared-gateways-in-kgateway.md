---
title: "Shared Gateways in kgateway"
url: "/blog/shared-gateways/"
date: "Fri, 28 Mar 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
In part 1 of this blog series, we explored persona-based management as a new concept of the Kubernetes Gateway API compared to the monolithic or single-owner approach of the legacy Ingress API. By defining clear roles for infrastructure providers, cluster operators, and application developers, Gateway API enables teams to work independently while still adhering to organizational policies. In part 2 , we highlighted a second key design change: the ability to easily spin up many gateway instances within the same cluster, which share basic configuration by using the same GatewayClass.
