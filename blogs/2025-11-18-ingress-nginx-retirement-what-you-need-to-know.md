---
title: "Ingress NGINX Retirement: What You Need to Know"
url: "/blog/nginx-ingress-retirement-what-you-need-to-know/"
date: "Tue, 18 Nov 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>During KubeConNA last week, the Kubernetes community <a href="https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/" rel="noopener" target="_blank">announced</a> the <strong>Ingress NGINX retirement</strong> and recommended that users move to the <strong>Gateway API</strong>, which is the modern replacement for Ingress. Best-effort maintenance of Ingress NGINX will continue until <strong>March 2026</strong>, meaning users need a migration plan soon.</p>
<p>This announcement is significant—Ingress NGINX has been one of the most popular ingress controllers for traffic into Kubernetes clusters. It’s part of the core Kubernetes project with over 19,000 stars on <a href="https://github.com/kubernetes/ingress-nginx" rel="noopener" target="_blank">GitHub</a>. In this blog, we’ll share key considerations to help you choose a replacement.</p>
<h3>What Is Ingress NGINX?<span class="hx-absolute -hx-mt-20" id="what-is-ingress-nginx"></span>
    <a class="subheading-anchor" href="#what-is-ingress-nginx"></a></h3><p>Before planning a migration, it’s important to understand what <strong>Ingress</strong> and <strong>Ingress NGINX</strong> are.</p>
<p>In Kubernetes, an <strong><a href="https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/" rel="noopener" target="_blank">Ingress Controller</a></strong> is essential—it watches Ingress objects in the cluster and programs NGINX accordingly, routing incoming traffic to the applications in your cluster.</p>
<p>Here’s a simple example of an Ingress object:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Ingress</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">example-ingress</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">host</span><span class="p">:</span><span class="w"> </span><span class="l">example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">http</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">paths</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span>- <span class="nt">path</span><span class="p">:</span><span class="w"> </span><span class="l">/</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">pathType</span><span class="p">:</span><span class="w"> </span><span class="l">Prefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">backend</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">service</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">example-service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">port</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">              </span><span class="nt">number</span><span class="p">:</span><span class="w"> </span><span class="m">80</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>This config routes traffic from <code>example.com</code> to the backend service <code>example-service</code> on port 80, allowing users to reach your application from outside the cluster.</p>
<h2>Why Gateway API?<span class="hx-absolute -hx-mt-20" id="why-gateway-api"></span>
    <a class="subheading-anchor" href="#why-gateway-api"></a></h2><p>One challenge with the Ingress API is <strong>inconsistent behavior across vendors</strong>, largely due to its reliance on annotations. These overloaded annotations are project-specific and can behave unpredictably when migrating between implementations. Earlier this year, Wiz Research disclosed <a href="https://www.wiz.io/blog/ingress-nginx-kubernetes-vulnerabilities" rel="noopener" target="_blank">several CVEs</a> in NGINX related to annotation-based authentication or UID configuration.</p>
<p>Another challenge is the <strong>lack of a proper status field</strong>, which makes troubleshooting difficult. While you can inspect rules, events, and annotations, it’s often unclear why an Ingress isn’t working.</p>
<p>Because of these limitations, the Kubernetes community has been evolving the Ingress API since KubeCon 2019. The Gateway API reached <strong><a href="https://kubernetes.io/blog/2023/10/31/gateway-api-ga/" rel="noopener" target="_blank">GA</a></strong> in <strong>October 2023</strong>, making core APIs like <code>Gateway</code>, <code>GatewayClass</code>, and <code>HTTPRoute</code> stable.</p>
<h2>Advantages of Gateway API<span class="hx-absolute -hx-mt-20" id="advantages-of-gateway-api"></span>
    <a class="subheading-anchor" href="#advantages-of-gateway-api"></a></h2><p>What we love about the Gateway API:</p>
