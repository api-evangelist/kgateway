---
title: "Delegation in kgateway - scaling routing with multi-tenant ownership"
url: "/blog/delegation-kgateway/"
date: "Fri, 09 May 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>As environments scale, traffic routing through Kubernetes gateways naturally becomes increasingly complex. Microservices architecture adoption tends to amplify this challenge as what used to be a single route for a monolith becomes hundreds or even thousands of path-based matchers across services, often bundled under a shared hostname.</p>
<p>While it&rsquo;s technically possible to configure all routes in a single HTTPRoute resource, this monolithic approach doesn&rsquo;t scale well in multi-team environments. A common failure mode observed is a situation where Team A inadvertently configures a route or policy that impacts Team B or Team C, potentially causing outages or other traffic-related issues.</p>
<p>Thankfully, API gateways such as kgateway have long solved this multi-tenancy problem by providing a feature called Route Delegation, which allows the user to split up complex routes into smaller, independently owned units. These units can form a routing hierarchy, where policies and matchers are delegated from parent routes to child routes, and even grandchild routes.</p>
<p>By allowing a complete route be assembled from separate config, we can form a tree of config objects which allow us to achieve the following benefits:</p>
<table>
  <thead>
      <tr>
          <th style="text-align: left;"><strong>Benefit</strong></th>
          <th style="text-align: left;"><strong>Description</strong></th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;">Organize routing rules by user groups</td>
          <td style="text-align: left;">With route delegation, you can break up large routing configurations into smaller routing configurations which makes them easier to maintain and to assign ownership to. Each routing configuration in the routing hierarchy contains the routing rules and policies for only a subset of routes.</td>
      </tr>
      <tr>
          <td style="text-align: left;">Restrict access to routing configuration</td>
          <td style="text-align: left;">Because route delegation lets you break up large routing configurations into smaller, manageable pieces, you can easily assign ownership and restrict access to the smaller routing configurations to the individual or teams that are responsible for a specific app or domain. For example, the network administrator can configure the top level routing rules, such as the hostname and main route match, and delegate the individual routing rules to other teams.</td>
      </tr>
      <tr>
          <td style="text-align: left;">Simplify blue-green route testing</td>
          <td style="text-align: left;">To test new routing configuration, you can easily delegate a specific number of traffic to the new set of routes.</td>
      </tr>
      <tr>
          <td style="text-align: left;">Optimize traffic flows</td>
          <td style="text-align: left;">Route delegation can be used to distribute traffic load across multiple paths or nodes in the cluster, which can improve network performance and reliability.</td>
      </tr>
      <tr>
          <td style="text-align: left;">Easier updates with limited blast radius</td>
          <td style="text-align: left;">Individual teams can easily update the routing configuration for their apps and manage the policies for their routes. If errors are introduced, the blast radius is limited to the set of routes that were changed.</td>
      </tr>
  </tbody>
</table>
<h2>Use Case: Shared Gateway, Isolated Ownership<span class="hx-absolute -hx-mt-20" id="use-case-shared-gateway-isolated-ownership"></span>
    <a class="subheading-anchor" href="#use-case-shared-gateway-isolated-ownership"></a></h2><p>Imagine a platform team managing a shared ingress Gateway for multiple API teams. The platform team defines which hostnames and top-level paths (like <code>/api/reviews</code> or <code>/api/details</code>) are allowed, but delegates ownership of what happens beyond those paths to specific namespaces.</p>
