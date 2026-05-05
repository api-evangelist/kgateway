---
title: "How Xepelin Runs Kubernetes Gateway API in Production with kgateway"
url: "/blog/how-xepelin-runs-kubernetes-gateway-api-in-production-with-kgateway/"
date: "Fri, 09 Jan 2026 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>Xepelin, one of the largest fintech companies in Latin America, was founded in Chile in 2019 and now also operates in Mexico. Xepelin focuses on providing personalized financial solutions designed to fuel growth without traditional bureaucratic processes.</p>
<p>At Xepelin, we have been running <strong>kgateway</strong> with <strong>Gateway API resources in production</strong> across multiple Kubernetes clusters, spanning ~500 namespaces and hundreds of route resources. Our platform integrates Kubernetes with Prometheus, OpenTelemetry, CloudWatch, Datadog, and other components to support a high-traffic, highly observable environment.</p>
<h3>Background<span class="hx-absolute -hx-mt-20" id="background"></span>
    <a class="subheading-anchor" href="#background"></a></h3><p>Xepelin’s platform runs on AWS using Kubernetes with EKS. We operate multiple environments and handle a significant amount of internal and external traffic. Over time, the traffic layer became more complex, both operationally and in terms of cost. Our main goals were to:</p>
<ul>
<li>Simplify the traffic architecture</li>
<li>Improve observability</li>
<li>Reduce infrastructure costs</li>
</ul>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/kgateway-at-xepelin-1.svg" width="" /> <figcaption style="font-style: italic;">Legacy traffic architecture at Xepelin based on AWS API Gateway, Lambda authorizers, and per-service Network Load Balancers, which led to operational complexity and high infrastructure costs.</figcaption></figure></div>
<p>As we operate the AWS API Gateway which controls all incoming traffic to the EKS clusters, with Lambda authorizers and Network Load Balancers (NLBs), we started to hit several issues:</p>
<ul>
<li>A large number of NLBs, leading to high infrastructure costs.</li>
<li>Fragmented traffic management, where Kubernetes specifications handled the Service (type LoadBalancer) configuration, while AWS API Gateway routes were created and modified separately using Terraform or the AWS Console. This split ownership resulted in a disjointed workflow and increased operational complexity when managing traffic changes end-to-end.</li>
<li>Fragmented observability across the traffic path, which made it impossible to have centralized metrics and end-to-end visibility across the API Gateway → NLB → Kubernetes Service chain. As a result, diagnosing issues such as sporadic 503 errors was slow and required manual investigation across multiple systems.</li>
</ul>
<p>Our initial model relied heavily on Kubernetes Services of type <strong>LoadBalancer</strong>, with AWS API Gateway routing traffic directly to the NLBs created by those services. This resulted in a large number of Network Load Balancers and steadily increasing operational costs.</p>
<p>At the time, we had the AWS Load Balancer Controller installed in each cluster. While this made it easy to create new load balancers, <strong>it also contributed significantly to infrastructure sprawl</strong>.</p>
<h3>Why kgateway?<span class="hx-absolute -hx-mt-20" id="why-kgateway"></span>
    <a class="subheading-anchor" href="#why-kgateway"></a></h3><p>Migrating away from an AWS API Gateway–centric architecture to a Kubernetes-native solution was not just a tooling change. It represented a <strong>fundamental shift</strong> in how we expose, operate, and observe our APIs. Because of this, choosing the gateway for north–south traffic was a critical decision.</p>
<p>The existing architecture had been in place for years and <strong>did not scale well</strong>. Any migration therefore had to meet one fundamental requirement: it needed to be as transparent as possible for both teams and existing traffic flows.</p>
<p>With that in mind, we focused on the following aspects.</p>
<h4>Transparent and low-friction migration<span class="hx-absolute -hx-mt-20" id="transparent-and-low-friction-migration"></span>
    <a class="subheading-anchor" href="#transparent-and-low-friction-migration"></a></h4><p>One of the main challenges was replicating the behavior that already existed in AWS API Gateway, especially the Lambda Authorizer.</p>
