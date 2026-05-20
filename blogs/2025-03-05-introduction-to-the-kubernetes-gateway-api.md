---
title: "Introduction to the Kubernetes Gateway API"
url: "/blog/introduction-to-kubernetes-gateway-api/"
date: "Wed, 05 Mar 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
Kubernetes has come a long way since its early days of exposing services via the original Ingress API. As more workloads adopt Kubernetes, the types of traffic management needed—ingress from the outside world, service-to-service (east-west) communication within the cluster, and egress to external systems—have also become more sophisticated. As the various implementations of ingress controllers emerged, it became clear that having a common, extensible standard for traffic management was critical to ensure stability, portability, and widespread community adoption.
