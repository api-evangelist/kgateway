---
title: "Adding Distributed Tracing to AI Gateway: My LFX Mentorship Journey"
url: "/blog/lfx-mentorship-2025-term2-zzk/"
date: "Thu, 28 Aug 2025 00:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>In today&rsquo;s rapidly evolving AI application landscape, effectively monitoring and debugging AI Gateways has become a critical challenge. This article shares my complete experience through the LFX Mentorship program, where I added OpenTelemetry distributed tracing support to kgateway&rsquo;s AI Gateway functionality.</p>
<p>From application strategies for LFX Mentorship to challenges and insights during project implementation, I hope this provides a valuable reference for students who want to participate in open source projects.</p>
<h2>My Background and Preparation<span class="hx-absolute -hx-mt-20" id="my-background-and-preparation"></span>
    <a class="subheading-anchor" href="#my-background-and-preparation"></a></h2><p>Before applying for LFX Mentorship, I had already been exposed to <a href="https://opentelemetry.io/" rel="noopener" target="_blank">OpenTelemetry</a> during my internship, gaining foundational knowledge in the observability domain. More importantly, I participated in the Jaeger community&rsquo;s development work to add Clickhouse support for traces (<a href="https://github.com/jaegertracing/jaeger/pull/6935" rel="noopener" target="_blank">PR #6935</a>), which gave me practical experience with distributed tracing.</p>
<p>These experiences made me feel that the LFX Mentorship project about AI Gateway distributed tracing was an excellent opportunity to deepen my learning and contribute to the open source community.</p>
<h2>Application Strategy: Contribute First, Apply Later<span class="hx-absolute -hx-mt-20" id="application-strategy-contribute-first-apply-later"></span>
    <a class="subheading-anchor" href="#application-strategy-contribute-first-apply-later"></a></h2><p>I know everyone gets excited when they see a project they&rsquo;re interested in and can&rsquo;t wait to apply. But I adopted a different strategy: <strong>first deeply understand the project, actively participate in the community, then submit the application</strong>.</p>
<h3>Deep Product Experience<span class="hx-absolute -hx-mt-20" id="deep-product-experience"></span>
    <a class="subheading-anchor" href="#deep-product-experience"></a></h3><p>Instead of rushing to submit my application, I first went to actually experience the product involved in the project: <a href="https://kgateway.dev/docs/envoy/2.1.x/ai/about/" rel="noopener" target="_blank">AI Gateway</a>. During usage, I discovered a practical problem - I couldn&rsquo;t directly access the LLM service providers mentioned in the official guide.</p>
<h3>Proactive Problem Solving<span class="hx-absolute -hx-mt-20" id="proactive-problem-solving"></span>
    <a class="subheading-anchor" href="#proactive-problem-solving"></a></h3><p>I didn&rsquo;t just report the problem, but proactively proposed a solution. I brought up this issue in the community Slack:</p>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/zzk-slack-first-context.png" width="750px" /> <figcaption style="font-style: italic;">First community interaction</figcaption></figure></div>
<p>Eventually, I successfully submitted a PR: <a href="https://github.com/kgateway-dev/kgateway/pull/11282" rel="noopener" target="_blank">Allow overriding default LLM provider endpoints</a>, which solved this configuration issue.</p>
<h3>Why This Approach Is Important<span class="hx-absolute -hx-mt-20" id="why-this-approach-is-important"></span>
    <a class="subheading-anchor" href="#why-this-approach-is-important"></a></h3><p>By contributing code before applying, I achieved several goals:</p>
<ul>
<li><strong>Deep project understanding</strong>: Truly understood the project&rsquo;s architecture and pain points</li>
<li><strong>Building trust</strong>: Proved to the mentor that I had the capability to complete project tasks</li>
<li><strong>Community integration</strong>: Established connections with community members in advance</li>
</ul>
<h2>Project Core: Adding Distributed Tracing to AI Gateway<span class="hx-absolute -hx-mt-20" id="project-core-adding-distributed-tracing-to-ai-gateway"></span>
    <a class="subheading-anchor" href="#project-core-adding-distributed-tracing-to-ai-gateway"></a></h2><h3>The Problem to Solve<span class="hx-absolute -hx-mt-20" id="the-problem-to-solve"></span>
    <a class="subheading-anchor" href="#the-problem-to-solve"></a></h3><p>AI Gateway is implemented using Envoy&rsquo;s extproc functionality, where all requests to LLM service providers are intercepted and processed by the extension program. Currently, AI Gateway has introduced Metrics in the observability domain (Logs, Metrics, Traces). However, when issues occur, Metrics can only tell us &lsquo;What&rsquo; happened, but not &lsquo;Where&rsquo; the issue occurred, we can only sift through massive amounts of logs for troubleshooting, which is both time-consuming and inefficient.</p>
<p>Distributed tracing can help us:</p>
<ul>
<li><strong>Visualize request flow</strong>: Clearly see the complete path of requests through the system</li>
<li><strong>Rapid problem identification</strong>: Quickly find issues by querying failed requests (like <code>attributes.http.status=400</code>)</li>
<li><strong>Performance analysis</strong>: Understand latency at various stages</li>
<li><strong>Monitor LLM calls</strong>: Track calls and performance across different LLM service providers</li>
</ul>
<h3>Project Goals<span class="hx-absolute -hx-mt-20" id="project-goals"></span>
    <a class="subheading-anchor" href="#project-goals"></a></h3><p>Our goal was to provide complete observability support for AI Gateway:</p>
<ol>
<li><strong>Configuration flexibility</strong>: Users can configure OpenTelemetry tracer and Span Exporter</li>
<li><strong>Standardized tracing</strong>: Follow <a href="https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans/" rel="noopener" target="_blank">OpenTelemetry&rsquo;s GenAI semantic conventions</a></li>
<li><strong>Multi-vendor support</strong>: Support multiple LLM service providers like OpenAI, Anthropic, and Gemini</li>
<li><strong>Ease of use</strong>: Provide simple configuration methods and clear documentation</li>
</ol>
<h3>Design Approach<span class="hx-absolute -hx-mt-20" id="design-approach"></span>
    <a class="subheading-anchor" href="#design-approach"></a></h3><p>The project was divided into two main parts. For more details, see: <a href="https://github.com/kgateway-dev/kgateway/blob/main/design/11177.md" rel="noopener" target="_blank">EP-11777</a>:</p>
<p><strong>Control Plane</strong>: Provides configuration interface through GatewayParameters API, where users can specify tracing backends, sampling rates, and other parameters.</p>
<p><strong>Data Plane</strong>: Implements actual distributed tracing logic in the AI extension server, collecting and exporting trace data.</p>
<p>The key challenge was handling response format differences across different LLM service providers. Each provider&rsquo;s API response structure is different, requiring us to uniformly extract key information such as model names, token usage, response status, etc., and format them according to OpenTelemetry standards.</p>
<h2>Important Lessons from the Development Process<span class="hx-absolute -hx-mt-20" id="important-lessons-from-the-development-process"></span>
    <a class="subheading-anchor" href="#important-lessons-from-the-development-process"></a></h2><h3>Critical Testing Strategy Lessons<span class="hx-absolute -hx-mt-20" id="critical-testing-strategy-lessons"></span>
    <a class="subheading-anchor" href="#critical-testing-strategy-lessons"></a></h3><p>Early in the project, I made an important mistake: skipping unit tests and going directly to E2E testing. At the time, I chose to send generated traces directly to Tempo for verification. While installing Tempo and Grafana via Helm was simple, and I could clearly observe trace generation after sending requests to the gateway, this approach became extremely complex when writing automated tests.</p>
<p><strong>The Problems</strong>:</p>
<ul>
<li>Needed to write complex TraceQL queries to verify data</li>
<li>TraceQL syntax was difficult to debug, consuming significant time</li>
<li>Test environment configuration was complex and hard to maintain</li>
</ul>
<p><strong>Better Solution</strong>:
After discussions with my mentor, Nina, I learned that the system already had a simpler method: send traces to OTel Collector and have the Collector output structured logs. This way, we could directly retrieve all traces data from container logs using <code>kubectl logs</code>, greatly simplifying the testing process.</p>
<h3>Key Takeaways<span class="hx-absolute -hx-mt-20" id="key-takeaways"></span>
    <a class="subheading-anchor" href="#key-takeaways"></a></h3><ol>
<li><strong>Write unit tests first</strong>: Unit tests can verify core logic faster, avoiding debugging in complex integration environments</li>
<li><strong>Seek help promptly</strong>: When encountering difficulties, don&rsquo;t go down rabbit holes alone - mentor experience can help you avoid detours</li>
<li><strong>Understand existing tools</strong>: Fully understand the project&rsquo;s existing testing tools and methods to avoid reinventing the wheel</li>
</ol>
<h3>Community Collaboration<span class="hx-absolute -hx-mt-20" id="community-collaboration"></span>
    <a class="subheading-anchor" href="#community-collaboration"></a></h3><p>Throughout the development process, support from the kgateway community was crucial. Whether for technical discussions or code review feedback, community members were always very helpful. This gave me a deep appreciation for the collaborative spirit of open source projects.</p>
<h2>Project Results and Value<span class="hx-absolute -hx-mt-20" id="project-results-and-value"></span>
    <a class="subheading-anchor" href="#project-results-and-value"></a></h2><h3>Implemented Features<span class="hx-absolute -hx-mt-20" id="implemented-features"></span>
    <a class="subheading-anchor" href="#implemented-features"></a></h3><p>Through this project, we added complete distributed tracing support to kgateway&rsquo;s AI Gateway:</p>
<ul>
<li><strong>Unified configuration interface</strong>: Users can easily configure distributed tracing through GatewayParameters</li>
<li><strong>Multi-vendor support</strong>: Support for mainstream LLM service providers like OpenAI, Anthropic, and Gemini</li>
<li><strong>Standardized tracing</strong>: Follows OpenTelemetry GenAI semantic conventions</li>
<li><strong>Flexible deployment</strong>: Supports sending to any OTLP-compatible backend storage</li>
</ul>
<h3>How to Experience the Distributed Tracing Feature<span class="hx-absolute -hx-mt-20" id="how-to-experience-the-distributed-tracing-feature"></span>
    <a class="subheading-anchor" href="#how-to-experience-the-distributed-tracing-feature"></a></h3><p>Want to experience this distributed tracing feature yourself? Follow these steps to quickly set up a complete test environment:</p>
<h4>1. Install Gateway<span class="hx-absolute -hx-mt-20" id="1-install-gateway"></span>
    <a class="subheading-anchor" href="#1-install-gateway"></a></h4><p>Follow the <a href="https://kgateway.dev/docs/envoy/latest/quickstart/" rel="noopener" target="_blank">Get started</a> guide. For the installation, choose the development version, v2.3.0-main.</p>
<h4>2. Enable AI Extension<span class="hx-absolute -hx-mt-20" id="2-enable-ai-extension"></span>
    <a class="subheading-anchor" href="#2-enable-ai-extension"></a></h4><p>As you follow the <a href="https://kgateway.dev/docs/envoy/2.1.x/ai/setup/" rel="noopener" target="_blank">Set up AI Gateway</a>, note the following configurations:</p>
<p>Upgrade kgateway and enable AI Gateway extension:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">helm upgrade -i -n kgateway-system kgateway oci://cr.kgateway.dev/kgateway-dev/charts/kgateway <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    --set gateway.aiExtension.enabled<span class="o">=</span><span class="nb">true</span> <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    --version v2.1.0-main</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Create GatewayParameters resource with distributed tracing configuration:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="l">kubectl apply -f- &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">GatewayParameters</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ai-gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway-system</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">kube</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">aiExtension</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">enabled</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">ports</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ai-monitoring</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">containerPort</span><span class="p">:</span><span class="w"> </span><span class="m">9092</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">tracing</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">endpoint</span><span class="p">:</span><span class="w"> </span><span class="s2">"http://opentelemetry-collector-traces.telemetry:4317"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">sampler</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="s2">"alwaysOn"</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">env</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">LOG_LEVEL</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">DEBUG</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">service</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">NodePort</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h4>3. Deploy OTel Stack<span class="hx-absolute -hx-mt-20" id="3-deploy-otel-stack"></span>
    <a class="subheading-anchor" href="#3-deploy-otel-stack"></a></h4><p>We only need the traces components. If you need other features (logs, metrics), please refer to: <a href="https://kgateway.dev/docs/envoy/latest/observability/otel-stack/" rel="noopener" target="_blank">OTel Stack</a></p>
<p>Install Tempo:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">helm upgrade --install tempo tempo <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --repo https://grafana.github.io/helm-charts <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --version 1.16.0 <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --namespace telemetry <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --create-namespace <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --values - <span class="s">&lt;&lt;EOF
</span></span></span><span class="line"><span class="cl"><span class="s">persistence:
</span></span></span><span class="line"><span class="cl"><span class="s">  enabled: false
</span></span></span><span class="line"><span class="cl"><span class="s">tempo:
</span></span></span><span class="line"><span class="cl"><span class="s">  receivers:
</span></span></span><span class="line"><span class="cl"><span class="s">    otlp:
</span></span></span><span class="line"><span class="cl"><span class="s">      protocols:
</span></span></span><span class="line"><span class="cl"><span class="s">        grpc:
</span></span></span><span class="line"><span class="cl"><span class="s">          endpoint: 0.0.0.0:4317
</span></span></span><span class="line"><span class="cl"><span class="s">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Install OTel Collector:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">helm upgrade --install opentelemetry-collector-traces opentelemetry-collector <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --repo https://open-telemetry.github.io/opentelemetry-helm-charts <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --version 0.127.2 <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --set <span class="nv">mode</span><span class="o">=</span>deployment <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --set image.repository<span class="o">=</span><span class="s2">"otel/opentelemetry-collector-contrib"</span> <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --set command.name<span class="o">=</span><span class="s2">"otelcol-contrib"</span> <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --namespace<span class="o">=</span>telemetry <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --create-namespace <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  -f -<span class="s">&lt;&lt;EOF
</span></span></span><span class="line"><span class="cl"><span class="s">config:
</span></span></span><span class="line"><span class="cl"><span class="s">  receivers:
</span></span></span><span class="line"><span class="cl"><span class="s">    otlp:
</span></span></span><span class="line"><span class="cl"><span class="s">      protocols:
</span></span></span><span class="line"><span class="cl"><span class="s">        grpc:
</span></span></span><span class="line"><span class="cl"><span class="s">          endpoint: 0.0.0.0:4317
</span></span></span><span class="line"><span class="cl"><span class="s">        http:
</span></span></span><span class="line"><span class="cl"><span class="s">          endpoint: 0.0.0.0:4318
</span></span></span><span class="line"><span class="cl"><span class="s">  exporters:
</span></span></span><span class="line"><span class="cl"><span class="s">    otlp/tempo:
</span></span></span><span class="line"><span class="cl"><span class="s">      endpoint: http://tempo.telemetry.svc.cluster.local:4317
</span></span></span><span class="line"><span class="cl"><span class="s">      tls:
</span></span></span><span class="line"><span class="cl"><span class="s">        insecure: true
</span></span></span><span class="line"><span class="cl"><span class="s">    debug:
</span></span></span><span class="line"><span class="cl"><span class="s">      verbosity: detailed
</span></span></span><span class="line"><span class="cl"><span class="s">  service:
</span></span></span><span class="line"><span class="cl"><span class="s">    pipelines:
</span></span></span><span class="line"><span class="cl"><span class="s">      traces:
</span></span></span><span class="line"><span class="cl"><span class="s">        receivers: [otlp]
</span></span></span><span class="line"><span class="cl"><span class="s">        processors: [batch]
</span></span></span><span class="line"><span class="cl"><span class="s">        exporters: [debug, otlp/tempo]
</span></span></span><span class="line"><span class="cl"><span class="s">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Configure Grafana for more intuitive trace observation</p>
<p>Install Grafana using <a href="https://grafana.com/docs/grafana/latest/setup-grafana/installation/helm/#deploy-grafana-using-helm-charts" rel="noopener" target="_blank">deploy-grafana-using-helm-charts</a>.</p>
<p>Configure Tempo as a data source, and fill in the URL field with: <code>http://tempo.telemetry:3100</code>





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/config-data-source-tempo.png" width="750px" /> <figcaption style="font-style: italic;">Configure Tempo as data source</figcaption></figure></div></p>
<h4>4. Create Tracing Policy<span class="hx-absolute -hx-mt-20" id="4-create-tracing-policy"></span>
    <a class="subheading-anchor" href="#4-create-tracing-policy"></a></h4><div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="l">kubectl apply -f- &lt;&lt;EOF</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPListenerPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">tracing-policy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway-system</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">ai-gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">tracing</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">provider</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">openTelemetry</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">serviceName</span><span class="p">:</span><span class="w"> </span><span class="l">http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">grpcService</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">          </span><span class="nt">backendRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">otel-collector-traces</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">telemetry</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">            </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">4317</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">spawnUpstreamSpan</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="l">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h4>5. Send Test Request<span class="hx-absolute -hx-mt-20" id="5-send-test-request"></span>
    <a class="subheading-anchor" href="#5-send-test-request"></a></h4><p>Send a test request using <a href="https://kgateway.dev/docs/envoy/2.1.x/ai/ollama/" rel="noopener" target="_blank">Ollama for local LLMs</a>:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl -v <span class="s2">"localhost:8080/ollama"</span> <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    -H <span class="s2">"Content-Type: application/json"</span> <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    -d <span class="s1">'{
</span></span></span><span class="line"><span class="cl"><span class="s1">        "model": "llama3.2",
</span></span></span><span class="line"><span class="cl"><span class="s1">        "messages": [
</span></span></span><span class="line"><span class="cl"><span class="s1">            {
</span></span></span><span class="line"><span class="cl"><span class="s1">                "role": "system",
</span></span></span><span class="line"><span class="cl"><span class="s1">                "content": "You are a helpful assistant."
</span></span></span><span class="line"><span class="cl"><span class="s1">            },
</span></span></span><span class="line"><span class="cl"><span class="s1">            {
</span></span></span><span class="line"><span class="cl"><span class="s1">                "role": "user",
</span></span></span><span class="line"><span class="cl"><span class="s1">                "content": "Hello!"
</span></span></span><span class="line"><span class="cl"><span class="s1">            }
</span></span></span><span class="line"><span class="cl"><span class="s1">        ]
</span></span></span><span class="line"><span class="cl"><span class="s1">    }'</span> <span class="p">|</span> jq</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h4>6. Verify Distributed Tracing<span class="hx-absolute -hx-mt-20" id="6-verify-distributed-tracing"></span>
    <a class="subheading-anchor" href="#6-verify-distributed-tracing"></a></h4><p>Check if traces are being collected properly via command line:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl -n telemetry logs deploy/opentelemetry-collector-traces <span class="p">|</span> grep <span class="s1">'llama'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>You should see output similar to this:</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><pre><code>Name           : gen_ai.request generate_content llama3.2
    -&gt; gen_ai.request.model: Str(llama3.2)
    -&gt; gen_ai.response.model: Str(llama3.2)</code></pre></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Observe traces more intuitively through Grafana:





<div><figure class="hx-block dark:hx-hidden"><img alt="" class="hx-block dark:hx-hidden" src="/blog/tempo-traces-result.png" width="750px" /> <figcaption style="font-style: italic;">Tempo traces visualization in Grafana</figcaption></figure></div></p>
<p>This indicates that the distributed tracing feature is working properly, and your AI Gateway requests are being completely traced and recorded!</p>
<h3>Personal Gains<span class="hx-absolute -hx-mt-20" id="personal-gains"></span>
    <a class="subheading-anchor" href="#personal-gains"></a></h3><p>Through LFX Mentorship, I not only completed a meaningful technical project, but more importantly:</p>
<ul>
<li><strong>Deep open source participation</strong>: Transformed from a user to a contributor, deeply understanding how open source projects operate</li>
<li><strong>Technical skill improvement</strong>: Gained deeper understanding in distributed tracing, AI Gateway architecture, and other areas</li>
<li><strong>Enhanced collaboration skills</strong>: Learned how to communicate and collaborate effectively in international teams</li>
</ul>
<h2>Advice for Students Who Want to Participate in Open Source<span class="hx-absolute -hx-mt-20" id="advice-for-students-who-want-to-participate-in-open-source"></span>
    <a class="subheading-anchor" href="#advice-for-students-who-want-to-participate-in-open-source"></a></h2><h3>LFX Mentorship Application Strategy<span class="hx-absolute -hx-mt-20" id="lfx-mentorship-application-strategy"></span>
    <a class="subheading-anchor" href="#lfx-mentorship-application-strategy"></a></h3><ol>