<p>This authorizer received each request, performed several validations, and interacted with AWS services such as Secrets Manager.</p>
<p>With kgateway, we were able to replicate this behavior thanks to its tight integration with Envoy and its support for external authorization.</p>
<h4>Full, first-class support for Gateway API<span class="hx-absolute -hx-mt-20" id="full-first-class-support-for-gateway-api"></span>
    <a class="subheading-anchor" href="#full-first-class-support-for-gateway-api"></a></h4><p>Full, native support for the Kubernetes Gateway API was a key requirement for us.</p>
<p>We wanted the Gateway API to be the primary way to model north–south traffic, not an optional layer. Kgateway stood out because the Gateway API is at the core of its design, which made it a natural fit for how we manage routing and traffic behavior in production.</p>
<h4>Built on Envoy<span class="hx-absolute -hx-mt-20" id="built-on-envoy"></span>
    <a class="subheading-anchor" href="#built-on-envoy"></a></h4><p>Kgateway is built on top of Envoy, one of the most widely used and powerful proxies in the cloud-native ecosystem. Envoy provides:</p>
<ul>
<li>Rich metrics per proxy and per route (counters, gauges, histograms)</li>
<li>Detailed visibility into request rates, latencies, errors, and upstream/downstream behavior</li>
<li>Advanced flexibility around timeouts, retries, circuit breaking, and more</li>
</ul>
<h4>History and maturity: Born as Gloo<span class="hx-absolute -hx-mt-20" id="history-and-maturity-born-as-gloo"></span>
    <a class="subheading-anchor" href="#history-and-maturity-born-as-gloo"></a></h4><p>Before being called kgateway, the project was known as Gloo and was developed by Solo.io. This was an important factor for us because:</p>
<ul>
<li>It has been proven across hundreds of production deployments</li>
<li>It is not experimental, but a very mature and stable project</li>
<li>It has processed billions of requests for large companies worldwide</li>
</ul>
<p>This track record was a major advantage over newer or less widely adopted alternatives.</p>
<h4>Advanced observability with native OpenTelemetry integration<span class="hx-absolute -hx-mt-20" id="advanced-observability-with-native-opentelemetry-integration"></span>
    <a class="subheading-anchor" href="#advanced-observability-with-native-opentelemetry-integration"></a></h4><h5>Metrics<span class="hx-absolute -hx-mt-20" id="metrics"></span>
    <a class="subheading-anchor" href="#metrics"></a></h5><ul>
<li>Control plane: The kgateway control plane exposes Prometheus metrics that give full visibility into how synchronization and reconciliation are behaving.</li>
<li>Data plane: Each Envoy pod exposes detailed Prometheus metrics describing real traffic behavior.
This allows metrics to be consumed directly by our existing observability stack or via an OpenTelemetry Collector, fully integrating with our monitoring infrastructure.</li>
</ul>
<h5>Access logs<span class="hx-absolute -hx-mt-20" id="access-logs"></span>
    <a class="subheading-anchor" href="#access-logs"></a></h5><p>Envoy access logs from the data plane are very easy to expose using resources managed by the kgateway controller itself. It also provides native exporters to OpenTelemetry collectors.</p>
<h5>Traces<span class="hx-absolute -hx-mt-20" id="traces"></span>
    <a class="subheading-anchor" href="#traces"></a></h5><p>Kgateway includes native trace exporters that can send traces directly to OpenTelemetry collectors.</p>
<h4>Fast synchronization and near-immediate feedback<span class="hx-absolute -hx-mt-20" id="fast-synchronization-and-near-immediate-feedback"></span>
    <a class="subheading-anchor" href="#fast-synchronization-and-near-immediate-feedback"></a></h4><p>One aspect that turned out to be more important than expected was synchronization speed between Kubernetes resources and the gateway data plane.
