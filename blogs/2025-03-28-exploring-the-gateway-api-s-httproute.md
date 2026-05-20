---
title: "Exploring the Gateway API's HTTPRoute"
url: "/blog/exploring-gateway-api-httproute/"
date: "Fri, 28 Mar 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
An HTTPRoute resource is the main resource in the Kubernetes Gateway API to define HTTP routing rules for one or more services in your cluster. Its configuration is simple and easy to comprehend. Take the following example route: apiVersion : gateway.networking.k8s.io/v1 kind : HTTPRoute metadata : name : httpbin spec : parentRefs : - name : my-gateway hostnames : - httpbin.example.com rules : - backendRefs : - name : httpbin port : 8000 The above routes all requests for the hostname httpbin.example.com to the backend service named httpbin listening on port 8000.
