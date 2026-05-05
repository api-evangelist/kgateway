---
title: "kgateway v2.1 release blog"
url: "/blog/kgateway-v2.1-release-blog/"
date: "Fri, 10 Oct 2025 10:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>We’re excited to announce the release kgateway v2.1, a release packed with exciting new features and improvements. Here are a few select updates the kgateway team would like to highlight!</p>
<h2>🌟 What&rsquo;s new in kgateway 2.1?<span class="hx-absolute -hx-mt-20" id="-whats-new-in-kgateway-21"></span>
    <a class="subheading-anchor" href="#-whats-new-in-kgateway-21"></a></h2><h3>Agentgateway integration<span class="hx-absolute -hx-mt-20" id="v21-agentgateway"></span>
    <a class="subheading-anchor" href="#v21-agentgateway"></a></h3><p>This release marks a major milestone — it’s the first version of integrating the open source project <a href="https://agentgateway.dev/" rel="noopener" target="_blank">agentgateway</a>! Agentgateway is a highly available, highly scalable, and enterprise-grade data plane that provides AI connectivity for LLMs, MCP tools, AI agents, and inference workloads. As part of this evolution, we’re beginning the deprecation of the Envoy-based AI Gateway and Envoy-based Inference Extension, since all related functionality is now implemented natively through agentgateway. You can still continue to use Envoy-based Gateways for API Gateway use cases.</p>
<p>For this release, agentgateway support is released as a beta. If you’re trying out the <code>agentgateway</code> <code>GatewayClass</code>, we recommend following the beta release feed to stay up to date with improvements, bug fixes, and breaking changes as the implementation is refined.</p>
<p>To get started with agentgateway, you simply install kgateway with the following Helm values:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">agentgateway</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">enabled</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Then you create a Gateway with the <code>agentgateway</code> <code>GatewayClass</code> as shown here:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="l">kubectl apply -f- &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway-system</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">labels</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">app</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">gatewayClassName</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">8080</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">allowedRoutes</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">namespaces</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">from</span><span class="p">:</span><span class="w"> </span><span class="l">All</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>You are now ready to try out agentgateway. Check out the <a href="/docs/envoy/2.1.x/ai/">agentgateway guides</a> to learn how to route traffic to an LLM provider, MCP tool server, or AI agent.</p>
<h3>K8s GW API 1.3.0 and Inference Extension 1.0.0<span class="hx-absolute -hx-mt-20" id="k8s-gw-api-130-and-inference-extension-100"></span>
    <a class="subheading-anchor" href="#k8s-gw-api-130-and-inference-extension-100"></a></h3><p>Kgateway is now fully conformant with the Kubernetes Gateway API version 1.3.0 and Inference Extension version 1.0.0. To learn more, check out the conformance test reports:</p>
<ul>
<li><a href="https://github.com/kubernetes-sigs/gateway-api/tree/main/conformance/reports/v1.3.0/kgateway" rel="noopener" target="_blank">Kubernetes Gateway API</a></li>
<li><a href="https://github.com/kubernetes-sigs/gateway-api-inference-extension/tree/main/conformance/reports/v1.0.2/gateway/kgateway" rel="noopener" target="_blank">Inference Extension</a></li>
</ul>
<h3>Global policy attachment<span class="hx-absolute -hx-mt-20" id="v21-global-policy-attachment"></span>
    <a class="subheading-anchor" href="#v21-global-policy-attachment"></a></h3><p>By default, you must attach policies to resources that are in the same namespace. Now, you can enable a feature to create a &ldquo;global&rdquo; namespace for policies. Then, these global policies can attach to resources in any namespace in your cluster through label selectors. For more information, see the <a href="/docs/envoy/latest/about/policies/global-attachment/">Global policy attachment</a> docs.</p>
<h3>Weighted routing<span class="hx-absolute -hx-mt-20" id="v21-weighted-routing"></span>
    <a class="subheading-anchor" href="#v21-weighted-routing"></a></h3><p>Now, you can configure weights for more fine-grained control over your routing rules. This feature is disabled by default. To enable it, see the <a href="/docs/envoy/latest/traffic-management/weighted-routes/">Weighted routing</a> docs.</p>
