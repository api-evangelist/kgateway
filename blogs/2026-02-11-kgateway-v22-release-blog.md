---
title: "kgateway v2.2 release blog"
url: "/blog/kgateway-v2.2-release-blog/"
date: "Wed, 11 Feb 2026 10:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>Kgateway v2.2 is packed with exciting new features and improvements. Here are a few select updates the kgateway team would like to highlight!</p>
<p>This release introduces major breaking changes to the agentgateway implementation. We have a new installation UX and new dedicated APIs. If you are currently running agentgateway with kgateway, please refer to our <a href="https://github.com/kgateway-dev/kgateway/blob/main/docs/guides/agentgateway-migration.md" rel="noopener" target="_blank">migration guide</a>.</p>
<h2>🔥Breaking changes<span class="hx-absolute -hx-mt-20" id="breaking-changes"></span>
    <a class="subheading-anchor" href="#breaking-changes"></a></h2><h3>Agentgateway-specific resources and Helm charts<span class="hx-absolute -hx-mt-20" id="agentgateway-specific-resources-and-helm-charts"></span>
    <a class="subheading-anchor" href="#agentgateway-specific-resources-and-helm-charts"></a></h3><p>This release separated kgateway and agentgateway controllers and introduced several agentgateway-specific resources, Helm charts, and controllers. Kgateway-specific resources were not changed.</p>
<div class="hx-overflow-x-auto hx-mt-6 hx-flex hx-rounded-lg hx-border hx-py-2 ltr:hx-pr-4 rtl:hx-pl-4 contrast-more:hx-border-current contrast-more:dark:hx-border-current hx-border-blue-200 hx-bg-blue-100 hx-text-blue-900 dark:hx-border-blue-200/30 dark:hx-bg-blue-900/30 dark:hx-text-blue-200">
  <div class="ltr:hx-pl-3 ltr:hx-pr-2 rtl:hx-pr-3 rtl:hx-pl-2"><div class="hx-select-none hx-text-xl" style="font-family: 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol';">ℹ️</div></div>

  <div class="hx-w-full hx-min-w-0 hx-leading-7">
    <div class="hx-mt-6 hx-leading-7 first:hx-mt-0">Note: If you used agentgateway with kgateway, you <strong>must update and migrate your agentgateway-specific configuration to the new resources</strong>. Agentgateway-specific fields in kgateway resources, such as TrafficPolicy or GatewayParameters were deprecated. You cannot upgrade your environment without migrating your resources first!</div>
  </div>
</div>

<p>The following table includes the new agentgateway-specific resources that were introduced in this release and compares them to kgateway. Make sure to migrate your agentgateway configuration to these new resources.</p>
<table>
  <thead>
      <tr>
          <th style="text-align: left;">Resource</th>
          <th style="text-align: left;">Agentgateway vs kgateway</th>
          <th style="text-align: left;">Description of change</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;">GatewayClass</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>agentgateway</li></ul><b>kgateway</b>: <ul><li>kgateway</li></ul></td>
          <td style="text-align: left;">Adds agentgateway-specific GatewayClass.</td>
      </tr>
      <tr>
          <td style="text-align: left;">Controller name</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>agentgateway.dev/agentgateway</li></ul><b>kgateway</b>: <ul><li>kgateway.dev/kgateway</li></ul></td>
          <td style="text-align: left;">Adds agentgateway-specific controller. <b>Related PRs:<ul><li><a href="https://github.com/kgateway-dev/kgateway/pull/13088">13088</a></li></ul></td>
      </tr>
      <tr>
          <td style="text-align: left;">Control plane deployment name</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>agentgateway</li></ul><b>kgateway</b>: <ul><li>kgateway</li></ul></td>
          <td style="text-align: left;">Adds agentgateway-specific control plane name.</td>
      </tr>
      <tr>
          <td style="text-align: left;">Default namespace</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>agentgateway-system</li></ul><b>kgateway</b>: <ul><li>kgateway-system</li></ul></td>
          <td style="text-align: left;">Updates docs to use agentgateway-specific namespace.</td>
      </tr>
      <tr>
          <td style="text-align: left;">CRD Helm chart location</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>oci://cr.agentgateway.dev/charts/agentgateway-crds</li></ul><b>kgateway</b>: <ul><li>oci://cr.kgateway.dev/kgateway-dev/charts/kgateway-crds</li></ul></td>
          <td style="text-align: left;">Adds agentgateway-specific CRD Helm charts. <b>Related PRs:<ul><li><a href="https://github.com/kgateway-dev/kgateway/pull/13062">13062</a></li><li><a href="https://github.com/kgateway-dev/kgateway/pull/12960">12960</a></li></ul></td>
      </tr>
      <tr>
          <td style="text-align: left;">Controller Helm chart location</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>oci://cr.agentgateway.dev/charts/agentgateway</li></ul><b>kgateway</b>: <ul><li>oci://cr.kgateway.dev/kgateway-dev/charts/kgateway</li></ul></td>
          <td style="text-align: left;">Adds agentgateway-specific controller Helm charts. <b>Related PRs:<ul><li><a href="https://github.com/kgateway-dev/kgateway/pull/13062">13062</a></li><li><a href="https://github.com/kgateway-dev/kgateway/pull/12960">12960</a></li></ul></td>
      </tr>
      <tr>
          <td style="text-align: left;">CRDs</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>AgentgatewayPolicy</li><li>AgentgatewayBackend</li><li>AgentgatewayParameters</li></ul><b>kgateway</b>: <ul><li>TrafficPolicy</li><li>Backend</li><li>GatewayParameters</li></ul></td>
          <td style="text-align: left;">Introduces agentgateway-specific custom resource.  Removes AI policy from TrafficPolicy.  Adds DirectResponse support to AgentgatewayPolicy.  Allows agentgateway proxy-specific parameters via AgentgatewayParameters resource only.<b>Related PRs:<ul><li><a href="https://github.com/kgateway-dev/kgateway/pull/12901">12901</a></li><li><a href="https://github.com/kgateway-dev/kgateway/pull/12723">12723</a></li><li><a href="https://github.com/kgateway-dev/kgateway/pull/13054">13054</a></li><li><a href="https://github.com/kgateway-dev/kgateway/pull/13018">13018</a></li><li><a href="https://github.com/kgateway-dev/kgateway/pull/13101">13101</a></li></ul></td>
      </tr>
      <tr>
          <td style="text-align: left;">API version in CRDs</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>agentgateway.dev/v1alpha1</li></ul><b>kgateway</b>: <ul><li>kgateway.dev/v1alpha1</li></ul></td>
          <td style="text-align: left;">Updates agentgateway resources to use the new <code>agentgateway.dev</code> group. <b>Related PRs:<ul><li><a href="https://github.com/kgateway-dev/kgateway/pull/13013">13013</a></li></ul></td>
      </tr>
      <tr>
          <td style="text-align: left;">Group in CRDs</td>
          <td style="text-align: left;"><b>agentgateway</b>: <ul><li>agentgateway.dev</li></ul><b>kgateway</b>: <ul><li>gateway.kgateway.dev</li></ul></td>
          <td style="text-align: left;">Updates agentgateway resources to use the new <code>agentgateway.dev</code> group.</td>
      </tr>
  </tbody>