With kgateway, changes to HTTPRoute resources are reflected in Envoy near-instantaneously, typically within seconds in our experience. This contrasts sharply with our previous AWS API Gateway setup, where changes were slower, involved multiple deployment steps, and often required manual cross-team coordination.
The low latency between HTTPRoute changes in the control plane and their application in the data plane was a key factor in choosing kgateway. It also fits very naturally with a GitOps model, where manifest changes are quickly reflected in live traffic.</p>
<h4>Focus on north–south traffic with a path to east–west<span class="hx-absolute -hx-mt-20" id="focus-on-northsouth-traffic-with-a-path-to-eastwest"></span>
    <a class="subheading-anchor" href="#focus-on-northsouth-traffic-with-a-path-to-eastwest"></a></h4><p>The primary goal of this migration was to fix north–south traffic routing, specifically:</p>
<ul>
<li>Reducing costs by cutting down the number of NLBs</li>
<li>Gaining real and detailed observability</li>
<li>Enforcing external security policies</li>
</ul>
<p>Beyond that, we wanted an architecture that could evolve toward a service mesh capable of handling east–west traffic.
Kgateway fit well because it:</p>
<ul>
<li>Uses Envoy as its data plane</li>
<li>Integrates cleanly with Gateway API</li>
<li>Can act as an ingress for a future service mesh</li>
<li>Works particularly well as an ingress for Istio Ambient Mode, where Envoy acts as a waypoint proxy to apply L7 policies without sidecars</li>
</ul>
<h4>Comparison with other tools<span class="hx-absolute -hx-mt-20" id="comparison-with-other-tools"></span>
    <a class="subheading-anchor" href="#comparison-with-other-tools"></a></h4><p>We also evaluated Kong and Traefik before we decided on kgateway:</p>
<table>
  <thead>
      <tr>
          <th style="text-align: left;"><strong>Feature / Tool</strong></th>
          <th style="text-align: left;"><strong>Kgateway</strong></th>
          <th style="text-align: left;"><strong>Kong</strong></th>
          <th style="text-align: left;"><strong>Traefik</strong></th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;"><em>Core technology</em></td>
          <td style="text-align: left;">Envoy</td>
          <td style="text-align: left;">Nginx/OpenResty</td>
          <td style="text-align: left;">Go (open source)</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Native Gateway API support</em></td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Sync speed (HTTPRoute → live route)</em></td>
          <td style="text-align: left;">Fast</td>
          <td style="text-align: left;">Variable</td>
          <td style="text-align: left;">Variable</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Extensible external authorization</em></td>
          <td style="text-align: left;">Decoupled, flexible</td>
          <td style="text-align: left;">Plugin-based (OSS and Enterprise)</td>
          <td style="text-align: left;">ForwardAuth middleware</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Native rate limiting</em></td>
          <td style="text-align: left;">Yes (local and global)</td>
          <td style="text-align: left;">Basic in OSS, advanced features enterprise</td>
          <td style="text-align: left;">Basic</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Configurable timeouts (&gt;30s)</em></td>
          <td style="text-align: left;">Per route / backend</td>
          <td style="text-align: left;">Plugin-based</td>
          <td style="text-align: left;">Limited</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Prometheus metrics per data-plane pod</em></td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>OpenTelemetry compatibility</em></td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Advanced L7 observability</em></td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Basic in OSS, enhanced in Enterprise</td>
          <td style="text-align: left;">Standard (traces, metrics, logs)</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>North–south traffic focus</em></td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
          <td style="text-align: left;">Yes</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>East–west traffic</em></td>
          <td style="text-align: left;">Via Istio</td>
          <td style="text-align: left;">Via separate service mesh</td>
          <td style="text-align: left;">Via separate service mesh</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Istio Ambient / Waypoint integration</em></td>
          <td style="text-align: left;">Waypoint-ready</td>
          <td style="text-align: left;">No</td>
          <td style="text-align: left;">No</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>AWS Lambda invocation</em></td>
          <td style="text-align: left;">Native</td>
          <td style="text-align: left;">Via plugins</td>
          <td style="text-align: left;">No (community plugins available)</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Licensing model</em></td>
          <td style="text-align: left;">Open source</td>
          <td style="text-align: left;">Open source (OSS) + Enterprise subscription</td>
          <td style="text-align: left;">Open source + Commercial products</td>
      </tr>
      <tr>
          <td style="text-align: left;"><em>Production maturity</em></td>
          <td style="text-align: left;">Very high (ex-Gloo)</td>
          <td style="text-align: left;">High</td>
          <td style="text-align: left;">High</td>
      </tr>
  </tbody>
