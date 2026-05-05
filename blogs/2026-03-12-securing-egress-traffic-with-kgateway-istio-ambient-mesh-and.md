---
title: "Securing Egress Traffic with kgateway, Istio Ambient Mesh, and Kyverno: LFX Mentorship Blog"
url: "/blog/securing-egress-traffic-with-kgateway-istio-ambient-mesh-and-kyverno-lfx-mentorship-blog/"
date: "Thu, 12 Mar 2026 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>Modern Kubernetes platforms already secure in-cluster (east–west) communication through service meshes like Istio. But the same rigor rarely applies to outbound (egress) traffic — calls leaving the cluster to reach APIs, model endpoints, or third-party services. Without standardized policies, teams often struggle to track which workloads are reaching out to the internet, whether those requests are properly authorized, and how to enforce consistent authentication without adding custom code in every service. This blog reflects my LFX Mentorship journey, where I deep-dived into designing a well governed, observable, and secure egress pathway for Kubernetes workloads.
Modern Kubernetes platforms already secure in-cluster (east–west) communication.</p>
<p>These inconsistencies create both visibility gaps and compliance risks. Platform engineers also need to ensure that failures in external dependencies don’t cascade back into the cluster. The goal of this post is to design a policy-enforced, resilient egress path that is observable, auditable, and governed, using three key technologies working together: <strong><a href="https://kgateway.dev/docs/envoy/latest/quickstart/" rel="noopener" target="_blank">kgateway</a></strong> (an Envoy-based pluggable gateway), <strong><a href="https://ambientmesh.io/docs/about/overview/" rel="noopener" target="_blank">Istio Ambient Mesh</a></strong> (providing secure L4 identity and mTLS), and <strong><a href="https://kyverno.io/docs/installation/" rel="noopener" target="_blank">Kyverno</a></strong> (for external authorization and configuration governance).</p>
<h2>The challenge with outgoing traffic<span class="hx-absolute -hx-mt-20" id="the-challenge-with-outgoing-traffic"></span>
    <a class="subheading-anchor" href="#the-challenge-with-outgoing-traffic"></a></h2><p>In most clusters, outbound requests leave through arbitrary nodes or NAT gateways, bypassing the control that meshes offer internally. This leads to shadow egress — workloads connecting directly to external endpoints without policy or inspection. Authorization checks are often inconsistent, and operations teams have little visibility into which service made which request.</p>
<p>Even when egress is centralized, resilience policies such as timeouts and retries vary across services, and new external hosts can appear without any governance review. Together, these issues create a fragile perimeter where traffic is secure inside the mesh, but uncontrolled once it leaves the cluster.</p>
<h2>The role of an egress gateway<span class="hx-absolute -hx-mt-20" id="the-role-of-an-egress-gateway"></span>
    <a class="subheading-anchor" href="#the-role-of-an-egress-gateway"></a></h2><p>An egress gateway solves this by acting as a single, managed exit point from the mesh. Every outbound HTTP(S) call passes through it, where you can consistently apply identity-aware routing, authorization policies, retry logic, and observability.</p>
<p>In this post, we’ll show how kgateway, an Envoy-powered gateway built for Gateway API and Ambient Mesh, can serve as that controlled exit point. By combining it with Kyverno’s external authorization (ExtAuth) and Istio Ambient’s secure overlay, we can inspect and govern all outbound traffic without sidecars or additional proxies.</p>
<h2>How the components work together and what does the Blog demonstrate<span class="hx-absolute -hx-mt-20" id="how-the-components-work-together-and-what-does-the-blog-demonstrate"></span>
    <a class="subheading-anchor" href="#how-the-components-work-together-and-what-does-the-blog-demonstrate"></a></h2><p>At the foundation, Istio Ambient Mesh provides L4 security and workload identity via its ztunnel layer. On top of that, kgateway manages egress traffic through L7 routing and resilience features, while Kyverno acts on two planes — as an external authorization service at runtime and as a policy engine for governance during configuration.</p>