</table>
<h3>Agentgateway-specific documentation moved<span class="hx-absolute -hx-mt-20" id="agentgateway-specific-documentation-moved"></span>
    <a class="subheading-anchor" href="#agentgateway-specific-documentation-moved"></a></h3><p>With the introduction of agentgateway-specific resources and Helm charts, the agentgateway documentation moved to the <a href="https://agentgateway.dev/docs/kubernetes" rel="noopener" target="_blank">agentgateway.dev</a> org.</p>
<p>Kgateway-specific documentation for Envoy gateways did not move and continues to be accessible via the <a href="https://kgateway.dev/docs/envoy" rel="noopener" target="_blank">kgateway.dev org</a>.</p>
<h3><code>KGW_ENABLE_EXPERIMENTAL_GATEWAY_API_FEATURES</code> to gate experimental Gateway API features and APIs<span class="hx-absolute -hx-mt-20" id="kgw_enable_experimental_gateway_api_features-to-gate-experimental-gateway-api-features-and-apis"></span>
    <a class="subheading-anchor" href="#kgw_enable_experimental_gateway_api_features-to-gate-experimental-gateway-api-features-and-apis"></a></h3><p>Use the <code>--set controller.extraEnv.KGW_ENABLE_GATEWAY_API_EXPERIMENTAL_FEATURES=true</code> setting in your Helm installation to enable experimental Kubernetes Gateway API features and APIs, such as the following:</p>
<ul>
<li>XListenerSet</li>
<li>Route SessionPersistence</li>
<li>HTTPCORSFilter</li>
<li>HTTPRouteRetry</li>
</ul>
<p>By default, the <code>KGW_ENABLE_EXPERIMENTAL_GATEWAY_API_FEATURES</code> is set to <code>false</code>. For more information, see the related <a href="https://github.com/kgateway-dev/kgateway/pull/12695" rel="noopener" target="_blank">kgateway PR</a>.</p>
<p>For setup steps, see the get started guide in the <a href="https://kgateway.dev/docs/envoy/latest/quickstart/" rel="noopener" target="_blank">kgateway</a> or <a href="https://agentgateway.dev/docs/kubernetes/latest/quickstart/" rel="noopener" target="_blank">agentgateway</a> docs.</p>
<h3>Agentgateway ExtAuth policies fail closed<span class="hx-absolute -hx-mt-20" id="agentgateway-extauth-policies-fail-closed"></span>
    <a class="subheading-anchor" href="#agentgateway-extauth-policies-fail-closed"></a></h3><p>Agentgateway external auth policies now fail closed when the auth server that is referenced in the backendRef is invalid. Previously, external auth policies failed open.</p>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/13273" rel="noopener" target="_blank">PR</a> for more information.</p>
<h3>AI prompt guard API alignment<span class="hx-absolute -hx-mt-20" id="ai-prompt-guard-api-alignment"></span>
    <a class="subheading-anchor" href="#ai-prompt-guard-api-alignment"></a></h3><p>The AI prompt guard API is updated to align with other enums. The values changed from <code>MASK</code> to <code>Mask</code> and <code>REJECT</code> to <code>Reject</code> as shown in the following example. These changes are enforced by CEL validation in the API.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">openai-prompt-guard</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway-system</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">labels</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">app</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">openai</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">backend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">ai</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">promptGuard</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">request</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span>- <span class="nt">response</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">message</span><span class="p">:</span><span class="w"> </span><span class="s2">"Rejected due to inappropriate content"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">regex</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">action</span><span class="p">:</span><span class="w"> </span><span class="l">Reject</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span>- <span class="s2">"credit card"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>For steps to set up prompt guards, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/llm/prompt-guards/" rel="noopener" target="_blank">docs</a>.</p>
<h2>🗑️ Deprecated or removed features<span class="hx-absolute -hx-mt-20" id="-deprecated-or-removed-features"></span>
    <a class="subheading-anchor" href="#-deprecated-or-removed-features"></a></h2><h3>HTTPListenerPolicy is deprecated<span class="hx-absolute -hx-mt-20" id="httplistenerpolicy-is-deprecated"></span>
    <a class="subheading-anchor" href="#httplistenerpolicy-is-deprecated"></a></h3><p>The HTTPListenerPolicy is now deprecated and is planned to be removed in future releases. If you currently use policies in the HTTPListenerPolicy resource, migrate them to the <code>httpSettings</code> under the ListenerPolicy.</p>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/13066" rel="noopener" target="_blank">PR</a> for more information.</p>
