---
title: "Canary Deployments with kgateway and Argo Rollouts"
url: "/blog/canary-deployments-argo-rollouts/"
date: "Tue, 01 Apr 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
Introduction A canary deployment is a technique that allows teams to release a new version of software to a subset of consumers (people or other software) in a safe manner. Gradually, more consumers are moved to the latest version of the software until eventually all consumers are using the new version, at which point the old version can be retired. Canary deployments reduce the risk of releasing new software by testing it with a small group of consumers while continuing to run the stable version for the majority.