</table>
<h3>How Xepelin uses kgateway<span class="hx-absolute -hx-mt-20" id="how-xepelin-uses-kgateway"></span>
    <a class="subheading-anchor" href="#how-xepelin-uses-kgateway"></a></h3><p>Today, kgateway is the core of our traffic layer at Xepelin. We use it consistently across production, development, and testing environments, which allows us to keep the same operational model throughout the entire service lifecycle.</p>
<p>When traffic reaches the platform, either through an Application Load Balancer for public traffic or a Network Load Balancer for internal traffic, both flows pass through Envoy proxies in the kgateway data plane. From there, all routing is handled declaratively within Kubernetes.</p>
<p>Applications are managed through an internal developer platform (IDP) that centralizes onboarding and governance. This IDP is responsible for creating HTTPRoute resources and binding them to the appropriate Gateway objects based on environment and traffic type. For application teams, the process is <strong>simple, consistent and self-service</strong>, while gateway complexity is fully abstracted away by the platform.</p>
<p>Synchronization speed is another key advantage. From the moment an HTTPRoute is created or updated, it typically takes only a few seconds for the route to become active. This fast feedback loop significantly improved our service lifecycle, reducing wait times and making deployments more predictable.</p>
<p>Beyond basic routing, we rely on kgateway extensions to control advanced behavior. Using BackendPolicies such as BackendConfigPolicy, we tune Envoy-specific parameters including upstream keep-alives, backend timeouts, and connection handling. We also apply native rate limiting, allowing us to protect services and standardize limits without relying on external systems.</p>
<p>From a security standpoint, we can integrate our own custom external auth service with kgateway. This internally developed service can be enabled and configured per gateway or per route. It allowed us to replicate and evolve the Lambda Authorizer model we used previously, while keeping authorization logic decoupled from the gateway and fully Kubernetes-native.</p>
<p>Overall, this model based on an internal IDP, Gateway API, kgateway, and custom external auth gave us a traffic layer that is <strong>predictable, fast to operate, and easy to scale</strong>.
Kgateway did not just replace the previous architecture, it became a core platform component that we continue to build on.</p>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/kgateway-at-xepelin-2.svg" width="" /> <figcaption style="font-style: italic;">Current traffic architecture using kgateway and the Kubernetes Gateway API, with centralized ingress, shared gateways, and declarative routing managed entirely inside Kubernetes.</figcaption></figure></div>
<h3>How we use kgateway telemetry<span class="hx-absolute -hx-mt-20" id="how-we-use-kgateway-telemetry"></span>
    <a class="subheading-anchor" href="#how-we-use-kgateway-telemetry"></a></h3><p>We currently use kgateway telemetry across two distinct layers. On one side, we rely on access logs for each gateway. On the other, we consume metrics from both the control plane and the data plane, with most of the focus on the data plane.</p>
<h4>Metrics<span class="hx-absolute -hx-mt-20" id="metrics-1"></span>
    <a class="subheading-anchor" href="#metrics-1"></a></h4><p>In practice, the metrics we rely on the most are those exposed by Envoy in the data plane, as they provide a clear and accurate picture of how traffic is actually behaving.</p>