<p>To learn more about the ListenerPolicy, see the <a href="https://kgateway.dev/docs/envoy/latest/about/policies/listenerpolicy/" rel="noopener" target="_blank">docs</a>.</p>
<h3>PerConnectionBufferLimit annotation deprecated<span class="hx-absolute -hx-mt-20" id="perconnectionbufferlimit-annotation-deprecated"></span>
    <a class="subheading-anchor" href="#perconnectionbufferlimit-annotation-deprecated"></a></h3><p>The  PerConnectionBufferLimit annotation on Gateway resources is deprecated. Configure PerConnectionBufferLimit  on the ListenerPolicy instead. For an example, see the <a href="https://kgateway.dev/docs/envoy/latest/traffic-management/buffering/" rel="noopener" target="_blank">docs</a>.</p>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/13016" rel="noopener" target="_blank">PR</a> for more information.</p>
<h3><code>spec.kube.floatingUserId</code> field removed from GatewayParameters CRD<span class="hx-absolute -hx-mt-20" id="speckubefloatinguserid-field-removed-from-gatewayparameters-crd"></span>
    <a class="subheading-anchor" href="#speckubefloatinguserid-field-removed-from-gatewayparameters-crd"></a></h3><p>This field was previously used to unset runAsUser values in security contexts. When migrating, users should use the supported <code>spec.kube.omitDefaultSecurityContext</code> field instead. When set to true, this field prevents the controller from injecting opinionated default security contexts, allowing your platform (e.g., OCP) to dynamically provide the appropriate security contexts.</p>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12747" rel="noopener" target="_blank">PR</a> for more information.</p>
<h2>🌟New features<span class="hx-absolute -hx-mt-20" id="new-features"></span>
    <a class="subheading-anchor" href="#new-features"></a></h2><h3>Highlighted agentgateway features<span class="hx-absolute -hx-mt-20" id="highlighted-agentgateway-features"></span>
    <a class="subheading-anchor" href="#highlighted-agentgateway-features"></a></h3><h4>MCP authentication<span class="hx-absolute -hx-mt-20" id="mcp-authentication"></span>
    <a class="subheading-anchor" href="#mcp-authentication"></a></h4><p>MCP authentication enables OAuth 2.0 protection for MCP servers, helping to implement the MCP Authorization specification. Agentgateway can act as a resource server, validating JWT tokens and exposing protected resource metadata.</p>
<p>The MCP authentication policy can be attached to an MCP backend using the AgentgatewayPolicy or inlined on the AgentgatewayBackend:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">keycloak-mcp-authn-policy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">mcp-backend-static</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayBackend</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">backend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">mcp</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">authentication</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">issuer</span><span class="p">:</span><span class="w"> </span><span class="l">http://keycloak:7080/realms/mcp</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">jwks</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">jwksPath</span><span class="p">:</span><span class="w"> </span><span class="l">realms/mcp/protocol/openid-connect/certs</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">backendRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">keycloak</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">7080</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">audiences</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span>- <span class="s2">"account"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">provider</span><span class="p">:</span><span class="w"> </span><span class="l">Keycloak</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">resourceMetadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">resource</span><span class="p">:</span><span class="w"> </span><span class="l">http://mcp-website-fetcher.default.svc.cluster.local/mcp</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">scopesSupported</span><span class="p">:</span><span class="w"> </span><span class="s1">'["tools/call/fetch"]'</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">bearerMethodsSupported</span><span class="p">:</span><span class="w"> </span><span class="s1">'["header"]'</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">resourceDocumentation</span><span class="p">:</span><span class="w"> </span><span class="l">http://mcp-website-fetcher.default.svc.cluster.local/docs</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">resourcePolicyUri</span><span class="p">:</span><span class="w"> </span><span class="l">http://mcp-website-fetcher.default.svc.cluster.local/policies</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See the following PRs for more information:</p>
<ul>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/12966" rel="noopener" target="_blank">12966</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13111" rel="noopener" target="_blank">13111</a></li>
</ul>
<p>For setup steps, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/mcp/auth/" rel="noopener" target="_blank">docs</a>.</p>
<h4>Inline and remote JWKS support<span class="hx-absolute -hx-mt-20" id="inline-and-remote-jwks-support"></span>
    <a class="subheading-anchor" href="#inline-and-remote-jwks-support"></a></h4><p>You can now define both inline and remote JWKS endpoints to automatically fetch and rotate keys from your identity provider on the AgentgatewayPolicy. See this <a href="https://github.com/kgateway-dev/kgateway/pull/12850" rel="noopener" target="_blank">PR</a> for more information.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="l">kubectl apply -f - &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">jwt-auth-policy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway-system</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="c"># Target the Gateway to apply JWT authentication to all routes</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway-proxy   </span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="c"># Configure JWT authentication</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">traffic</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">jwtAuthentication</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="c"># Validation mode - determines how strictly JWTs are validated</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">mode</span><span class="p">:</span><span class="w"> </span><span class="l">Strict   </span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="c"># List of JWT providers (identity providers)</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">providers</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span>- <span class="c"># Issuer URL - must match the 'iss' claim in JWT tokens</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">issuer</span><span class="p">:</span><span class="w"> </span><span class="s2">"${KEYCLOAK_ISSUER}"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="c"># JWKS configuration for remote key fetching</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">jwks</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">remote</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="c"># Path to the JWKS endpoint, relative to the backend root</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">jwksPath</span><span class="p">:</span><span class="w"> </span><span class="s2">"${KEYCLOAK_JWKS_PATH}"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="c"># Cache duration for JWKS keys (reduces load on identity provider)</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">cacheDuration</span><span class="p">:</span><span class="w"> </span><span class="s2">"5m"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="c"># Reference to the Keycloak service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">backendRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">keycloak</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">keycloak</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">8080</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>You can also set TLS options when connecting to a remote JWKS source. See this <a href="https://github.com/kgateway-dev/kgateway/pull/13014" rel="noopener" target="_blank">PR</a> for more information.</p>
<p>To see an example in agentgateway, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/security/jwt/setup/" rel="noopener" target="_blank">docs</a>.</p>
<h4>Azure OpenAI backends<span class="hx-absolute -hx-mt-20" id="azure-openai-backends"></span>
    <a class="subheading-anchor" href="#azure-openai-backends"></a></h4><p>Agentgateway now natively supports Azure OpenAI backends:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayBackend</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ai-providers</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">ai</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">groups</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">providers</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">azure</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">azureopenai</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">endpoint</span><span class="p">:</span><span class="w"> </span><span class="l">test</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12836" rel="noopener" target="_blank">PR</a> for more information.</p>