<li><strong>Prepare in advance</strong>: Start following and participating in target projects before applying</li>
<li><strong>Proactive contribution</strong>: Build community trust through small PRs or issue reports</li>
<li><strong>Deep understanding</strong>: Ensure you truly understand the project&rsquo;s tech stack and business value</li>
<li><strong>Demonstrate capability</strong>: Prove your technical ability with actual code contributions</li>
</ol>
<h3>Project Execution Advice<span class="hx-absolute -hx-mt-20" id="project-execution-advice"></span>
    <a class="subheading-anchor" href="#project-execution-advice"></a></h3><ol>
<li><strong>Maintain communication</strong>: Regularly sync progress and issues with mentors and community</li>
<li><strong>Incremental approach</strong>: Start with small features, gradually building complete solutions</li>
<li><strong>Value testing</strong>: Write unit tests first, then integration tests</li>
<li><strong>Documentation first</strong>: Good documentation helps the community better understand and use your contributions</li>
</ol>
<h2>Conclusion<span class="hx-absolute -hx-mt-20" id="conclusion"></span>
    <a class="subheading-anchor" href="#conclusion"></a></h2><p>From applying for LFX Mentorship to completing the AI Gateway distributed tracing project, this experience gave me a deep understanding of the power and value of open source communities. Technology itself is important, but what&rsquo;s more important is learning how to collaborate with the community, how to solve real problems, and how to create value for a broader user base.</p>
<p>If you&rsquo;re also interested in open source contribution, why not start today? Find a project you&rsquo;re interested in and submit your first issue or PR. Every small contribution is a step toward bigger goals.</p>
<p>Finally, special thanks to my mentors <a href="https://github.com/npolshakova" rel="noopener" target="_blank">@Nina</a> and <a href="https://github.com/andy-fong" rel="noopener" target="_blank">@Andy</a> for taking their valuable time to guide and help me complete this project!</p>