<ul>
<li>Extensibility: You can define custom traffic policies, rate limits, and more.</li>
<li>Status field: Provides clear insights into whether a resource is accepted, programmed, and error-free.</li>
</ul>
<p>Example with <strong>kgateway</strong> using the core Gateway and the extended TrafficPolicy resources:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">gatewayClassName</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">transformation</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">transformation</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">request</span><span class="p">:</span><span class="w">  
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">add</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">x-forwarded-uri</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="s1">'https://{{ request_header(":authority") }}{{ request_header(":path") }}'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>After deployment, the status field for each resource lets you see whether resources are programmed correctly and if any errors exist. Here’s an example of the status field for the <code>http</code> Gateway resource:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">addresses</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">IPAddress</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="m">172.18.0.7</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">conditions</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="l">...</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">lastTransitionTime</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025-11-17T22:59:37Z"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">message</span><span class="p">:</span><span class="w"> </span><span class="l">Successfully accepted Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">observedGeneration</span><span class="p">:</span><span class="w"> </span><span class="m">1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">reason</span><span class="p">:</span><span class="w"> </span><span class="l">Accepted</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">status</span><span class="p">:</span><span class="w"> </span><span class="s2">"True"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">Accepted</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">lastTransitionTime</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025-11-17T22:59:37Z"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">message</span><span class="p">:</span><span class="w"> </span><span class="l">Successfully programmed Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">observedGeneration</span><span class="p">:</span><span class="w"> </span><span class="m">1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">reason</span><span class="p">:</span><span class="w"> </span><span class="l">Programmed</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">status</span><span class="p">:</span><span class="w"> </span><span class="s2">"True"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">Programmed</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">attachedRoutes</span><span class="p">:</span><span class="w"> </span><span class="m">1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">conditions</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="l">...</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">lastTransitionTime</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025-11-17T22:59:37Z"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">message</span><span class="p">:</span><span class="w"> </span><span class="l">Successfully resolved all references</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">observedGeneration</span><span class="p">:</span><span class="w"> </span><span class="m">1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">reason</span><span class="p">:</span><span class="w"> </span><span class="l">ResolvedRefs</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">status</span><span class="p">:</span><span class="w"> </span><span class="s2">"True"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">ResolvedRefs</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">lastTransitionTime</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025-11-17T22:59:37Z"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">message</span><span class="p">:</span><span class="w"> </span><span class="l">Successfully programmed Listener</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">observedGeneration</span><span class="p">:</span><span class="w"> </span><span class="m">1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">reason</span><span class="p">:</span><span class="w"> </span><span class="l">Programmed</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">status</span><span class="p">:</span><span class="w"> </span><span class="s2">"True"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">Programmed</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">http</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h2>Envoy Proxy<span class="hx-absolute -hx-mt-20" id="envoy-proxy"></span>
    <a class="subheading-anchor" href="#envoy-proxy"></a></h2><p>Many Gateway API <a href="https://gateway-api.sigs.k8s.io/implementations/" rel="noopener" target="_blank">implementations</a> are built on <strong>Envoy</strong>, a high-performance modern proxy. Projects like Istio, kgateway, Contour, Cilium, Envoy Gateway, and Emissary-Ingress use Envoy as the data plane.</p>
<p>While multiple gateways may share the same data plane, the <strong>control plane</strong> is what sets them apart. The control plane translates Gateway API resources into Envoy configuration. For small setups, this is simple, but for large-scale deployments (e.g., 20,000 routes → 500,000+ lines of Envoy config), control plane <strong>efficiency and scalability</strong> are critical.</p>
<p>Check out our <a href="https://kgateway.dev/blog/design-kgateway-for-scalability/" rel="noopener" target="_blank">blog</a> on designing kgateway for scalability, which includes simple tests to evaluate control plane performance during route changes.</p>
<h2>Gateway API Benchmark<span class="hx-absolute -hx-mt-20" id="gateway-api-benchmark"></span>
    <a class="subheading-anchor" href="#gateway-api-benchmark"></a></h2><p>Performance matters. Regardless of whether your environment is bare-metal, virtualized, cloud, Kubernetes, or serverless, network speed and application responsiveness remain crucial.</p>
<p>One of the most thorough benchmarks comes from John Howard, who recently published <a href="https://github.com/howardjohn/gateway-api-bench/blob/main/README-v2.md" rel="noopener" target="_blank">v2</a> with reproducible test scripts. We recommend checking it out, running the tests in your environment, and engaging with his findings.</p>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/nginx-ingress-retirement-1.png" width="" /> <figcaption style="font-style: italic;">Resource Consumption During Route Scale Test from John’s Benchmark</figcaption></figure></div>
<h2>Inference and Agentic AI<span class="hx-absolute -hx-mt-20" id="inference-and-agentic-ai"></span>
    <a class="subheading-anchor" href="#inference-and-agentic-ai"></a></h2><p>According to the latest CNCF <a href="https://www.cncf.io/reports/state-of-cloud-native-development/" rel="noopener" target="_blank">State of Cloud Native Development Report</a>, ~1/3 of cloud native developers are using AI.</p>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/nginx-ingress-retirement-2.png" width="750px" /> <figcaption style="font-style: italic;">Trends in Cloud Native Development</figcaption></figure></div>