<p>For setup steps, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/llm/providers/azureopenai/" rel="noopener" target="_blank">docs</a>.</p>
<h4>Model aliasing<span class="hx-absolute -hx-mt-20" id="model-aliasing"></span>
    <a class="subheading-anchor" href="#model-aliasing"></a></h4><p>The modelAliases field in the AgentgatewayPolicy now allows users to define friendly model name aliases that map to actual provider model names (e.g., &ldquo;fast&rdquo; can map to &ldquo;gpt-3.5-turbo&rdquo;).</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">backend-ai-prompt</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="s2">"agentgateway.dev"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayBackend</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">test</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">backend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">ai</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">modelAliases</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">fast</span><span class="p">:</span><span class="w"> </span><span class="l">gpt-3.5-turbo</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">smart</span><span class="p">:</span><span class="w"> </span><span class="l">gpt-4-turbo</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12479" rel="noopener" target="_blank">PR</a> for more information.</p>
<h4>CSRF<span class="hx-absolute -hx-mt-20" id="csrf"></span>
    <a class="subheading-anchor" href="#csrf"></a></h4><p>Cross-Site Request Forgery (CSRF) is an attack that tricks an authenticated user into unknowingly executing actions chosen by an attacker. While application frameworks commonly provide CSRF protections, agentgateway allows enforcing these defenses at a centralized access point, reducing the burden on individual application teams.</p>
<p>You can now configure CSRF policies using the traffic field in the AgentgatewayPolicy:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">csrf-gw-policy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway-base</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">gw</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">traffic</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">csrf</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">additionalOrigins</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="l">example.org</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12516" rel="noopener" target="_blank">PR</a> for more information.</p>
<p>For setup steps, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/security/csrf/" rel="noopener" target="_blank">docs</a>.</p>
<h4>Path-based API format routing (completions, messages, models, passthrough)<span class="hx-absolute -hx-mt-20" id="path-based-api-format-routing-completions-messages-models-passthrough"></span>
    <a class="subheading-anchor" href="#path-based-api-format-routing-completions-messages-models-passthrough"></a></h4><p>Agentgateway now supports explicit route typing and path-based routing to agentgateway backends, enabling a single backend to support multiple LLM API formats (OpenAI, Anthropic, models, and passthrough) based on request URL. It introduces a RouteType enum, path-to-type mappings, and supporting translation logic and tests to enable flexible multi-format API handling.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">agw</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">my-route</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">backend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">ai</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">routes</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">"/v1/chat/completions": </span><span class="l">Completions</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">"/v1/embeddings": </span><span class="l">Passthrough</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">"/v1/models": </span><span class="l">Passthrough</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">"*": </span><span class="l">Passthrough</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12590" rel="noopener" target="_blank">PR</a> for more information.</p>
<p>For setup steps, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/llm/providers/multiple-endpoints/" rel="noopener" target="_blank">docs</a>.</p>
<h4>OpenAI Responses API, Anthropic token counting, and Bedrock prompt caching<span class="hx-absolute -hx-mt-20" id="openai-responses-api-anthropic-token-counting-and-bedrock-prompt-caching"></span>
    <a class="subheading-anchor" href="#openai-responses-api-anthropic-token-counting-and-bedrock-prompt-caching"></a></h4><p>You can now route traffic for the OpenAI Responses API and Anthropic token-counting endpoints, and configure prompt caching for Amazon Bedrock to improve performance and reduce costs. These enhancements enable significantly faster response times and can reduce LLM-related costs by up to 90% by avoiding repeated prompt processing.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">bedrock-caching-policy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">bedrock-route</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">backend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">ai</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">promptCaching</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">cacheSystem</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">cacheMessages</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">cacheTools</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">minTokens</span><span class="p">:</span><span class="w"> </span><span class="m">1024</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">modelAliases</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">"fast": </span><span class="s2">"amazon.nova-micro-v1:0"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">"smart": </span><span class="s2">"anthropic.claude-3-5-sonnet-20241022-v2:0"</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12855" rel="noopener" target="_blank">PR</a> for more information.</p>
<h4>Stateful/stateless session routing for MCP backends<span class="hx-absolute -hx-mt-20" id="statefulstateless-session-routing-for-mcp-backends"></span>
    <a class="subheading-anchor" href="#statefulstateless-session-routing-for-mcp-backends"></a></h4><p>You can now configure the MCP session behavior for requests to be <code>Stateful</code> or <code>Stateless</code> on the AgentgatewayBackend. Behavior defaults to <code>Stateful</code> if not set.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayBackend</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">mcp-static-no-protocol</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">mcp</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">targets</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">default-target</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">static</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">host</span><span class="p">:</span><span class="w"> </span><span class="l">mcp-server.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">8080</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">sessionRouting</span><span class="p">:</span><span class="w"> </span><span class="l">Stateless</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/13201" rel="noopener" target="_blank">PR</a> for more information.</p>
<h4>Tracing support<span class="hx-absolute -hx-mt-20" id="tracing-support"></span>
    <a class="subheading-anchor" href="#tracing-support"></a></h4><p>You can now dynamically configure tracing for agentgateway using the AgentgatewayPolicy <code>frontend</code> field:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">agw</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">gw</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">frontend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">tracing</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">backendRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">opentelemetry-collector</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">4317</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">GRPC</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">clientSampling</span><span class="p">:</span><span class="w"> </span><span class="s2">"true"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">randomSampling</span><span class="p">:</span><span class="w"> </span><span class="s2">"true"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">resources</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">deployment.environment.name</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">expression</span><span class="p">:</span><span class="w"> </span><span class="s1">'"production"'</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">service.version</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span><span class="nt">expression</span><span class="p">:</span><span class="w"> </span><span class="s1">'"test"'</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">attributes</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">add</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span>- <span class="nt">expression</span><span class="p">:</span><span class="w"> </span><span class="s1">'"literal"'</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">custom</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span>- <span class="nt">expression</span><span class="p">:</span><span class="w"> </span><span class="s1">'request.headers["x-header-tag"]'</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">request</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/13226" rel="noopener" target="_blank">PR</a> for more information on the new changes.</p>