<p>For example:</p>
<ul>
<li>
<p>The <strong>platform team</strong> defines the public interface (<code>bookinfo.example.com/api/reviews</code>) but doesn&rsquo;t care how traffic is split between <code>/v1</code>, <code>/v2</code>, etc.</p>
</li>
<li>
<p>The <strong>reviews team</strong> owns the internal routing logic—versioning, rewrites, backend selection—within their namespace.</p>
</li>
</ul>
<p>In the lab example, we follow a similar scenario where leveraging kgateway delegation provides us a separation of responsibilities to help reduce operational bottlenecks and improve developer velocity - allowing teams to independently modify or version their routes without opening platform tickets or waiting for centralized approval.</p>
<p>In order to accomplish this, we first create the following <code>HTTPRoute</code> manifests in our cluster first starting with the parent route</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">parent</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parentRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">my-gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hostnames</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="l">bookinfo.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/api/reviews</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">reviews</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/api/details</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">details</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>This parent route achieves two key goals:</p>
<ol>
<li>
<p><strong>Defines base-level constraints</strong>: Only traffic for <code>bookinfo.example.com</code> and certain top-level paths is allowed.</p>
</li>
<li>
<p><strong>Delegates routing authority</strong>: Requests matching <code>/api/reviews</code> or <code>/api/details</code> are forwarded to child <code>HTTPRoute</code> resources in the <code>reviews</code> and <code>details</code> namespaces, respectively.</p>
</li>
</ol>
<p><strong>Note:</strong> The <code>backendRefs</code> here point to other <code>HTTPRoute</code> objects—not <code>Service</code> objects—enabling <strong>chained routing logic</strong>. This allows child routes to fully own how traffic is handled beyond the initial match.</p>
<p>Next, we configure the following child <code>HTTPRoute</code> resource for each respective team, in this case the <code>reviews</code> and <code>details</code> API teams, which inherit the host and its routing hierarchy from the parent route configured in the previous step.</p>
<p><strong>Child Route #1:</strong></p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">reviews</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">reviews</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/api/reviews</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">filters</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">URLRewrite</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">urlRewrite</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">ReplacePrefixMatch</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">replacePrefixMatch</span><span class="p">:</span><span class="w"> </span><span class="l">/reviews</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">reviews</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">9080</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p><strong>Child Route #2:</strong></p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">details</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">details</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/api/details</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">filters</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">URLRewrite</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">urlRewrite</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">ReplacePrefixMatch</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">replacePrefixMatch</span><span class="p">:</span><span class="w"> </span><span class="l">/details</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">details</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">9080</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Why do the child routes matter? These child routes are the entry point for API autonomy, allowing developers to define routing logic for their particular application - how traffic is processed, rewritten, and forwarded - all without affecting other tenants on the shared gateway. Specifically, the following benefits can be observed when using delegation:</p>
<p><strong>Team autonomy and velocity:</strong> Developers can modify, version, or extend their routing logic (e.g. split traffic between <code>/v1</code>, <code>/v2</code>, etc.) without waiting on the platform team or modifying the shared gateway. This accelerates development and deployment workflows.</p>
<p>Example:</p>
<p>A team wants to A/B test between two backends. They can add weighted routes or request-based match conditions in their child route independently.</p>
<p><strong>Logical separation of concerns:</strong> The platform team manages global ingress policy (e.g., only exposing <code>/api/reviews</code>) while the API team manages internal business logic and backend routing under that prefix. This separation supports least privilege access and prevents unrelated teams from affecting shared ingress behavior.</p>
<p><strong>URL decoupling and abstraction:</strong> The external path (<code>/api/reviews</code>) is decoupled from internal application routing (<code>/reviews</code>). This allows developers the ability to change internal app structure without affecting the public APIs and is especially helpful in API versioning, migrations, or exposing legacy systems behind modern URLs.</p>
<p><strong>Scoped permissions and governance:</strong> Because child routes live in the team&rsquo;s namespace, RBAC can enforce that only the owning team can edit them. Platform teams can enforce naming conventions and restrict delegation scopes in the parent route.</p>
<p><strong>Reusability and modularity:</strong> Child routes can evolve independently and be composed into higher-order routing structures (e.g. introducing a &ldquo;grandchild&rdquo; route)</p>
<h2>Summary<span class="hx-absolute -hx-mt-20" id="summary"></span>
    <a class="subheading-anchor" href="#summary"></a></h2><p>In this blog, we explored how route delegation in kgateway is a powerful tool for scaling API gateway management and providing API autonomy across teams in large multitenant environments. It combines:</p>
<ul>
<li>Clear separation of responsibilities</li>
<li>Safe and flexible developer self-service</li>
<li>Enforced governance and policy inheritance</li>
</ul>
<p>If you are enjoying this learning series on Gateway API, we have more for you in store so stay tuned!</p>
<p>For a complete, hands-on example covering the concepts of shared gateways in this blog, check out our <a href="https://youtu.be/5uUGN4Qn_1c" rel="noopener" target="_blank">video</a> and <a href="https://www.solo.io/resources/lab/route-delegation-in-kgateway?web&amp;utm_source=organic%20&amp;utm_medium=FY26&amp;utm_campaign=WW_GEN_LAB_kgateway.dev&amp;utm_content=community" rel="noopener" target="_blank">hands-on lab</a> on route delegation!</p>