<p>If you’re adopting AI workloads, it’s worth considering a <strong>consistent</strong> Gateway not only for Ingress traffic but also for inference and agentic AI workloads. The CNCF <a href="https://www.cncf.io/reports/cncf-technology-landscape-radar/" rel="noopener" target="_blank">Tech Radar</a> ranks gateways for inference and agentic AI usage, helping guide your selection.</p>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/nginx-ingress-retirement-3.png" width="750px" /> <figcaption style="font-style: italic;">Maturity Ratings for AI Inferencing from the Tech Radar</figcaption></figure></div>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/nginx-ingress-retirement-4.png" width="600px" /> <figcaption style="font-style: italic;">Agentic AI Radar from the Tech Radar</figcaption></figure></div>
<h2>Migrate with the kgateway ingress2gateway tool<span class="hx-absolute -hx-mt-20" id="migrate-with-the-kgateway-ingress2gateway-tool"></span>
    <a class="subheading-anchor" href="#migrate-with-the-kgateway-ingress2gateway-tool"></a></h2><p>To help you migrate from Ingress to kgateway, the kgateway team forked the Kubernetes ingress2gateway tool. You can use the <a href="https://github.com/kgateway-dev/ingress2gateway" rel="noopener" target="_blank">kgateway ingress2gateway tool</a> to convert Ingress manifests to Gateway API and kgateway resources, including enhanced support for migrating Ingress NGINX-specific annotations.</p>
<p>To get started, check out the docs.</p>
<div class="hextra-cards hx-mt-4 hx-gap-4 hx-grid not-prose">
  <a class="hextra-card hx-group hx-flex hx-flex-col hx-justify-start hx-overflow-hidden hx-rounded-lg hx-border hx-border-gray-200 hx-text-current hx-no-underline dark:hx-shadow-none hover:hx-shadow-gray-100 dark:hover:hx-shadow-none hx-shadow-gray-100 active:hx-shadow-sm active:hx-shadow-gray-200 hx-transition-all hx-duration-200 hover:hx-border-gray-300 hx-bg-transparent hx-shadow-sm dark:hx-border-neutral-800 hover:hx-bg-slate-50 hover:hx-shadow-md dark:hover:hx-border-neutral-700 dark:hover:hx-bg-neutral-900" href="../../docs/envoy/latest/migrate/"><span class="hextra-card-icon hx-flex hx-font-semibold hx-items-start hx-gap-2 hx-p-4 hx-text-gray-700 hover:hx-text-gray-900 dark:hx-text-neutral-200 dark:hover:hx-text-neutral-50">kgateway ingress2gateway tool</span></a>
</div>

<h2>Wrapping Up<span class="hx-absolute -hx-mt-20" id="wrapping-up"></span>
    <a class="subheading-anchor" href="#wrapping-up"></a></h2><p>With Ingress NGINX retiring soon, you need a Gateway solution that:</p>
<ul>
<li>Is based on the Kubernetes Gateway API.</li>
<li>Provides consistent performance, whether you have 2 or 2,000 routes.</li>
<li>Is open-source and ideally hosted in a vendor-neutral foundation like CNCF.</li>
<li>Has a thriving community of users.</li>
<li>Provides a consistent Gateway for inference and/or agentic AI workloads if your organization is using or planning to adopt AI.</li>
<li>Is easy to use.</li>
</ul>
<p>Are there any important criteria we missed for evaluating your next Gateway? We’d love to hear what matters most to you.</p>
<p>Good luck choosing the best replacement for NGINX Ingress! We hope your migration is as smooth as possible. If you have any questions regarding kgateway (we may be biased but we believe it meets all of the criteria above) or Istio, feel free to <a href="https://kgateway.dev/slack/" rel="noopener" target="_blank">reach out</a> to our maintainers for assistance.</p>