<p>For setup steps, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/observability/tracing/" rel="noopener" target="_blank">docs</a>.</p>
<h4>CipherSuite configuration via frontend TLS policy<span class="hx-absolute -hx-mt-20" id="ciphersuite-configuration-via-frontend-tls-policy"></span>
    <a class="subheading-anchor" href="#ciphersuite-configuration-via-frontend-tls-policy"></a></h4><p>You can now configure the cipher-suites and min and max TLS version on the agentgateway proxy by using the <code>spec.frontend.tls</code> fields in the AgentgatewayPolicy.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">agentgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">AgentgatewayPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">tls-policy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span>- <span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">test</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">frontend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">tls</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">alpnProtocols</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="l">h2</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="l">http/1.1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">handshakeTimeout</span><span class="p">:</span><span class="w"> </span><span class="l">15s</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">minProtocolVersion</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.2"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">maxProtocolVersion</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.3"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">cipherSuites</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="l">TLS13_AES_256_GCM_SHA384</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="l">TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/13219" rel="noopener" target="_blank">PR</a> for more information.</p>
<h4>Basic auth, api-key auth, and JWT auth<span class="hx-absolute -hx-mt-20" id="basic-auth-api-key-auth-and-jwt-auth"></span>
    <a class="subheading-anchor" href="#basic-auth-api-key-auth-and-jwt-auth"></a></h4><p>Agentgateway proxies now support basic auth, API key auth and JWT auth.
See this <a href="https://github.com/kgateway-dev/kgateway/pull/12886" rel="noopener" target="_blank">PR</a> for more information.</p>
<p>For a JWT setup example, see the <a href="https://agentgateway.dev/docs/kubernetes/latest/security/jwt/setup/" rel="noopener" target="_blank">docs</a>.</p>
<h3>Highlighted Envoy features<span class="hx-absolute -hx-mt-20" id="highlighted-envoy-features"></span>
    <a class="subheading-anchor" href="#highlighted-envoy-features"></a></h3><h4>API Gateway feature gaps<span class="hx-absolute -hx-mt-20" id="api-gateway-feature-gaps"></span>
    <a class="subheading-anchor" href="#api-gateway-feature-gaps"></a></h4><p>




<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/img/feature-gap-epic.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div>





<div><figure class="hx-hidden dark:hx-block"><img alt="" class="hx-hidden dark:hx-block" src="/img/feature-gap-epic.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div></p>
<p>Issue: <a href="https://github.com/kgateway-dev/kgateway/issues/12910" rel="noopener" target="_blank">https://github.com/kgateway-dev/kgateway/issues/12910</a></p>
<p>One of the most common themes of feedback we received from the v2.1 release is that there were several missing features which can be considered “tablestakes” for API gateways. This feedback was completely valid and we took it to heart, so we gathered the most requested features and made sure to deliver them for the v2.2 release! Huge thanks goes to all that gave us this important feedback on Slack, GitHub, or anywhere else!</p>
<h4>API key authentication<span class="hx-absolute -hx-mt-20" id="api-key-authentication"></span>
    <a class="subheading-anchor" href="#api-key-authentication"></a></h4><p>API key authentication is a common security mechanism for API gateways, allowing clients to authenticate using API keys provided in HTTP headers. API key authentication support has now been added to TrafficPolicies, allowing API keys to be provided via HTTP headers, query parameters, or cookies.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">api-key-auth-gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">gw</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">apiKeyAuth</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">keySources</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">header</span><span class="p">:</span><span class="w"> </span><span class="s2">"api-key"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">forwardCredential</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">secretRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">api-keys-secret</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See the following PRs for more information:</p>
<ul>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/12962" rel="noopener" target="_blank">12962</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13217" rel="noopener" target="_blank">13217</a></li>
</ul>
<h4>Basic auth<span class="hx-absolute -hx-mt-20" id="basic-auth"></span>
    <a class="subheading-anchor" href="#basic-auth"></a></h4><p>TrafficPolicy now provides configuration for HTTP Basic authentication. Basic authentication checks if an incoming request has a valid username and password before routing the request to a backend service.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">route-basicauth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">route-secure</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">basicAuth</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">users</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="l">alice:{SHA}W6ph5Mm5Pz8GgiULbPgzG37mj9g=</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span>- <span class="l">bob:{SHA}W6ph5Mm5Pz8GgiULbPgzG37mj9g=</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12983" rel="noopener" target="_blank">PR</a> for more information.</p>
<h4>JWT Authentication<span class="hx-absolute -hx-mt-20" id="jwt-authentication"></span>
    <a class="subheading-anchor" href="#jwt-authentication"></a></h4><p>You can now configure JWT policies in the TrafficPolicy. To setup your JWT providers, use the GatewayExtension resource.</p>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12811" rel="noopener" target="_blank">PR</a> for more information.</p>
<h4>OAuth2 and OIDC flows<span class="hx-absolute -hx-mt-20" id="oauth2-and-oidc-flows"></span>
    <a class="subheading-anchor" href="#oauth2-and-oidc-flows"></a></h4><p>Added capability to protect traffic using OAuth2/OIDC policy when using Envoy as the Gateway proxy.</p>