<h3>Deep merging for extauth and extproc policies<span class="hx-absolute -hx-mt-20" id="deep-merge"></span>
    <a class="subheading-anchor" href="#deep-merge"></a></h3><p>You can now apply deep merging for extAuth and extProc policies. In addition, you can use the <code>kgateway.dev/policy-weight</code> annotation to determine the priority in which multiple extAuth and extProc policies are merged. For more information, see <a href="/docs/envoy/latest/about/policies/merging/#merging-annotation">Policy priority during merging</a>.</p>
<h3>Additional proxy pod template customization<span class="hx-absolute -hx-mt-20" id="podtemplate"></span>
    <a class="subheading-anchor" href="#podtemplate"></a></h3><p>Kgateway now has more options to customize the gateway proxies&rsquo; default pod template, including configuration for <code>nodeSelectors</code>,<code>affinity</code>, <code>tolerations</code>, <code>topologySpreadConstraints</code>, and <code>externalTrafficPolicy</code>.</p>
<p>For more information, see <a href="/docs/envoy/latest/setup/customize/">Customize the gateway</a>. To find all the values that you can change, see the <a href="/docs/envoy/latest/reference/api/#pod">PodTemplate reference</a> in the GatewayParameters API.</p>
<h3>Header modifier filter for TrafficPolicy<span class="hx-absolute -hx-mt-20" id="header-modifier"></span>
    <a class="subheading-anchor" href="#header-modifier"></a></h3><p>Now, you can apply header request and response modifiers in a TrafficPolicy. This way, you get more flexible policy attachment options such as a gateway-level policy. For more information, see the <a href="/docs/envoy/latest/traffic-management/header-control/">Header control</a> docs. Note that this feature is available only for Envoy-based kgateway proxies, not the agentgateway proxy.</p>
<h3>Horizontal Pod Autoscaling<span class="hx-absolute -hx-mt-20" id="hpa"></span>
    <a class="subheading-anchor" href="#hpa"></a></h3><p>You can bring your own Horizontal Pod Autoscaler (HPA) plug-in to kgateway. This way, you can automatically scale kgateway control and data plane pods up and down based on certain thresholds, like memory and CPU consumption. See <a href="/docs/envoy/latest/setup/hpa/">Horizontal Pod Autoscaling (HPA)</a> for more information.</p>
<h3>HTTP1.0/0.9 support<span class="hx-absolute -hx-mt-20" id="http10"></span>
    <a class="subheading-anchor" href="#http10"></a></h3><p>Configure your gateway proxy to <a href="/docs/envoy/latest/setup/http10/">accept the HTTP/1.0 and HTTP/0.9 protocols</a> so that you can support legacy applications.</p>
<h3>Dynamic Forward Proxy<span class="hx-absolute -hx-mt-20" id="dfp"></span>
    <a class="subheading-anchor" href="#dfp"></a></h3><p>You can now configure the gateway proxy to use a Dynamic Forward Proxy (DFP) filter. This filter allows the proxy to act as a generic HTTP(S) forward proxy without the need to preconfigure all possible upstream hosts. Instead, the DFP dynamically resolves the upstream host at request time by using DNS. Check out <a href="/docs/envoy/latest/traffic-management/dfp/">Dynamic Forward Proxy (DFP)</a> for more information.</p>
<h3>Session affinity<span class="hx-absolute -hx-mt-20" id="session-affinity"></span>
    <a class="subheading-anchor" href="#session-affinity"></a></h3><p>You can now configure different types of session affinity for your Envoy-based gateway proxies:</p>
<ul>
<li><a href="/docs/envoy/latest/traffic-management/session-affinity/loadbalancing/">Change the default loadbalancing algorithm</a>: By default, incoming requests are forwarded to the instance with the least requests. You can change this behavior and instead use a round robin or random algorithm to forward the request to a backend service.</li>
<li><a href="/docs/envoy/latest/traffic-management/session-affinity/consistent-hashing/">Consistent hashing</a>: Set up soft session affinity between a client and a backend service by using consistent hashing algorithms.</li>
<li><a href="/docs/envoy/latest/traffic-management/session-affinity/session-persistence/">Session persistence</a>: Set up “strong” session affinity or sticky sessions to ensure that traffic from a client is always routed to the same backend instance for the duration of a session.</li>
</ul>
<h3>Enhanced retries and timeout capabilities<span class="hx-absolute -hx-mt-20" id="retries-timeouts"></span>
    <a class="subheading-anchor" href="#retries-timeouts"></a></h3><p>Retries and timeout capabilities were enhanced for your Envoy-based gateway proxies. Check out the following guides for more information:</p>