<p>Together, they form a layered security and governance model for egress traffic. The Ollama container running outside the cluster serves as our real-world external endpoint — representing a model API or SaaS service — to validate that all outbound requests follow the intended security and control flow.</p>
<p>This guide walks you through how to register an external host using an Istio ServiceEntry, route outbound traffic via a dedicated kgateway egress proxy, and then enforce Kyverno-based authorization at Layer 7.</p>
<p>We will also introduce a Kyverno ClusterPolicy to add governance at the control plane, ensuring all new external ServiceEntries are properly labeled and approved. Finally, we’ll test resilience by pausing the external Ollama service and observing retry and timeout behaviors (503/504) enforced through kgateway’s TrafficPolicy.</p>
<h2>Architecture<span class="hx-absolute -hx-mt-20" id="architecture"></span>
    <a class="subheading-anchor" href="#architecture"></a></h2><pre class="mermaid hx-mt-6">
  flowchart LR
    subgraph "Ambient Mesh (ztunnel layer)"
        Client["curl-test-client\nNamespace: default\nLabel: istio.io/dataplane-mode=ambient"]
        KGW["kGateway egress Gateway\nGatewayClass: kgateway"]
    end

    Client --&gt;|"mTLS via ztunnel"| KGW
    KGW --&gt;|"ExtAuth gRPC"| KyvernoRuntime["Kyverno authz server\n(Envoy gRPC)"]
    ServiceEntry --&gt;|"DNS/MESH_EXTERNAL"| Ollama["External Ollama server\nDocker host :11434"]

    subgraph "Control Plane Governance"
        KyvernoAdmission["Kyverno ClusterPolicy\nsecurity.corp/egress-approved"]
    end

    KyvernoAdmission -.-&gt;|"validates"| ServiceEntry
    KyvernoAdmission -.-&gt;|"watches"| KGW
    KyvernoRuntime -.-&gt;|"status"| KyvernoAdmission
</pre><h2>Prequisites<span class="hx-absolute -hx-mt-20" id="prequisites"></span>
    <a class="subheading-anchor" href="#prequisites"></a></h2><p>Before setting up the integration between kgateway, Istio Ambient Mesh, and Kyverno, ensure that your local or lab environment includes the following tools and configurations:</p>
<ul>
<li><a href="https://docs.docker.com/get-docker/" rel="noopener" target="_blank">Docker</a></li>
<li><a href="https://kind.sigs.k8s.io/docs/user/quick-start/" rel="noopener" target="_blank">kind (Kubernetes in Docker)</a></li>
<li><a href="https://kubernetes.io/docs/tasks/tools/" rel="noopener" target="_blank">kubectl</a></li>
<li><a href="https://helm.sh/docs/intro/install/" rel="noopener" target="_blank">Helm</a></li>
<li><a href="https://kgateway.dev/docs/envoy/latest/quickstart/" rel="noopener" target="_blank">kgateway</a></li>
<li><a href="https://kyverno.io/docs/installation/" rel="noopener" target="_blank">Kyverno</a></li>
</ul>
<h1>How Istio Ambient Mesh is different from Service Mesh</h1><p>Istio Ambient Mesh is a sidecar-less data plane model designed to reduce operational overhead and improve resource efficiency for service-to-service communication. Instead of using per-pod sidecar proxies, Ambient splits its data plane into two layers:</p>
<ul>
<li><strong>Secure Overlay Layer (L4)</strong> — Handled by the lightweight <em>ztunnel</em>, providing mTLS, identity, and network telemetry.</li>
<li><strong>Waypoint Proxy Layer (L7)</strong> — Handles application-layer policies such as routing, authentication, RBAC, and external authorization.</li>
</ul>
<p>This separation lets platform teams choose when L7 processing is necessary, reducing cost and computational overhead for workloads that only require transport security.</p>
<h2>kgateway&rsquo;s integration with Ambient Mesh<span class="hx-absolute -hx-mt-20" id="kgateways-integration-with-ambient-mesh"></span>
    <a class="subheading-anchor" href="#kgateways-integration-with-ambient-mesh"></a></h2><p>kgateway integrates to Ambient Mesh for managing our workloads through Layer 4 and Layer 7 network policies. But the thing that sets its apart from other Gateway solutions is that, kgateway is the first project that can be used as a pluggable waypoint for Istio.
kgateway has been built on same Envoy engine that Istio’s waypoint implementation uses, which has certain features including Istio API Compatability, Shared Observability, Faster Adoption of Security Featrues and Unified Configurational Model with Ambient Mesh.</p>
<h2>Prepare your kgateway environment<span class="hx-absolute -hx-mt-20" id="prepare-your-kgateway-environment"></span>
    <a class="subheading-anchor" href="#prepare-your-kgateway-environment"></a></h2><p>Before integrating kagteway with Istio Ambient, ensure we have:</p>