<p>You can configure kgateway with your OIDC provider&rsquo;s details by creating a GatewayExtension resource to hold the OIDC configuration, a Backend and BackendTLSPolicy to allow kgateway to communicate with the OIDC endpoints, and a Secret for your client secret.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">GatewayExtension</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">google-oauth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">oauth2</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">backendRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Backend</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">google-oauth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">issuerURI</span><span class="p">:</span><span class="w"> </span><span class="l">https://accounts.google.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="c"># tokenEndpoint and authorizationEndpoint can be omitted to use OpenID provider config discovery using the issuerURI</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="c">#tokenEndpoint: https://oauth2.googleapis.com/token</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="c">#authorizationEndpoint: https://accounts.google.com/o/oauth2/v2/auth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">credentials</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="c"># FIXME: replace with your OAuth2 client ID</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">clientID</span><span class="p">:</span><span class="w"> </span><span class="l">your-client-id</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">clientSecretRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">google-oauth-client-secret</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">redirectURI</span><span class="p">:</span><span class="w"> </span><span class="l">https://example.com:8443/oauth2/redirect</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">logoutPath</span><span class="p">:</span><span class="w"> </span><span class="l">/logout</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">forwardAccessToken</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">scopes</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"openid"</span><span class="p">,</span><span class="w"> </span><span class="s2">"email"</span><span class="p">]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Backend</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">google-oauth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">Static</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">static</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">hosts</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span>- <span class="nt">host</span><span class="p">:</span><span class="w"> </span><span class="l">oauth2.googleapis.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">443</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">BackendTLSPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">google-oauth-tls</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Backend</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">google-oauth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">validation</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">hostname</span><span class="p">:</span><span class="w"> </span><span class="l">oauth2.googleapis.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">wellKnownCACertificates</span><span class="p">:</span><span class="w"> </span><span class="l">System</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Secret</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">google-oauth-client-secret</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">data</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="c"># FIXME: replace with your base64 encoded OAuth2 client secret</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">client-secret</span><span class="p">:</span><span class="w"> </span><span class="l">Y2xpZW50LXNlY3JldA==</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See the following PRs for more information:</p>
<ul>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13051" rel="noopener" target="_blank">13051</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13099" rel="noopener" target="_blank">13099</a></li>
</ul>
<h4>ListenerPolicy CRD and ProxyProtocol<span class="hx-absolute -hx-mt-20" id="listenerpolicy-crd-and-proxyprotocol"></span>
    <a class="subheading-anchor" href="#listenerpolicy-crd-and-proxyprotocol"></a></h4><p>Kgateway now exposes config to accept incoming network traffic with Proxy Protocol via the ListenerPolicy resource.</p>
<p>The new ListenerPolicy also supports <code>preserveExternalRequestId</code>, <code>generateRequestId</code>, so users can now disable the generation of Request ID and preserve the external request ID.</p>
<p>Example fields on the new ListenerPolicy:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">ListenerPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http-listener-policy-all-fields</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">gw</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">default</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">httpSettings</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">useRemoteAddress</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">xffNumTrustedHops</span><span class="p">:</span><span class="w"> </span><span class="m">2</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">serverHeaderTransformation</span><span class="p">:</span><span class="w"> </span><span class="l">AppendIfAbsent</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">streamIdleTimeout</span><span class="p">:</span><span class="w"> </span><span class="l">30s</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">acceptHttp10</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">preserveExternalRequestId</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">generateRequestId</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">maxRequestHeadersKb</span><span class="p">:</span><span class="w"> </span><span class="m">64</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">healthCheck</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">path</span><span class="p">:</span><span class="w"> </span><span class="s2">"/health_check"</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See the following PRs for more information:</p>
<ul>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/12979" rel="noopener" target="_blank">12979</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13250" rel="noopener" target="_blank">13250</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13224" rel="noopener" target="_blank">13224</a></li>
</ul>
<p>To learn more about the ListenerPolicy, see the <a href="https://kgateway.dev/docs/envoy/latest/about/policies/listenerpolicy/" rel="noopener" target="_blank">docs</a>. To set up Proxy Protocol with a ListenerPolicy, check out this <a href="https://kgateway.dev/docs/envoy/latest/traffic-management/proxy-protocol/" rel="noopener" target="_blank">guide</a>.</p>
<h4><code>earlyRequestHeaderModifier</code> to perform header modifications before route selection<span class="hx-absolute -hx-mt-20" id="earlyrequestheadermodifier-to-perform-header-modifications-before-route-selection"></span>
    <a class="subheading-anchor" href="#earlyrequestheadermodifier-to-perform-header-modifications-before-route-selection"></a></h4><p>Kgateway now supports sanitizing HTTP headers from an incoming request. This is especially useful for gateways that are handling untrusted downstream traffic.
Before, there were ways to do this with kgateway using a transformation policy or the header modifier feature, but these occur as &ldquo;standard&rdquo; filters in an already executing filter chain, they will not guarantee that the headers are removed before any routing or processing decisions are made. Now we support <code>earlyRequestHeaderModifier</code> on the ListenerPolicy:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">ListenerPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">early-header-mutation</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">example-gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"> </span><span class="nt">default</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">   </span><span class="nt">httpSettings</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">     </span><span class="nt">earlyRequestHeaderModifier</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">add</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="s2">"x-added-one"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="s2">"v1"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="s2">"x-added-two"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="s2">"v2"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">set</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="s2">"x-set"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">           </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="s2">"s1"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">       </span><span class="nt">remove</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">         </span>- <span class="s2">"x-remove"</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12992" rel="noopener" target="_blank">PR</a> for more information.</p>
<p>For setup steps, see the <a href="https://kgateway.dev/docs/envoy/latest/traffic-management/header-control/early-request-header-modifier/" rel="noopener" target="_blank">docs</a>.</p>
<h4>Metrics and logs for Envoy xDS errors<span class="hx-absolute -hx-mt-20" id="metrics-and-logs-for-envoy-xds-errors"></span>
    <a class="subheading-anchor" href="#metrics-and-logs-for-envoy-xds-errors"></a></h4><p>Kgateway now has a warning log and metric when envoy NACKs. 2 metrics are added. One metric is a total counter, and the other is a gauge for the current state. Both are labeled by (gwname, gw-ns, envoy-resource).</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-sh"><span class="line"><span class="cl"><span class="c1"># TYPE kgateway_envoy_xds_rejects_active gauge</span>