<ul>
<li><a href="/docs/envoy/latest/resiliency/retry/retry/">Request retries</a></li>
<li><a href="/docs/envoy/latest/resiliency/timeouts/request/">Request timeouts</a></li>
<li><a href="/docs/envoy/latest/resiliency/retry/per-try-timeout/">Per-try timeouts</a></li>
<li><a href="/docs/envoy/latest/resiliency/timeouts/idle/">Idle timeouts</a></li>
<li><a href="/docs/envoy/latest/resiliency/timeouts/idle-stream/">Idle stream timeouts</a></li>
</ul>
<h3>Passive health checks with outlier detection<span class="hx-absolute -hx-mt-20" id="outlier-detection"></span>
    <a class="subheading-anchor" href="#outlier-detection"></a></h3><p>You can now configure passive health checks and remove unhealthy hosts from the load balancing pool with an outlier detection policy. An outlier detection policy sets up several conditions, such as retries and ejection percentages, that kgateway uses to determine if a service is unhealthy. When an unhealthy service is detected, the outlier detection policy defines how the service is removed from the pool of healthy destinations to send traffic to. For more information, see <a href="/docs/envoy/latest/resiliency/outlier-detection/">Outlier detection</a>.</p>
<h3>New kgateway operations dashboard<span class="hx-absolute -hx-mt-20" id="kgateway-dashboard"></span>
    <a class="subheading-anchor" href="#kgateway-dashboard"></a></h3><p>When you install the <a href="/docs/envoy/latest/observability/otel-stack/">OTel stack</a>, you can now leverage the new kgateway operations dashboard for Grafana. This dashboard shows important metrics at a glance, such as the translation and reconciliation time, total number of operations, the number of resources in your cluster, and latency.</p>
<p>




<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/img/kgateway-dashboard.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div>





<div><figure class="hx-hidden dark:hx-block"><img alt="" class="hx-hidden dark:hx-block" src="/img/kgateway-dashboard.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div></p>
<h3>Leader election enabled<span class="hx-absolute -hx-mt-20" id="kgateway-dashboard"></span>
    <a class="subheading-anchor" href="#kgateway-dashboard"></a></h3><p>Leader election is now enabled by default to ensure that you can run kgateway in a multi-control plane replica setup for high availability.</p>
<p>You can disable leader election by setting the <code>controller.disableLeaderElection</code> to <code>true</code> in your Helm chart.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl">helm upgrade -i --namespace kgateway-system --version v2.1.0 kgateway oci://cr.kgateway.dev/kgateway-dev/charts/kgateway --set controller.disableLeaderElection<span class="o">=</span>true</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h2>🔥 Breaking changes from the previous release<span class="hx-absolute -hx-mt-20" id="-breaking-changes-from-the-previous-release"></span>
    <a class="subheading-anchor" href="#-breaking-changes-from-the-previous-release"></a></h2><h3>Kubernetes Gateway API version v1.4.0<span class="hx-absolute -hx-mt-20" id="kubernetes-gateway-api-version-v140"></span>
    <a class="subheading-anchor" href="#kubernetes-gateway-api-version-v140"></a></h3><p>Now, kgateway supports version 1.4.0 of the Kubernetes Gateway API. As part of this change, the BackendTLSPolicy API version in the experimental channel is promoted from <code>v1alpha3</code> to <code>v1</code>. Before you upgrade kgateway, make sure to upgrade the Kubernetes Gateway API to version 1.4.0.</p>
<div class="hextra-scrollbar hx-overflow-x-auto hx-overflow-y-hidden hx-overscroll-x-contain">
  <div class="hx-mt-4 hx-flex hx-w-max hx-min-w-full hx-border-b hx-border-gray-200 hx-pb-px dark:hx-border-neutral-800"><button class="hextra-tabs-toggle data-[state=selected]:hx-border-primary-500 data-[state=selected]:hx-text-primary-600 data-[state=selected]:dark:hx-border-primary-500 data-[state=selected]:dark:hx-text-primary-600 hx-mr-2 hx-rounded-t hx-p-2 hx-font-medium hx-leading-5 hx-transition-colors -hx-mb-0.5 hx-select-none hx-border-b-2 hx-border-transparent hx-text-gray-600 hover:hx-border-gray-200 hover:hx-text-black dark:hx-text-gray-200 dark:hover:hx-border-neutral-800 dark:hover:hx-text-white" tabindex="0" type="button">Standard</button><button class="hextra-tabs-toggle data-[state=selected]:hx-border-primary-500 data-[state=selected]:hx-text-primary-600 data-[state=selected]:dark:hx-border-primary-500 data-[state=selected]:dark:hx-text-primary-600 hx-mr-2 hx-rounded-t hx-p-2 hx-font-medium hx-leading-5 hx-transition-colors -hx-mb-0.5 hx-select-none hx-border-b-2 hx-border-transparent hx-text-gray-600 hover:hx-border-gray-200 hover:hx-text-black dark:hx-text-gray-200 dark:hover:hx-border-neutral-800 dark:hover:hx-text-white" type="button">Experimental</button></div>