<ol>
<li>Follow the <a href="https://kgateway.dev/docs/envoy/latest/quickstart/" rel="noopener" target="_blank">Get started guide</a> to install kgateway in a kind cluster</li>
<li>Follow the <a href="https://kgateway.dev/docs/envoy/latest/install/sample-app/" rel="noopener" target="_blank">Sample app guide</a> to create a gateway proxy with an HTTP listener and deploy the httpbin sample app.</li>
<li>Set up an ambient mesh in your cluster to secure service-to-service communication with mutual TLS by following the <a href="https://ambientmesh.io/docs/quickstart/" rel="noopener" target="_blank">ambientmesh.io</a> quickstart documentation.</li>
<li>Deploy the Ollama Container at port number 11434, binding to 0.0.0.0 so the Kubernetes virtual machine can access it via the host&rsquo;s bridge network.
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>docker run -d -v ollama:/root/.ollama -p 11434:11434 -e OLLAMA_HOST=0.0.0.0 ollama/ollama --name ollama-server</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>Get the Container IP of ollama container which will be inserted at all the **<code>address filds</code> which is 172.17.0.2 in our case.
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' &lt;CONTIANER_NAME&gt;</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
Here it will get container&rsquo;s IP as an output:
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>172.17.0.2</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>Install the Kyverno Envoy authorization plugin that will receive external authorization (ExtAuth) calls from kgateway:
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>helm install kyverno-authz-server --namespace default --create-namespace --wait \
--version 0.1.0 --repo https://kyverno.github.io/kyverno-envoy-plugin \
kyverno-authz-server</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
</ol>
<p>While Istio ambient provides Authorization, Authentication and Egress still, there are several scenarios where kgateway can offer a more powerful alternative:</p>
<h2>Securely Egress Traffic with kgateway + Istio Integration<span class="hx-absolute -hx-mt-20" id="securely-egress-traffic-with-kgateway--istio-integration"></span>
    <a class="subheading-anchor" href="#securely-egress-traffic-with-kgateway--istio-integration"></a></h2><p>To establish a dedicated, policy-enforced egress path, we must combine three core resources: the <strong>Istio ServiceEntry</strong> (to register the external host), the <strong>kgateway Egress Gateway</strong>, and the <strong>HTTPRoute</strong> (to apply the routing and security logic).</p>
<ul>
<li>Our target is an external Ollama container running on the host machine (host.docker.internal) on the default port 11434 and <code>ServiceEntries</code> injects the external Ollama endpoint into the Istio service registry for which we&rsquo;ll use static resolution for our local Docker Desktop bridge.</li>
<li>Define the Gateway resource, leveraging the kgateway GatewayClass to instantiate a dedicated proxy that is explicitly listening for outbound traffic to the Ollama host with an HTTPRoute.</li>
</ul>
<h3>Why a ServiceEntry is used instead of a <code>kgateway Backend</code>?<span class="hx-absolute -hx-mt-20" id="why-a-serviceentry-is-used-instead-of-a-kgateway-backend"></span>
    <a class="subheading-anchor" href="#why-a-serviceentry-is-used-instead-of-a-kgateway-backend"></a></h3><p>kgateway also supports defining a <strong>Backend</strong> object (with static endpoints) that the HTTPRoute could reference directly. While that approach works for simple north-south scenarios, it keeps the external host outside of Istio’s service registry.</p>