<p>We primarily monitor:</p>
<ul>
<li>Request counts</li>
<li>Latencies</li>
<li>Throughput</li>
<li>Errors and timeouts</li>
<li>Envoy response codes and flags</li>
</ul>
<p>What makes these metrics especially valuable is their <strong>level of granularity</strong>. We can break them down by <strong>backend service, Gateway API resource, environment, and even custom tags</strong>. This is straightforward to achieve thanks to kgateway’s native OpenTelemetry integration, which allows us to enrich telemetry without locking ourselves into a specific observability vendor.</p>
<p>In production, we send these metrics to Datadog. In development and testing environments, we send them to Prometheus and visualize them in Grafana. The model stays the same, only the backend changes.</p>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/kgateway-at-xepelin-3.svg" width="" /> <figcaption style="font-style: italic;">Datadog dashboards built from Envoy and kgateway metrics, providing detailed visibility into request rates, latencies, errors, and traffic behavior per service.</figcaption></figure></div>
<p>Through our Grafana dashboards, we can easily drill into response codes and investigate non-200 responses in more detail when needed.</p>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/kgateway-at-xepelin-4.svg" width="" /> <figcaption style="font-style: italic;">Per-request Envoy access logs from kgateway, including HTTP status codes, request paths, upstream services, and end-to-end request duration.</figcaption></figure></div>
<h4>Logs: understanding individual requests<span class="hx-absolute -hx-mt-20" id="logs-understanding-individual-requests"></span>
    <a class="subheading-anchor" href="#logs-understanding-individual-requests"></a></h4><p>In parallel, we rely heavily on Envoy access logs to gain detailed visibility at the individual request level. Currently, these logs are sent to Amazon CloudWatch, with the option to route them to Loki in the future.
Access logs are critical when something does not look right. Metrics tell you that something is wrong; logs tell you exactly what happened. More than once, Envoy response codes and flags found in access logs helped us determine whether an issue originated in the backend, the gateway, or somewhere in between.</p>
<h4>Metrics and logs together<span class="hx-absolute -hx-mt-20" id="metrics-and-logs-together"></span>
    <a class="subheading-anchor" href="#metrics-and-logs-together"></a></h4><p>In day-to-day operations, the real value comes from using metrics and logs together. Metrics help us detect issues quickly, while logs allow us to understand them in depth.
Having this level of visibility into the traffic layer was a major improvement over our previous architecture and is now a fundamental part of how we operate kgateway in production.</p>
<h3>Cost and operational impact<span class="hx-absolute -hx-mt-20" id="cost-and-operational-impact"></span>
    <a class="subheading-anchor" href="#cost-and-operational-impact"></a></h3><p>One of the clearest impacts of adopting kgateway was on both cost and operational overhead.
Before the migration, our model was essentially one application, one Network Load Balancer. As the platform grew, the number of NLBs grew with it, eventually becoming expensive and difficult to manage.</p>
<p>After consolidating traffic through kgateway, we reduced the number of NLBs by approximately <strong>96%</strong>. Instead of provisioning a load balancer per service, we now route multiple applications through a small set of shared entry points.</p>
<p>This change had an immediate impact on infrastructure costs, but the bigger win was operational simplicity. With far fewer NLBs:</p>
<ul>
<li>There is less infrastructure to maintain</li>
<li>Fewer components can break in the traffic path</li>
<li>Networking per environment is significantly simpler</li>
<li>Teams spend less time dealing with load balancer-related issues</li>
</ul>
<p>In practice, kgateway helped us move away from maintaining large amounts of repetitive infrastructure and <strong>focus more on improving the platform itself</strong>. While the cost savings were important, the <strong>reduction in operational effort and complexity was just as valuable</strong>.</p>
<h3>Challenges and lessons learned<span class="hx-absolute -hx-mt-20" id="challenges-and-lessons-learned"></span>
    <a class="subheading-anchor" href="#challenges-and-lessons-learned"></a></h3><p>Adopting kgateway was very positive overall, but it also came with important lessons that we now consider essential for stable operation.</p>
