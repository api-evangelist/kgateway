---
title: "Transformations in kgateway"
url: "/blog/transformation-in-kgateway/"
date: "Thu, 17 Apr 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
Transformations are a feature in kgateway that allows for the transformation of an incoming request or outgoing response. It offers the addition, removal, or replacement of HTTP headers and the manipulation of request or response body. While the Kubernetes Gateway API provides filters for request and response header modifiers , those filters are scoped to the manipulation of headers only, and provide only rudimentary capabilities such as adding, removing or updating headers with static values supplied as strings.
