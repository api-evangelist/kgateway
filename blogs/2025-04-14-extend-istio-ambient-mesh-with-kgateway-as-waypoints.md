---
title: "Extend Istio Ambient Mesh with Kgateway as Waypoints"
url: "/blog/extend-istio-ambient-kgateway-waypoint/"
date: "Mon, 14 Apr 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
Ambient mesh is the new sidecarless data plane mode in the Istio service mesh . One of the key innovations of ambient mesh is that it splits Istio’s functionality into two distinct layers: a lightweight, secure overlay layer (implemented by a purpose-built node proxy called ztunnel ) and a Layer 7 processing layer (implemented by L7 proxies called waypoints ). As we designed ambient mesh with two distinct layers, we purposefully kept the secure overlay layer very lightweight with minimal function.