<p>Using a ServiceEntry delivers two mesh-native advantages:</p>
<ol>
<li><strong>Unified observability:</strong> By registering the Ollama target with a ServiceEntry, Istio recognizes it as part of the mesh topology. That means ambient telemetry, metrics, and traces automatically include calls to the external service, in addition to the kgateway-specific stats. A kgateway Backend would only emit gateway-layer metrics.</li>
<li><strong>Ambient mesh security:</strong> Because the egress gateway itself participates in Ambient Mesh, Istio can enforce mTLS from the ztunnel layer to the gateway. ServiceEntries are the mechanism Ambient uses to represent external workloads, so mTLS and L4 policy continue to apply even though the destination lives outside the cluster.</li>
</ol>
<p>In short, a kgateway Backend is great when you only need Gateway API features, but combining a ServiceEntry with kgateway lets you extend Istio’s observability and identity controls all the way to that external dependency.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">egress-kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">gatewayClassName</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">8080</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">allowedRoutes</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">namespaces</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">from</span><span class="p">:</span><span class="w"> </span><span class="l">All</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">networking.istio.io/v1beta1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">ServiceEntry</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ollama-external-host</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">labels</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">security.corp/egress-approved</span><span class="p">:</span><span class="w"> </span><span class="s2">"true"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hosts</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="s2">"host.docker.internal"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">location</span><span class="p">:</span><span class="w"> </span><span class="l">MESH_EXTERNAL</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">ports</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">number</span><span class="p">:</span><span class="w"> </span><span class="m">11434</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http-ollama</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">resolution</span><span class="p">:</span><span class="w"> </span><span class="l">DNS</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ollama-egress-route</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default </span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parentRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">egress-kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">sectionName</span><span class="p">:</span><span class="w"> </span><span class="l">http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hostnames</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="s2">"host.docker.internal"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ollama-external-host</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">11434</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">ServiceEntry</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="s2">"networking.istio.io"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h3>Enroll kgateway and workloads into Ambient Mesh<span class="hx-absolute -hx-mt-20" id="enroll-kgateway-and-workloads-into-ambient-mesh"></span>
    <a class="subheading-anchor" href="#enroll-kgateway-and-workloads-into-ambient-mesh"></a></h3><p>For the egress gateway to participate in Ambient Mesh and benefit from mTLS and unified observability, we need to enroll the namespace where the gateway is deployed. This ensures traffic from mesh-enabled workloads to the gateway is secured via Ambient&rsquo;s ztunnel layer.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl">kubectl label ns default istio.io/dataplane-mode<span class="o">=</span>ambient</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>This label instructs Istio to configure ztunnel sockets on all pods in the <code>default</code> namespace, including the kgateway egress proxy pods. Now traffic from mesh-enabled clients (like our test client) to the gateway will be secured with mTLS.</p>
<h2>Managing CEL based RBAC and integrating exAuth with Kyverno into our Request FLow<span class="hx-absolute -hx-mt-20" id="managing-cel-based-rbac-and-integrating-exauth-with-kyverno-into-our-request-flow"></span>
    <a class="subheading-anchor" href="#managing-cel-based-rbac-and-integrating-exauth-with-kyverno-into-our-request-flow"></a></h2><p>While Istio Ambient gives us coarse authorization at L4, scenarios like header-based controls, API key validation, or integration with corporate IdPs require richer context. This is where <a href="https://kyverno.io" rel="noopener" target="_blank">Kyverno</a> enters the picture for the rest of the tutorial: it exposes an Envoy-compatible gRPC endpoint so that kgateway can delegate per-request decisions (ExtAuth) right before the traffic leaves the cluster, and later we’ll use the same engine to enforce configuration governance. kgateway’s External Authorization capability allows us to route every Ollama request through this Kyverno service.</p>
<p>kgateway allows us to easily delegate the authorization step for the Ollama API call to an external gRPC service, implementing a Zero Trust defense-in-depth strategy.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-YAML"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1  </span><span class="w"> </span><span class="c"># TrafficPolicy for ExtAuth + Corrected kgateway Resilience</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ollama-external-auth-policy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ollama-egress-route</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">retry</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">attempts</span><span class="p">:</span><span class="w"> </span><span class="m">3</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">perTryTimeout</span><span class="p">:</span><span class="w"> </span><span class="l">10s</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">statusCodes</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="m">503</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="m">504</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">timeouts</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">request</span><span class="p">:</span><span class="w"> </span><span class="l">30s</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">extAuth</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">extensionRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">kyverno-authz-server</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>The TrafficPolicy above does more than just hook up ExtAuth. It also adds the following policies:</p>
<ul>
<li><strong>Retries &amp; timeouts</strong> guarantee that kgateway will automatically re-attempt authorization calls if Kyverno is slow or briefly unavailable, instead of immediately failing user traffic.</li>
<li><strong>Fail-open vs fail-closed:</strong> The GatewayExtension controls this stance. In regulated environments we typically fail closed (default), but for low-risk demos we can set <code>failOpen: true</code> so traffic continues if Kyverno is down, keeping user flows alive while logging the gap.</li>
</ul>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-YAML"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1  </span><span class="w"> </span><span class="c"># GatewayExtension defines the Kyverno server endpoint</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">GatewayExtension</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">kyverno-authz-server</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">ExtAuth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">extAuth</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">failOpen</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">   </span><span class="c"># keep requests blocked if Kyverno is unavailable</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">grpcService</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">backendRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">kyverno-authz-server</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">9081</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>In production you would tune <code>failOpen</code> per criticality: customer-facing payment traffic might prefer <code>false</code>, whereas internal experimentation clusters might temporarily set it to <code>true</code> during policy rollouts.</p>
<h2>Authorization rules for Ollama traffic<span class="hx-absolute -hx-mt-20" id="authorization-rules-for-ollama-traffic"></span>
    <a class="subheading-anchor" href="#authorization-rules-for-ollama-traffic"></a></h2><p>With traffic flowing through ExtAuth now, you can define the actual rules Kyverno must evaluate. For this tutorial, set up a positive confirmation header before any prompt reaches the external model. The following Envoy AuthorizationPolicy expresses that business rule using CEL expressions:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-YAML"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion: envoy.kyverno.io/v1alpha1   # Kyverno AuthorizationPolicy</span><span class="p">:</span><span class="w"> </span><span class="l">RESTORING SECURITY (Conditional Access)</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AuthorizationPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">demo-policy.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">failurePolicy</span><span class="p">:</span><span class="w"> </span><span class="l">Fail</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">variables</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">force_authorized</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">expression</span><span class="p">:</span><span class="w"> </span><span class="l">object.attributes.request.http.?headers["x-force-authorized"].orValue("")</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">allowed</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">expression</span><span class="p">:</span><span class="w"> </span><span class="l">variables.force_authorized in ["enabled", "true"]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">authorizations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">expression</span><span class="p">:</span><span class="w"> </span><span class="p">&gt;</span><span class="sd">
