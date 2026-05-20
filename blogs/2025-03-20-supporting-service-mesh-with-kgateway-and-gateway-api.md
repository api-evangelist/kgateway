---
title: "Supporting Service Mesh with kgateway and Gateway API"
url: "/blog/supporting-service-mesh-kgateway-gateway-api/"
date: "Thu, 20 Mar 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
A service mesh allows you to configure and control traffic between Kubernetes services that are deployed in a cluster. Service-to-service communication is also referred to as “east-west” traffic and does not involve ingress gateways. When the Gateway API “SIG” (Special Interest Group) got together to design the service mesh integration, their goal was to accommodate service mesh use cases while keeping changes to the API at a minimum.