</span></span><span class="line"><span class="cl">kgateway_envoy_xds_rejects_active<span class="o">{</span><span class="nv">gateway_name</span><span class="o">=</span><span class="s2">"gw"</span>,gateway_namespace<span class="o">=</span><span class="s2">"default"</span>,type_url<span class="o">=</span><span class="s2">"envoy.config.route.v3.RouteConfiguration"</span><span class="o">}</span> <span class="m">0</span>
</span></span><span class="line"><span class="cl"><span class="c1"># HELP kgateway_envoy_xds_rejects_total Total number of xDS responses rejected by envoy proxy</span>
</span></span><span class="line"><span class="cl"><span class="c1"># TYPE kgateway_envoy_xds_rejects_total counter</span>
</span></span><span class="line"><span class="cl">kgateway_envoy_xds_rejects_total<span class="o">{</span><span class="nv">gateway_name</span><span class="o">=</span><span class="s2">"gw"</span>,gateway_namespace<span class="o">=</span><span class="s2">"default"</span>,type_url<span class="o">=</span><span class="s2">"envoy.config.route.v3.RouteConfiguration"</span><span class="o">}</span> <span class="m">1</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/13003" rel="noopener" target="_blank">PR</a> for more information.</p>
<h4>Multi-arch image support using upstream Envoy for ARM<span class="hx-absolute -hx-mt-20" id="multi-arch-image-support-using-upstream-envoy-for-arm"></span>
    <a class="subheading-anchor" href="#multi-arch-image-support-using-upstream-envoy-for-arm"></a></h4><p>




<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/img/arm-builds.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div>