</span></span></span><span class="line"><span class="cl"><span class="sd">      variables.allowed
</span></span></span><span class="line"><span class="cl"><span class="sd">        ? envoy.Allowed().Response()
</span></span></span><span class="line"><span class="cl"><span class="sd">        : envoy.Denied(403).WithBody("Access denied by Kyverno policy").Response()</span><span class="w">      
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Here’s what this means operationally:</p>
<ul>
<li><strong>Positive assertion header:</strong> Only requests that carry <code>x-force-authorized: true</code> or <code>enabled</code> are forwarded to Ollama. This mirrors how enterprises often want application teams to explicitly “tag” the traffic as compliant with business policies.</li>
<li><strong>Deterministic deny path:</strong> Anything else triggers a 403 with a human-readable reason so SREs can quickly debug missing headers.</li>
<li><strong>Failure policy:</strong> We keep <code>failurePolicy: Fail</code> so that malformed requests or Kyverno evaluation errors default to “deny,” ensuring the external model never sees unaudited requests.</li>
</ul>
<h1>Kyverno Configuration Policy/ Configuration Governance</h1><p>Runtime enforcement alone is not enough in real enterprises. Platform or security teams usually require an approval workflow before new egress endpoints are allowed. The following ClusterPolicy models that review step: every ServiceEntry must carry the <code>security.corp/egress-approved: &quot;true&quot;</code> label, signaling that an administrator verified the destination and its data-sharing risk.</p>
<p>In a corporate process, this label might be set only after a Jira ticket is approved or a risk assessment is completed. Without it, developers cannot onboard shadow endpoints, and Kyverno blocks the object at admission time.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-YAML"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">kyverno.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">ClusterPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">enforce-serviceentry-egress-label</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">validationFailureAction</span><span class="p">:</span><span class="w"> </span><span class="l">Enforce </span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">background</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w"> 
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">require-approved-egress-label</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">match</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">any</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span>- <span class="nt">resources</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">kinds</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"ServiceEntry"</span><span class="p">]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">names</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"ollama-external-host"</span><span class="p">]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">validate</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">message</span><span class="p">:</span><span class="w"> </span><span class="s2">"External ServiceEntries must include the label 'security.corp/egress-approved: true' to ensure policy review."</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">pattern</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">labels</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">security.corp/egress-approved</span><span class="p">:</span><span class="w"> </span><span class="s2">"?*"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h1>See it in Action/ Testing</h1><h3>Deploy the test client<span class="hx-absolute -hx-mt-20" id="deploy-the-test-client"></span>
    <a class="subheading-anchor" href="#deploy-the-test-client"></a></h3><p>To test the security policies applied by kgateway, use a simple Pod named curl-test-client. Its primary role is to serve as the mesh-enabled client that originates the outbound traffic, to test Layer 4 security (mTLS) and Layer 7 policies (CEL RBAC/ExtAuth). It is labeled for Ambient Mesh enrollment.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-YAML"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">apps/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Deployment</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">curl-test-client</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">labels</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">app</span><span class="p">:</span><span class="w"> </span><span class="l">curl-client</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">istio.io/dataplane-mode</span><span class="p">:</span><span class="w"> </span><span class="l">ambient</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">replicas</span><span class="p">:</span><span class="w"> </span><span class="m">1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">selector</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">matchLabels</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">app</span><span class="p">:</span><span class="w"> </span><span class="l">curl-client</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">template</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">labels</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">app</span><span class="p">:</span><span class="w"> </span><span class="l">curl-client</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">istio.io/dataplane-mode</span><span class="p">:</span><span class="w"> </span><span class="l">ambient</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">containers</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">client</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">image</span><span class="p">:</span><span class="w"> </span><span class="l">curlimages/curl</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">command</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"sleep"</span><span class="p">,</span><span class="w"> </span><span class="s2">"3600"</span><span class="p">]</span><span class="w"> 
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">imagePullPolicy</span><span class="p">:</span><span class="w"> </span><span class="l">IfNotPresent</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h3>Verify authorization policies for the Ollama service<span class="hx-absolute -hx-mt-20" id="verify-authorization-policies-for-the-ollama-service"></span>
    <a class="subheading-anchor" href="#verify-authorization-policies-for-the-ollama-service"></a></h3><p>We will use the client app that we previously deployed to execute tests against the <code>host.docker.internal</code> domain through the kgateway egress gateway.</p>