<h4>Every backend is different<span class="hx-absolute -hx-mt-20" id="every-backend-is-different"></span>
    <a class="subheading-anchor" href="#every-backend-is-different"></a></h4><p>Each application, seen by Envoy as an upstream, maintains its own connection pool. There is no single configuration that works for everything. Defining well-tuned BackendPolicies per service, including keep-alives, timeouts, and retries, was critical to achieving stable connections and avoiding intermittent errors.</p>
<h4>Gateway API requires a mindset shift<span class="hx-absolute -hx-mt-20" id="gateway-api-requires-a-mindset-shift"></span>
    <a class="subheading-anchor" href="#gateway-api-requires-a-mindset-shift"></a></h4><p>Coming from traditional Ingress and load balancer models, it takes time to adjust to the Gateway API way of modeling traffic. Concepts like Gateways, HTTPRoutes, and namespace-level delegation are not complex, but they are different. Once the model clicks, things start to make a lot more sense.</p>
<h4>The community helped a lot<span class="hx-absolute -hx-mt-20" id="the-community-helped-a-lot"></span>
    <a class="subheading-anchor" href="#the-community-helped-a-lot"></a></h4><p>During adoption, we had many questions around both design and implementation. Our experience with the kgateway community was extremely positive—responses were clear and fast, helping us move forward without getting stuck.</p>
<h4>North–south traffic still requires careful tuning<span class="hx-absolute -hx-mt-20" id="northsouth-traffic-still-requires-careful-tuning"></span>
    <a class="subheading-anchor" href="#northsouth-traffic-still-requires-careful-tuning"></a></h4><p>Even though kgateway abstracts much of the complexity, timeouts and keep-alives still need to be aligned across ALBs, NLBs, Envoy, and backends. When these settings are misaligned, hard-to-debug issues can appear unless strong telemetry is in place.</p>
<h4>Without telemetry, this would be impossible to operate<span class="hx-absolute -hx-mt-20" id="without-telemetry-this-would-be-impossible-to-operate"></span>
    <a class="subheading-anchor" href="#without-telemetry-this-would-be-impossible-to-operate"></a></h4><p>Having metrics and logs from day one was critical. On multiple occasions, Envoy telemetry was what allowed us to pinpoint whether an issue lived in the gateway, the backend, or outside the cluster.</p>
<h4>External auth is flexible, but not trivial<span class="hx-absolute -hx-mt-20" id="external-auth-is-flexible-but-not-trivial"></span>
    <a class="subheading-anchor" href="#external-auth-is-flexible-but-not-trivial"></a></h4><p>Moving authorization outside the gateway was the right decision, but it requires careful design around latency, timeouts, and resilience. Authorization remains part of the critical request path, and it must be treated as such.</p>
<p>Overall, kgateway solved many structural problems for us, but like any core platform component, it works best when operated with clear practices and good operational discipline. These lessons are now baked into how we design and run the traffic layer at Xepelin.</p>
<h3>Wrapping up<span class="hx-absolute -hx-mt-20" id="wrapping-up"></span>
    <a class="subheading-anchor" href="#wrapping-up"></a></h3><p>We are very happy with all the features we explored with kgateway, including built-in external authorization, native OpenTelemetry integration, and high-performance routing. We’re also testing the waypoint proxy integration with Istio ambient mesh in a proof-of-concept phase, and so far it has helped us validate our future service mesh architecture.</p>
<p>We look forward to continuing to work with the kgateway community and exploring the AI gateway capabilities that kgateway is bringing into this space. If you are looking for an alternative for Ingress NGINX, we highly recommend you check out kgateway and reach out to the <a href="https://kgateway.dev/slack/" rel="noopener" target="_blank">community</a> for questions.</p>