<div><figure class="hx-hidden dark:hx-block"><img alt="" class="hx-hidden dark:hx-block" src="/img/arm-builds.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div></p>
<p>After years of waiting, we have finally delivered multi-arch Envoy builds! Thank you for the extreme patience shown by our community users as we navigated several technical challenges to get to this point.</p>
<p>Some historical context is needed to explain how we got to this point. The 1.x version of kgateway, Gloo Edge, had a custom Envoy build which included Gloo-specific filters, notably the filter which enables <a href="https://kgateway.dev/docs/envoy/latest/traffic-management/transformations/" rel="noopener" target="_blank">our powerful transformation feature</a>. The build process for this custom Envoy predated the widespread adoption of multi-arch builds in upstream Envoy. It would have taken a very big investment to rework our entire pipeline to modernize the process.</p>
<p>However, thanks to the new Envoy feature <a href="https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/advanced/dynamic_modules" rel="noopener" target="_blank">dynamic modules</a>, we are able to extract our custom functionality out of the full Envoy build process, enabling us to move away from our legacy Envoy build pipeline. A massive thanks goes to all the upstream Envoy dynamic module contributors!</p>
<p>Starting in v2.2, kgateway will use a Rust-based dynamic module (rustformation) for the transformation functionality. With that, we were able to use the vanilla Envoy upstream arm64 image and bundle our Rust dynamic module with that in our envoy-wrapper image to support arm64 build. While rustformation will now be used by default, with this being the first version to use the dynamic module, we want to give current users a way to go back to the old C++ base transformation filter, which is possible as the x86 build is still using the envoy-gloo binary.</p>
<p>Please refer to the release note and this <a href="https://github.com/kgateway-dev/kgateway/blob/v2.2.x/docs/guides/transformation.md" rel="noopener" target="_blank">guide</a> to see the major differences. The C++ base transformation is being deprecated and will be completely removed in the next release. The dynamic module development in Envoy is very active, including C++ and Go SDK in the current main development branch. It’s a great way to build and iterate on custom features for Envoy.</p>
<p>Note: Strict validation is currently not supported for transformation policies with multi-arch builds. (<a href="https://github.com/kgateway-dev/kgateway/pull/13356" rel="noopener" target="_blank">#13356</a>)</p>
<p>See the following PRs for more information:</p>
<ul>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13356" rel="noopener" target="_blank">13356</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13194" rel="noopener" target="_blank">13194</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13319" rel="noopener" target="_blank">13319</a></li>
</ul>
<h3>TLS Listener Improvements<span class="hx-absolute -hx-mt-20" id="tls-listener-improvements"></span>
    <a class="subheading-anchor" href="#tls-listener-improvements"></a></h3><h4>FrontendTLSConfig support<span class="hx-absolute -hx-mt-20" id="frontendtlsconfig-support"></span>
    <a class="subheading-anchor" href="#frontendtlsconfig-support"></a></h4><p>Kgateway and agentgateway now implement the <a href="https://gateway-api.sigs.k8s.io/reference/1.4/spec/#frontendtlsconfig" rel="noopener" target="_blank">FrontendTLSConfig</a>. This config allows you to set up a mutual TLS listener on the gateway.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="l">kubectl apply -f- &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">mtls</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway-system</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">gatewayClassName</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">tls</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">frontend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">default</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">validation</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">mode</span><span class="p">:</span><span class="w"> </span><span class="l">AllowValidOnly</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">caCertificateRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ca-cert</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">ConfigMap</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">https-8443</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPS</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">8443</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">tls</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">mode</span><span class="p">:</span><span class="w"> </span><span class="l">Terminate</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">certificateRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">https-cert</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Secret</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">allowedRoutes</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">namespaces</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">from</span><span class="p">:</span><span class="w"> </span><span class="l">All</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">https-8444</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPS</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">8444</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">tls</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">mode</span><span class="p">:</span><span class="w"> </span><span class="l">Terminate</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">certificateRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">https-cert</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Secret</span><span class="w">
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
<p>FrontendTLS supports the following configurations:</p>
<ul>
<li><strong>Default (required)</strong>: Create the default client certificate validation configuration for all Gateway listeners that handle HTTPS traffic. For an example, see the <a href="https://kgateway.dev/docs/envoy/latest/setup/listeners/mtls/#default" rel="noopener" target="_blank">Default configuration for all listeners guide</a>.</li>
<li><strong>perPort (optional)</strong>: Override the default configuration with port-specific configuration. The configuration is applied only to matching ports that handle HTTPS traffic. For all other ports that handle HTTPS traffic, the default configuration continues to apply. For an example, see the <a href="https://kgateway.dev/docs/envoy/latest/setup/listeners/mtls/#perport" rel="noopener" target="_blank">Per port configuration guide</a>.</li>
</ul>
<p>In addition, you can choose between the following validation modes:</p>
<ul>
<li><strong>AllowValidOnly</strong>: A connection between a client and the gateway proxy can only be established if the gateway can validate the client’s TLS certificate successfully. For an example, see the <a href="https://kgateway.dev/docs/envoy/latest/setup/listeners/mtls/#default" rel="noopener" target="_blank">Default configuration for all listeners guide</a>.</li>
<li><strong>AllowInsecureFallback</strong>: The gateway proxy can establish a TLS connection, even if the client TLS certificate could not be validated successfully. For an example, see the <a href="https://kgateway.dev/docs/envoy/latest/setup/listeners/mtls/#perport" rel="noopener" target="_blank">Per port configuration guide</a>.</li>
</ul>
<p>To support FrontendTLSConfig, the following changes were introduced:</p>
<ul>
<li>Allow multiple <code>caCertificateRefs</code>. You can use secrets or configmaps to reference your TLS credentials. See this <a href="https://github.com/kgateway-dev/kgateway/pull/12895" rel="noopener" target="_blank">PR</a> for more information.</li>
<li>Allow configuring cipher suites, ecdh curves, minimum TLS version, and maximum TLS version using the TLS options map. See this <a href="https://github.com/kgateway-dev/kgateway/pull/12917" rel="noopener" target="_blank">PR</a> for more information.</li>
<li>Added the <code>kgateway.dev/verify-certificate-hash</code> to listener TLS options to allow configuration of validating client certificates. See this <a href="https://github.com/kgateway-dev/kgateway/pull/13064" rel="noopener" target="_blank">PR</a> for more details.</li>
<li>Added <code>kgateway.dev/verify-certificate-hash</code> and <code>kgateway.dev/verify-subject-alt-names</code> annotations to limit connections to clients that present certificates with a specific Subject Alt Name and certificate hash. See the following PRs for more details:
<ul>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13064" rel="noopener" target="_blank">13064</a></li>
<li><a href="https://github.com/kgateway-dev/kgateway/pull/13097" rel="noopener" target="_blank">13097</a></li>
</ul>
</li>
</ul>
<p>For more information, see the following doc guides:</p>
<ul>
<li><a href="https://kgateway.dev/docs/envoy/latest/setup/listeners/mtls/" rel="noopener" target="_blank">mTLS (Frontend TLS)</a></li>
<li><a href="https://kgateway.dev/docs/envoy/latest/setup/listeners/tls-settings/" rel="noopener" target="_blank">Additional TLS settings</a></li>
</ul>
<h4>TLS termination for TCPRoutes<span class="hx-absolute -hx-mt-20" id="tls-termination-for-tcproutes"></span>
    <a class="subheading-anchor" href="#tls-termination-for-tcproutes"></a></h4><p>You can now terminate TLS connections for TCPRoutes.</p>
<p>See this <a href="https://github.com/kgateway-dev/kgateway/pull/12906" rel="noopener" target="_blank">PR</a> for more information.</p>
<h3>Ingress Migration<span class="hx-absolute -hx-mt-20" id="ingress-migration"></span>
    <a class="subheading-anchor" href="#ingress-migration"></a></h3><p>If you’re currently running <a href="https://kubernetes.github.io/ingress-nginx/" rel="noopener" target="_blank">Ingress Nginx</a> to support the Kubernetes Ingress API, <a href="https://github.com/kgateway-dev/ingress2gateway" rel="noopener" target="_blank">ingress2gateway</a> can help you migrate to Gateway API by translating your existing Ingress manifests into Gateway, HTTPRoute, and implementation-specific policy resources. The tool provides coverage for common Ingress Nginx annotations (auth, rate limiting, CORS, session affinity, backend TLS, SSL redirect, and more) and can emit resources tailored for either kgateway (Envoy) or agentgateway. Choose your migration guide to learn more:</p>
<ul>
<li><a href="https://kgateway.dev/docs/envoy/latest/migrate/" rel="noopener" target="_blank">Kgateway (Envoy) migration guide</a></li>
<li><a href="https://agentgateway.dev/docs/kubernetes/latest/migrate/" rel="noopener" target="_blank">Agentgateway migration guide</a></li>
</ul>
<h2>Release notes<span class="hx-absolute -hx-mt-20" id="release-notes"></span>
    <a class="subheading-anchor" href="#release-notes"></a></h2><p>Check out the full details of the kgateway v2.2 release in our <a href="https://kgateway.dev/docs/envoy/latest/reference/release-notes/" rel="noopener" target="_blank">release notes</a>.</p>
<h2>Availability<span class="hx-absolute -hx-mt-20" id="availability"></span>
    <a class="subheading-anchor" href="#availability"></a></h2><p>kgateway v2.2  is available for download on <a href="https://github.com/kgateway-dev/kgateway" rel="noopener" target="_blank">GitHub</a>.</p>
<p>To get started with kgateway, check out our getting started guides for <a href="https://kgateway.dev/docs/envoy/latest/quickstart/" rel="noopener" target="_blank">kgateway</a> or <a href="https://agentgateway.dev/docs/kubernetes/latest/quickstart/" rel="noopener" target="_blank">agentgateway</a>.</p>
<h2>Get Involved<span class="hx-absolute -hx-mt-20" id="get-involved"></span>
    <a class="subheading-anchor" href="#get-involved"></a></h2><p>The simplest way to get involved with kgateway is by joining our <a href="https://kgateway.dev/slack/" rel="noopener" target="_blank">Slack</a> and <a href="https://github.com/kgateway-dev/community?tab=readme-ov-file#community-meetings" rel="noopener" target="_blank">community meetings</a>.</p>
<h2>Contributors<span class="hx-absolute -hx-mt-20" id="contributors"></span>
    <a class="subheading-anchor" href="#contributors"></a></h2><p>Thanks to the 40+ contributors who made this release possible:</p>
<p>




<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/img/contributors.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div>





<div><figure class="hx-hidden dark:hx-block"><img alt="" class="hx-hidden dark:hx-block" src="/img/contributors.png" width="" /> <figcaption style="font-style: italic;"></figcaption></figure></div></p>