<p>Authorized Test (With Required Header)
This test should be allowed because the header x-force-authorized: true satisfies the Kyverno Envoy AuthorizationPolicy. The traffic is then proxied by kgateway to the Ollama container.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl">kubectl <span class="nb">exec</span> -it deploy/curl-test-client -n default -- curl http://egress-kgateway.default:8080/ -v -H <span class="s2">"Host: host.docker.internal"</span> -H <span class="s2">"x-force-authorized: true"</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Expected Output:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl">&lt; HTTP/1.1 <span class="m">200</span> OK
</span></span><span class="line"><span class="cl">...
</span></span><span class="line"><span class="cl">Ollama is running%</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>If we would not include the true or enabled label with our test command then we will receive an error with 403 Forbidden as:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>&lt; HTTP/1.1 403 Forbidden
&lt; content-type: text/plain
Access denied by Kyverno policy</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h3>Verify retry policies on the Kyverno server<span class="hx-absolute -hx-mt-20" id="verify-retry-policies-on-the-kyverno-server"></span>
    <a class="subheading-anchor" href="#verify-retry-policies-on-the-kyverno-server"></a></h3><p>While the TrafficPolicy defines the retry logic (attempts: 3, on 503 or 504), we must prove that the kgateway egress gateway actually executes the retries when an upstream service fails. We will simulate an upstream connection failure by temporarily pausing the external Ollama container.
To test out an error, lets stop our Docker Ollama-server:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>docker pause ollama-server
echo "Ollama container is paused. Proceeding to send request..."</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Now Let&rsquo;s try sending our request to ollama-server which is stopped thus unreachable:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>kubectl exec -it deploy/curl-test-client -n default -- curl http://egress-kgateway.default:8080/ -v -H "Host: host.docker.internal" -H "x-force-authorized: true"</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Send a request. kgateway will try three times (as configured by attempts: 3 in the TrafficPolicy) before ultimately failing and returning a 503 Service Unavailable or a 504 Gateway Timeout error to the client like:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>&lt; HTTP/1.1 504 Gateway Timeout
&lt; content-type: text/plain
upstream request timeout</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Now let&rsquo;s look at the logs from the kgateway egress gateway pod. We look for the final access log entry, which must contain the URX and UF flags to prove the retries occurred.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl"><span class="nv">KGW_POD</span><span class="o">=</span><span class="k">$(</span>kubectl get pods -n default -l gateway.networking.k8s.io/gateway-name<span class="o">=</span>egress-kgateway -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.items[0].metadata.name}'</span><span class="k">)</span>   <span class="c1"># Get the pod name for the egress-kgateway</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl">kubectl logs <span class="nv">$KGW_POD</span> -n default <span class="p">|</span> grep <span class="s1">'POST /api/generate'</span> <span class="p">|</span> tail -n <span class="m">1</span>   <span class="c1"># View the last request log entry</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>You should see an output similar to the following:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>{
  "response_flags": "URX,UF",
  "upstream_transport_failure_reason": "delayed connect error: Connection refused",
  "attempt_count": 2
}</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h1>Blog&rsquo;s Technical Conclusion</h1><p>While Istio Ambient Mesh simplifies service-to-service security by removing sidecars and introducing a layered data plane model, this guide extended its capabilities to demonstrate how kgateway strengthens that foundation with fine-grained policy control, external authorization, and governance enforcement. This paves the way for platform teams to confidently manage both north-south and east-west traffic using a single control model — all while keeping the lightweight operational benefits that Ambient Mesh was designed for.
Through this Blog we addressed one of the most overlooked challenges in service mesh deployments on <strong>How can we extend mesh-level security beyond internal workloads while ensuring Outbound traffic follows the same policy rigor, auditability, and compliance as in-cluster communication.</strong></p>
<h1>Demo</h1><p><a href="https://youtu.be/5PegECeu0v0" rel="noopener" target="_blank">Demo</a></p>


    
    <div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
      
    </div>