</div>
<div>
<div class="hextra-tabs-panel hx-rounded hx-pt-6 hx-hidden data-[state=selected]:hx-block" id="tabs-panel-0" tabindex="0"><div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">
<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl">kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.0/standard-install.yaml</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</div>
<div class="hextra-tabs-panel hx-rounded hx-pt-6 hx-hidden data-[state=selected]:hx-block" id="tabs-panel-1"><div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">
<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl">kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.0/experimental-install.yaml</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</div>
</div>
<h3>AI Backend API changes<span class="hx-absolute -hx-mt-20" id="v21-ai-backend-api-changes"></span>
    <a class="subheading-anchor" href="#v21-ai-backend-api-changes"></a></h3><p>The AI Backend API is updated to simplify the configuration of various LLM features. For more information, see the <a href="/docs/envoy/latest/reference/api/#aibackend">API reference</a> and <a href="/docs/envoy/2.1.x/ai/">AI guides</a> docs.</p>
<h3>Route delegation annotation for policy merging<span class="hx-absolute -hx-mt-20" id="v21-delegation-policy-merging"></span>
    <a class="subheading-anchor" href="#v21-delegation-policy-merging"></a></h3><p>The route delegation feature for policy merging is expanded to reflect its broader role of applying not only to routes, but also to policies. This update includes the following changes:</p>
<ul>
<li>The annotation is renamed from <code>delegation.kgateway.dev/inherited-policy-priority</code> to the simpler <code>kgateway.dev/inherited-policy-priority</code>.</li>
<li>Now, the following values are accepted: <code>ShallowMergePreferParent</code> and <code>ShallowMergePreferChild</code></li>
<li>The default behavior of parent route policies taking precedence over child routes policies is reversed. Now, child routes take precedence, which aligns better with the precedence defaults across other resources in the kgateway and Gateway APIs.</li>
</ul>
<p>To maintain the previous default behavior of 2.0, update your annotations to <code>kgateway.dev/inherited-policy-priority: ShallowMergePreferParent</code>. For more information, check out the <a href="/docs/envoy/latest/about/policies/merging/">Policy merging</a> docs.</p>
<h3>Fail open policy for ExtProc providers<span class="hx-absolute -hx-mt-20" id="fail-open-policy-for-extproc-providers"></span>
    <a class="subheading-anchor" href="#fail-open-policy-for-extproc-providers"></a></h3><p>The default fail open policy for ExtProc providers changed from <code>false</code> to <code>true</code>. Because of that, requests are forwarded to the upstream service, even if the ExtProc server is unavailable. To change this policy, set the <code>spec.extProc.failOpen</code> field to <code>false</code> in your GatewayExtension resource.</p>
<h3>Helm changes for agentgateway<span class="hx-absolute -hx-mt-20" id="helm-changes-for-agentgateway"></span>
    <a class="subheading-anchor" href="#helm-changes-for-agentgateway"></a></h3><p>The Helm value to enable the agentgateway integration changed from <code>agentGateway</code> to <code>agentgateway</code>. To enable agentgateway, use the following values in your Helm chart:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">agentgateway</span><span class="p">:</span><span class="w"> 
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">enabled</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h3>Helm changes for waypoints<span class="hx-absolute -hx-mt-20" id="helm-changes-for-waypoints"></span>
    <a class="subheading-anchor" href="#helm-changes-for-waypoints"></a></h3><p>The kgateway waypoint integration is disabled by default. To enable the integration, use the following values in your Helm chart:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">waypoint</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">enabled</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h3><code>ai.llm.hostOverride.insecureSkipVerify</code> removed from Backend<span class="hx-absolute -hx-mt-20" id="aillmhostoverrideinsecureskipverify-removed-from-backend"></span>
    <a class="subheading-anchor" href="#aillmhostoverrideinsecureskipverify-removed-from-backend"></a></h3><p>The <code>insecureSkipVerify</code> flag was removed for AI Backends. To configure this option, use a <a href="/docs/envoy/latest/reference/api/#backendconfigpolicy">BackendConfigPolicy</a> instead.</p>