<h1>My LFX Mentorship Experience</h1><p>It&rsquo;s always great to contribute to a project and gain skills, experience, expertise and knowledge while working on certain different features of that project, but one of the most important part of contributing to an open source or being a part of an amazing project like kgateway is building up a strong confidence towards the future of the project and building up a commitement for the future to keep contributing as much value back to the project, to keep project&rsquo;s growth gain a consistent or exponential growth sustainably over the interval.</p>
<p>This is only possible with project&rsquo;s strong aim to solve problems in the most required format and with a strong community which is committed to keep the things <strong>Up and Running</strong>, inspired in a direction to solve issues that really messes around with the minds and tasks of the Developers but still remains ignored amid its complexity.</p>
<p>For a Mentee, his Mentors are the faces of the entire project&rsquo;s community, on which they are working! Thus, it would&rsquo;nt have been possible without the help of my Mentors - <strong>Nina Polshakova</strong> who kept helping me in the journey of this entire mentorship with every PR added &amp; weekly targets, while contributing constantly to the latest Release v2.1 of kgateway with its first and biggest integration with agentgateway, and <strong>Lin Sun</strong> who helped me and the presiding the entire open source community while managing projects like agentgateway and kagent while working on kgateway&rsquo;s latest release v2.1!</p>
<p>Community at <strong>Kgateway</strong> have been constantly focussing to solve issues which revolves around integration, observability, Security, AI collaboration and Governance implementing the Kubernetes Gateway API with a control plane that scales from lightweight microgateway deployments between services, to massively parallel centralized gateways handling billions of API calls. Community have been markign a huge impact by managing projects that can really impact the Workload orchestration and management through projects like:</p>
<ul>
<li><strong>kgateway</strong> to focus on control plane while integrating with Envoy-based data plane for secure API management of ingress, egress and service mesh.</li>
<li><strong>agentgateway</strong> to provide an AI-first enterprise data plane for connectivity across agents, MCP tools, LLMs and interferences written in Rust.</li>
</ul>
<h2>Community Interaction &amp; Participation<span class="hx-absolute -hx-mt-20" id="community-interaction--participation"></span>
    <a class="subheading-anchor" href="#community-interaction--participation"></a></h2><p>It&rsquo;s always a good idea to go through the project and interract with the community before applying, but many menntees get too excited here and staright away apply for the mentorship which might highlight add some lack of interest in the project in Future.</p>
<p>For me, <strong>the value that the project have been adding to the entire landscape matters the most</strong>. kgateway attracted my attention with its contributions, which made me to go through the documentation and even make a <a href="https://www.youtube.com/watch?v=RWQbUvVBUTI" rel="noopener" target="_blank">YouTube Video</a>, explaining about Architecture of kgateway&rsquo;s Control Plane &amp; Data Plane components, and Project&rsquo;s Deployment Patterns, before the mentorship was even announced!</p>
<p>For collaborating with the community Members, I tied attending Community Meetings and even organized a <a href="https://www.youtube.com/watch?v=raWq9Q0Pmws" rel="noopener" target="_blank">Podcast</a> with my Mentor <strong>Lin Sun</strong> on my <a href="https://www.youtube.com/@AryanParashar_" rel="noopener" target="_blank">YouTube channel</a> on the ocassion of kagteway getting accepted as a CNCF Sandbox Project, where I tried understanding about the kgateway community&rsquo;s future goals, its capabilities which sets it apart from other gateway projects and get to know about the experiences of the kgateway&rsquo;s community members who have have been leading and contributing to the entire Kubernetes Project!</p>
<p>These community collaborations and experiences attracted my attention to feel that working closely and contributing to this Project can really increase my experience and then LFX Mentorship came up as the best opportunity for me to get myself involved deeply within the Project and it&rsquo;s capabilities!</p>
<h2>Contributions to be Made during Entire Mentorship<span class="hx-absolute -hx-mt-20" id="contributions-to-be-made-during-entire-mentorship"></span>
    <a class="subheading-anchor" href="#contributions-to-be-made-during-entire-mentorship"></a></h2><p>Working on a Mentorship Project can be really exciting that has been bound to the aim of a mentorship, but it should also be kept in our mind that we are contributing in an Open Source Project and our mentorship aim is a part of it which includes:</p>
<ol>
<li><strong>Deeply Understand our Project</strong>: Go through the Documentation.</li>
<li><strong>Finding Bugs</strong>: Run through the entire Documentation and run codes in your own playground to build experience.</li>
<li><strong>Fixing Bugs</strong>: Open Issues, Merge PRs to solve bugs and improve End user experience.</li>
<li><strong>Improve Documentation &amp; Add Blogs</strong>: To make the Learning Curve less Steeper and easier to understand.</li>
<li><strong>Reflect your Technical Experience</strong>: prove your technical expertise through actual code contributions.</li>
<li><strong>Participation in Community</strong>: Participate in Community&rsquo;s slack channel, Community Meetings to understand the future goals, Present requirements and problems that users are experiencing and maintain Regularly sync progress with mentors</li>
</ol>
<h3>My Contributions made during Mentoship<span class="hx-absolute -hx-mt-20" id="my-contributions-made-during-mentoship"></span>
    <a class="subheading-anchor" href="#my-contributions-made-during-mentoship"></a></h3><p>During this mentorship, I had the opportunity to work across several areas of the kgateway ecosystem, contributing through hands-on development, documentation improvements, and technical deep dives including:</p>
<ol>
<li>Documnetation for ServiceEntries management with kgateway and Istio.</li>
<li>Fixing Bugs in <strong>exAuth</strong> Policies of kgateway.</li>
<li>Opening Issues related to Docuemntation improvements like <strong>Traffic Management with gRPC services, GAMMA Integration, exAuth, etc</strong>.</li>
<li>Contributed to kgateway&rsquo;s latest version <strong>v2.1</strong> Release Blog.</li>
<li>Building Security integration of kgateway with Istio and Kyverno for its exAuth and CEL based Authz testing policies.</li>
<li>Running Demo for Security integration of kgateway with Istio &amp; Kyverno.</li>
<li>Contributing to <strong>Argo Rollouts</strong> integration to kgateway, while using <strong>agentgateway</strong> as its gatewayClass.</li>
<li>Advocating for kgateway project through a demo video on my Youtube channel with its agentgaetway integration in version v2.1 and MCP walkthrough.</li>
</ol>
<h2>Mentorship Conclusion<span class="hx-absolute -hx-mt-20" id="mentorship-conclusion"></span>
    <a class="subheading-anchor" href="#mentorship-conclusion"></a></h2><p>My LFX Mentorship with the kgateway community has been much more than completing weekly tasks or shipping a handful of PRs — it has fundamentally shaped how I think about open-source collaboration, real-world infrastructure design, and the responsibility that comes with contributing to a CNCF ecosystem project. This journey helped me understand the true depth of modern API gateways, the evolving role of Ambient Mesh, and how governance, observability, security, and automation must work together to create reliable platforms.</p>
<p>Most importantly, this mentorship strengthened my confidence as an engineer. Through constant guidance from my mentors, code reviews, architecture discussions, community meetings, and hands-on problem solving, I learned how to approach complex challenges with clarity, break problems into solvable pieces, and design solutions that serve both developers and platform teams. I also gained an appreciation for the culture of open source — where ideas are welcomed, collaboration is encouraged, and every contribution, big or small, moves the project forward.</p>
<p>The experience I gained around Gateway API, Envoy, kgateway, agentgateway, Argo Rollouts and Istio Ambient Mesh will continue to influence my career for years to come. But beyond the technical depth, the mentorship taught me the value of community, communication, and consistency.
This mentorship has been one of the most meaningful steps in my cloud-native journey. Working closely with the kgateway community gave me the opportunity to contribute to real features, solve practical problems, and understand how modern API gateways are evolving around security, governance, and interoperability. From improving documentation and debugging exAuth policies to helping with the v2.1 release, building security demos, and integrating Argo Rollouts with agentgateway. I am deeply grateful to my mentors, the kgateway community, and the CNCF for trusting me with this opportunity and for helping me grow as an open-source contributor.</p>
<p>This is not the end — just the beginning. I will be continuing my regular contributions, sharing what I’ve learned with the community, and helping the <strong>kgateway</strong> Project evolve as it grows. The journey has been inspiring, and I’m really excited to keep building, collaborating, and contributing to the future of <strong>kgateway, aganetgateway</strong> and all cloud-native technologies.</p>