<h3>Disable per route policies<span class="hx-absolute -hx-mt-20" id="disable-per-route-policies"></span>
    <a class="subheading-anchor" href="#disable-per-route-policies"></a></h3><p>The configuration for disabling policies on a route changed. Previously, you used the <code>enablement</code> field, such as in <code>extAuth.enablement</code> to enable or disable a policy on a route. Now, you use the <code>disable</code> field instead as shown in the following example:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">disable-all-extauth-for-route-2-1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">infra</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">route-2</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">sectionName</span><span class="p">:</span><span class="w"> </span><span class="l">rule1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">extAuth</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">disable</span><span class="p">:</span><span class="w"> </span>{}<span class="w"> </span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Disabling policies can be applied to CORS, extAuth, extProc, and rate limit policies.</p>
<h2>See the new features in action!<span class="hx-absolute -hx-mt-20" id="see-the-new-features-in-action"></span>
    <a class="subheading-anchor" href="#see-the-new-features-in-action"></a></h2><p>Explore some of the new features from the v2.1 release or follow along with the <a href="https://github.com/AdminTurnedDevOps/agentic-demo-repo/tree/main/agentgateway-oss-k8s/kgateway-agentgateway-2.1-key-features" rel="noopener" target="_blank">demo</a>.


    
    <div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
      
    </div>
</p>
<p>You can also check out the agentgateway and kgateway integration in action routing to <a href="https://github.com/AdminTurnedDevOps/agentic-demo-repo/tree/main/agentgateway-oss-k8s/a2a-k8s" rel="noopener" target="_blank">Agent-to-Agent (A2A) workloads</a> and <a href="https://github.com/AdminTurnedDevOps/agentic-demo-repo/tree/main/agentgateway-oss-k8s/mcp-connection-k8s-agentgateway" rel="noopener" target="_blank">Model Context Protocol (MCP) servers</a>:


    
    <div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
      
    </div>
</p>
<h2>🗑️ Deprecated or removed features<span class="hx-absolute -hx-mt-20" id="-deprecated-or-removed-features"></span>
    <a class="subheading-anchor" href="#-deprecated-or-removed-features"></a></h2><p>AI Gateway and Inference Extension support for Envoy-based gateway proxies is deprecated and is planned to be removed in version 2.2. If you want to use AI capabilities, use an <a href="/docs/envoy/2.1.x/ai/">agentgateway proxy</a> instead. To learn more about why we think that agentgateway is better suited as a gateway for agentic AI and MCP workloads, check out this <a href="https://www.solo.io/blog/why-do-we-need-a-new-gateway-for-ai-agents" rel="noopener" target="_blank">blog</a>.</p>
<h2>Release notes<span class="hx-absolute -hx-mt-20" id="release-notes"></span>
    <a class="subheading-anchor" href="#release-notes"></a></h2><p>Check out the full details of the kgateway v1.2 release in our <a href="https://kgateway.dev/docs/envoy/latest/reference/release-notes/" rel="noopener" target="_blank">release notes</a>.</p>
<h2>Availability<span class="hx-absolute -hx-mt-20" id="availability"></span>
    <a class="subheading-anchor" href="#availability"></a></h2><p>Ready to get started? Download the latest release on <a href="https://github.com/kgateway-dev/kgateway/releases" rel="noopener" target="_blank">GitHub</a>. Then, check out our <a href="https://kgateway.dev/docs/envoy/latest/quickstart/" rel="noopener" target="_blank">getting started guide</a> to install kgateway.</p>
<h2>Get Involved<span class="hx-absolute -hx-mt-20" id="get-involved"></span>
    <a class="subheading-anchor" href="#get-involved"></a></h2><p>The simplest way to get involved with kgateway is by joining our <a href="https://kgateway.dev/slack/" rel="noopener" target="_blank">slack</a> and <a href="https://github.com/kgateway-dev/community?tab=readme-ov-file#community-meetings" rel="noopener" target="_blank">community meetings</a>.</p>
<p>Thank you for your continued feedback and support!</p>
