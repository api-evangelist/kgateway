---
title: "What Comes After Ingress NGINX? A Migration Guide to Gateway API"
url: "/blog/ingress-nginx-migration-gateway-api/"
date: "Tue, 27 Jan 2026 10:00:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>With the latest news of the Ingress NGINX Controller migration, thousands of engineers are attempting to figure out how they can migrate to Gateway API, the new standard for gateway traffic to various applications running within Kubernetes. Because teams could have hundreds or more Ingress manifests using the Controller, a method of turning those manifests into Gateway API core and implementation-specific objects is necessary.</p>
<p>In this blog post, you’ll learn a bit about the “why” behind the deprecation and how to migrate your Ingress NGINX configurations.</p>
<h2>Why The Deprecation?<span class="hx-absolute -hx-mt-20" id="why-the-deprecation"></span>
    <a class="subheading-anchor" href="#why-the-deprecation"></a></h2><p>When Kubernetes first came out, there was a need to show an example of how to manage ingress traffic. Because of that, Ingress NGINX was created. However, because of its breadth of usage and support across the various cloud providers, it ended up becoming a standard. Since the standard was created, there were still only a handful of people working on maintenance and new features around Ingress NGINX.</p>
<h2>Implementing Ingress2gateway Via Kgateway<span class="hx-absolute -hx-mt-20" id="implementing-ingress2gateway-via-kgateway"></span>
    <a class="subheading-anchor" href="#implementing-ingress2gateway-via-kgateway"></a></h2><p>With what you’ve learned so far throughout this blog post regarding the Ingress NGINX retirement, you may be thinking to yourself, “Alright, so how do I keep the lights on and remove the Controller?”</p>
<p>The answer is by migrating to Gateway API.</p>
<p>But because that would be incredibly cumbersome to do manually, you can use the Ingress2gateway migration tool. Since Gateway API was designed to be extensible, migrating most production use cases also requires implementation-specific objects. For example, migrating an Ingress with the <code>“nginx.ingress.kubernetes.io/auth-type: basic”</code> annotation requires an implementation-specific object such as kgateway’s TrafficPolicy.</p>
<div class="hx-overflow-x-auto hx-mt-6 hx-flex hx-flex-col hx-rounded-lg hx-border hx-py-4 hx-px-4 contrast-more:hx-border-current contrast-more:dark:hx-border-current hx-border-blue-200 hx-bg-blue-100 hx-text-blue-900 dark:hx-border-blue-200/30 dark:hx-bg-blue-900/30 dark:hx-text-blue-200">
  <p class="hx-flex hx-items-center hx-font-medium"><svg class="hx-inline-block hx-align-middle hx-mr-2" fill="none" height="16px" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" stroke-linecap="round" stroke-linejoin="round"></svg>Note</p>

  <div class="hx-w-full hx-min-w-0 hx-leading-7">
    <div class="hx-mt-6 hx-leading-7 first:hx-mt-0"><p>Technically, you can keep the underlying Ingress object and change the ingressClassName. Why not just do that? Because the Kubernetes project is moving toward Gateway API. In short, there’s no reason to migrate to something that’s considered deprecated by the community. If you just rename the Ingress objects, you&rsquo;re creating more tech debt for yourself down the road when you eventually have to migrate.</p>
</div>
  </div>
</div>
<p>In the next two sections, you’ll first deploy an object with Ingress NGINX into your Kubernetes cluster and then you’ll learn how to migrate it to <strong>kgateway</strong>, a <a href="https://gateway-api.sigs.k8s.io/implementations/#kgateway" rel="noopener" target="_blank">conformant Gateway API implementation</a>.</p>
<h2>Using The Migration Tool<span class="hx-absolute -hx-mt-20" id="using-the-migration-tool"></span>
    <a class="subheading-anchor" href="#using-the-migration-tool"></a></h2><p>The first step is to download the migration tool. You can find the installation options for your Operating System in the <a href="/docs/envoy/latest/migrate/install/">docs here</a>.</p>
<p>You can verify that the command works by running the build/binary, as in the following output.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway
</span></span><span class="line"><span class="cl">Convert Ingress manifests to Gateway API manifests
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl">Usage:
</span></span><span class="line"><span class="cl">  ingress2gateway <span class="o">[</span>command<span class="o">]</span>
</span></span><span class="line"><span class="cl">
</span></span><span class="line"><span class="cl">Available Commands:
</span></span><span class="line"><span class="cl">  completion  Generate the autocompletion script <span class="k">for</span> the specified shell
</span></span><span class="line"><span class="cl">  <span class="nb">help</span>        Help about any <span class="nb">command</span>
</span></span><span class="line"><span class="cl">  print       Prints Gateway API objects generated from ingress and provider-specific resources.
</span></span><span class="line"><span class="cl">  version     Print the version number of ingress2gateway</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>With the binary, you can use the print command with the providers and emitter flags to do the following:</p>
<ul>
<li>Specify that you want the source to be the Ingress NGINX Controller.</li>
<li>Choose what you want to migrate to (the &ldquo;emitter&rdquo;), which in this case is kgateway using the standard from Kubernetes Gateway API CRDs (kgateway is the current supported implementation in this downstream fork).</li>
<li>Convert the source objects to Kubernetes Gateway API objects that work with the chosen emitter.</li>
</ul>
<p>For example, to print the Kubernetes objects that the conversion produces, run the following command.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway print --providers<span class="o">=</span>ingress-nginx --emitter<span class="o">=</span>kgateway</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>Next, you’ll find three key use cases that many organizations deploy within production environments and how to convert them with ingress2gateway.</p>
<h2>Deploying Ingress NGINX Implementations<span class="hx-absolute -hx-mt-20" id="deploying-ingress-nginx-implementations"></span>
    <a class="subheading-anchor" href="#deploying-ingress-nginx-implementations"></a></h2><p>In this section, you&rsquo;ll cover three key scenarios that many production-level environments use:</p>
<ol>
<li>TLS/SSL</li>
<li>Auth</li>
<li>CORS</li>
</ol>
<p>The goal with the three test cases is to ensure that the ingress2gateway tool works as expected for various use cases depending on your environment.</p>
<h3>Deployment Setup<span class="hx-absolute -hx-mt-20" id="deployment-setup"></span>
    <a class="subheading-anchor" href="#deployment-setup"></a></h3><p>Deploy a test application, which is only a simple HTTP service. It runs in a Kubernetes Deployment and has a Kubernetes Service.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl apply -f - <span class="s">&lt;&lt;EOF
</span></span></span><span class="line"><span class="cl"><span class="s">apiVersion: apps/v1
</span></span></span><span class="line"><span class="cl"><span class="s">kind: Deployment
</span></span></span><span class="line"><span class="cl"><span class="s">metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">  name: api-app
</span></span></span><span class="line"><span class="cl"><span class="s">  namespace: default
</span></span></span><span class="line"><span class="cl"><span class="s">spec:
</span></span></span><span class="line"><span class="cl"><span class="s">  replicas: 2
</span></span></span><span class="line"><span class="cl"><span class="s">  selector:
</span></span></span><span class="line"><span class="cl"><span class="s">    matchLabels:
</span></span></span><span class="line"><span class="cl"><span class="s">      app: api-app
</span></span></span><span class="line"><span class="cl"><span class="s">  template:
</span></span></span><span class="line"><span class="cl"><span class="s">    metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">      labels:
</span></span></span><span class="line"><span class="cl"><span class="s">        app: api-app
</span></span></span><span class="line"><span class="cl"><span class="s">    spec:
</span></span></span><span class="line"><span class="cl"><span class="s">      containers:
</span></span></span><span class="line"><span class="cl"><span class="s">      - name: httpbin
</span></span></span><span class="line"><span class="cl"><span class="s">        image: kennethreitz/httpbin
</span></span></span><span class="line"><span class="cl"><span class="s">        ports:
</span></span></span><span class="line"><span class="cl"><span class="s">        - containerPort: 80
</span></span></span><span class="line"><span class="cl"><span class="s">---
</span></span></span><span class="line"><span class="cl"><span class="s">apiVersion: v1
</span></span></span><span class="line"><span class="cl"><span class="s">kind: Service
</span></span></span><span class="line"><span class="cl"><span class="s">metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">  name: api-service
</span></span></span><span class="line"><span class="cl"><span class="s">  namespace: default
</span></span></span><span class="line"><span class="cl"><span class="s">spec:
</span></span></span><span class="line"><span class="cl"><span class="s">  selector:
</span></span></span><span class="line"><span class="cl"><span class="s">    app: api-app
</span></span></span><span class="line"><span class="cl"><span class="s">  ports:
</span></span></span><span class="line"><span class="cl"><span class="s">  - port: 80
</span></span></span><span class="line"><span class="cl"><span class="s">    targetPort: 80
</span></span></span><span class="line"><span class="cl"><span class="s">---
</span></span></span><span class="line"><span class="cl"><span class="s">apiVersion: apps/v1
</span></span></span><span class="line"><span class="cl"><span class="s">kind: Deployment
</span></span></span><span class="line"><span class="cl"><span class="s">metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">  name: admin-app
</span></span></span><span class="line"><span class="cl"><span class="s">  namespace: default
</span></span></span><span class="line"><span class="cl"><span class="s">spec:
</span></span></span><span class="line"><span class="cl"><span class="s">  replicas: 1
</span></span></span><span class="line"><span class="cl"><span class="s">  selector:
</span></span></span><span class="line"><span class="cl"><span class="s">    matchLabels:
</span></span></span><span class="line"><span class="cl"><span class="s">      app: admin-app
</span></span></span><span class="line"><span class="cl"><span class="s">  template:
</span></span></span><span class="line"><span class="cl"><span class="s">    metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">      labels:
</span></span></span><span class="line"><span class="cl"><span class="s">        app: admin-app
</span></span></span><span class="line"><span class="cl"><span class="s">    spec:
</span></span></span><span class="line"><span class="cl"><span class="s">      containers:
</span></span></span><span class="line"><span class="cl"><span class="s">      - name: nginx
</span></span></span><span class="line"><span class="cl"><span class="s">        image: nginx:alpine
</span></span></span><span class="line"><span class="cl"><span class="s">        ports:
</span></span></span><span class="line"><span class="cl"><span class="s">        - containerPort: 80
</span></span></span><span class="line"><span class="cl"><span class="s">---
</span></span></span><span class="line"><span class="cl"><span class="s">apiVersion: v1
</span></span></span><span class="line"><span class="cl"><span class="s">kind: Service
</span></span></span><span class="line"><span class="cl"><span class="s">metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">  name: admin-service
</span></span></span><span class="line"><span class="cl"><span class="s">  namespace: default
</span></span></span><span class="line"><span class="cl"><span class="s">spec:
</span></span></span><span class="line"><span class="cl"><span class="s">  selector:
</span></span></span><span class="line"><span class="cl"><span class="s">    app: admin-app
</span></span></span><span class="line"><span class="cl"><span class="s">  ports:
</span></span></span><span class="line"><span class="cl"><span class="s">  - port: 80
</span></span></span><span class="line"><span class="cl"><span class="s">    targetPort: 80
</span></span></span><span class="line"><span class="cl"><span class="s">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<h3>Use Case 1: TLS/SSL<span class="hx-absolute -hx-mt-20" id="use-case-1-tlsssl"></span>
    <a class="subheading-anchor" href="#use-case-1-tlsssl"></a></h3><ol>
<li>
<p>Create certs for testing.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">openssl req -x509 -nodes -days <span class="m">365</span> -newkey rsa:2048 -keyout tls.key -out tls.crt -subj <span class="s2">"/CN=localhost/O=dev”</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Create a new Kubernetes secret for TLS certs.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl create secret tls my-tls-cert <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --cert<span class="o">=</span>tls.crt <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>  --key<span class="o">=</span>tls.key</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Apply the Ingress configuration.</p>
<div class="hx-overflow-x-auto hx-mt-6 hx-flex hx-flex-col hx-rounded-lg hx-border hx-py-4 hx-px-4 contrast-more:hx-border-current contrast-more:dark:hx-border-current hx-border-blue-200 hx-bg-blue-100 hx-text-blue-900 dark:hx-border-blue-200/30 dark:hx-bg-blue-900/30 dark:hx-text-blue-200">
  <p class="hx-flex hx-items-center hx-font-medium"><svg class="hx-inline-block hx-align-middle hx-mr-2" fill="none" height="16px" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" stroke-linecap="round" stroke-linejoin="round"></svg>Note</p>

  <div class="hx-w-full hx-min-w-0 hx-leading-7">
    <div class="hx-mt-6 hx-leading-7 first:hx-mt-0"><p>Notice the annotation that specifies which control plane/gateway you’re switching to. The reason is to ensure that the Gateway object you’re planning on using is supported and works as expected—using the Kubernetes Gateway API CRDs allows you to stay as agnostic as possible.</p>
</div>
  </div>
</div>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl apply -f - <span class="s">&lt;&lt;EOF
</span></span></span><span class="line"><span class="cl"><span class="s">apiVersion: networking.k8s.io/v1
</span></span></span><span class="line"><span class="cl"><span class="s">kind: Ingress
</span></span></span><span class="line"><span class="cl"><span class="s">metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">  name: tls-ingress
</span></span></span><span class="line"><span class="cl"><span class="s">  namespace: default
</span></span></span><span class="line"><span class="cl"><span class="s">  annotations:
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/ssl-redirect: "true"
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
</span></span></span><span class="line"><span class="cl"><span class="s">spec:
</span></span></span><span class="line"><span class="cl"><span class="s">  ingressClassName: nginx
</span></span></span><span class="line"><span class="cl"><span class="s">  tls:
</span></span></span><span class="line"><span class="cl"><span class="s">  - hosts:
</span></span></span><span class="line"><span class="cl"><span class="s">    - api.example.com
</span></span></span><span class="line"><span class="cl"><span class="s">    secretName: my-tls-cert
</span></span></span><span class="line"><span class="cl"><span class="s">  rules:
</span></span></span><span class="line"><span class="cl"><span class="s">  - host: api.example.com
</span></span></span><span class="line"><span class="cl"><span class="s">    http:
</span></span></span><span class="line"><span class="cl"><span class="s">      paths:
</span></span></span><span class="line"><span class="cl"><span class="s">      - path: /
</span></span></span><span class="line"><span class="cl"><span class="s">        pathType: Prefix
</span></span></span><span class="line"><span class="cl"><span class="s">        backend:
</span></span></span><span class="line"><span class="cl"><span class="s">          service:
</span></span></span><span class="line"><span class="cl"><span class="s">            name: api-service
</span></span></span><span class="line"><span class="cl"><span class="s">            port:
</span></span></span><span class="line"><span class="cl"><span class="s">              number: 80
</span></span></span><span class="line"><span class="cl"><span class="s">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Run the ingress2gateway tool. You should see output similar to the following.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway print --providers<span class="o">=</span>ingress-nginx --emitter<span class="o">=</span>kgateway</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-v0.3.0</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">gatewayClassName</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">hostname</span><span class="p">:</span><span class="w"> </span><span class="l">api.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">api-example-com-http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">hostname</span><span class="p">:</span><span class="w"> </span><span class="l">api.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">api-example-com-https</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">443</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPS</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">tls</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">certificateRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">my-tls-cert</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parents</span><span class="p">:</span><span class="w"> </span><span class="p">[]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-v0.3.0</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">tls-ingress-api-example-com-https</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hostnames</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="l">api.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parentRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">sectionName</span><span class="p">:</span><span class="w"> </span><span class="l">api-example-com-https</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">api-service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parents</span><span class="p">:</span><span class="w"> </span><span class="p">[]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-v0.3.0</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">tls-ingress-api-example-com-http-redirect</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hostnames</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="l">api.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parentRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">sectionName</span><span class="p">:</span><span class="w"> </span><span class="l">api-example-com-http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">filters</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">requestRedirect</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">scheme</span><span class="p">:</span><span class="w"> </span><span class="l">https</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">statusCode</span><span class="p">:</span><span class="w"> </span><span class="m">301</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">RequestRedirect</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
</ol>
<h3>Functional Tests<span class="hx-absolute -hx-mt-20" id="functional-tests"></span>
    <a class="subheading-anchor" href="#functional-tests"></a></h3><ol>
<li>
<p>Apply the new objects that were printed above.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway print --providers<span class="o">=</span>ingress-nginx --emitter<span class="o">=</span>kgateway <span class="p">|</span> kubectl apply -f -</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Verify the Gateway is accepted.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl get gateway nginx -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.conditions[?(@.type=="Accepted")].status}'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Ensure that the HTTP Routes are attached.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl get httproute -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{range .items[*]}{.metadata.name}: {.status.parents[0].conditions[?(@.type=="Accepted")].status}{"\n"}{end}'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Get the IP address of the Gateway.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl"><span class="nv">GATEWAY_IP</span><span class="o">=</span><span class="k">$(</span>kubectl get gateway nginx -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.addresses[0].value}'</span><span class="k">)</span> <span class="nb">echo</span> <span class="nv">$GATEWAY_IP</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Test the HTTP redirect (should return a 301).</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl -I --resolve api.example.com:80:<span class="nv">$GATEWAY_IP</span> http://api.example.com/</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Test the HTTPS route.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl -k --resolve api.example.com:443:<span class="nv">$GATEWAY_IP</span> https://api.example.com/</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
</ol>
<table>
  <thead>
      <tr>
          <th style="text-align: left;">Test</th>
          <th style="text-align: left;">Result</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;">Gateway accepted</td>
          <td style="text-align: left;">True</td>
      </tr>
      <tr>
          <td style="text-align: left;">HTTPRoutes attached</td>
          <td style="text-align: left;">Both True</td>
      </tr>
      <tr>
          <td style="text-align: left;">HTTP redirect</td>
          <td style="text-align: left;">301 to <a href="https://api.example.com/" rel="noopener" target="_blank">https://api.example.com/</a></td>
      </tr>
      <tr>
          <td style="text-align: left;">HTTPS route</td>
          <td style="text-align: left;">Returns httpbin.org page from backend</td>
      </tr>
  </tbody>
</table>
<h2>Use Case 2: Auth<span class="hx-absolute -hx-mt-20" id="use-case-2-auth"></span>
    <a class="subheading-anchor" href="#use-case-2-auth"></a></h2><ol>
<li>
<p>Create a username and password (the secret is stored in a Kubernetes secret).</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">htpasswd -c auth admin
</span></span><span class="line"><span class="cl">kubectl create secret generic basic-auth --from-file<span class="o">=</span>auth</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Apply the Ingress configuration.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl apply -f - <span class="s">&lt;&lt;EOF
</span></span></span><span class="line"><span class="cl"><span class="s">apiVersion: networking.k8s.io/v1
</span></span></span><span class="line"><span class="cl"><span class="s">kind: Ingress
</span></span></span><span class="line"><span class="cl"><span class="s">metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">  name: auth-ingress
</span></span></span><span class="line"><span class="cl"><span class="s">  namespace: default
</span></span></span><span class="line"><span class="cl"><span class="s">  annotations:
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/auth-type: basic
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/auth-secret: basic-auth
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/auth-secret-type: auth-file
</span></span></span><span class="line"><span class="cl"><span class="s">spec:
</span></span></span><span class="line"><span class="cl"><span class="s">  ingressClassName: nginx
</span></span></span><span class="line"><span class="cl"><span class="s">  rules:
</span></span></span><span class="line"><span class="cl"><span class="s">  - host: admin.example.com
</span></span></span><span class="line"><span class="cl"><span class="s">    http:
</span></span></span><span class="line"><span class="cl"><span class="s">      paths:
</span></span></span><span class="line"><span class="cl"><span class="s">      - path: /
</span></span></span><span class="line"><span class="cl"><span class="s">        pathType: Prefix
</span></span></span><span class="line"><span class="cl"><span class="s">        backend:
</span></span></span><span class="line"><span class="cl"><span class="s">          service:
</span></span></span><span class="line"><span class="cl"><span class="s">            name: admin-service
</span></span></span><span class="line"><span class="cl"><span class="s">            port:
</span></span></span><span class="line"><span class="cl"><span class="s">              number: 80
</span></span></span><span class="line"><span class="cl"><span class="s">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Use the ingress2gateway tool to convert it.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway print --providers<span class="o">=</span>ingress-nginx --emitter<span class="o">=</span>kgateway</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>You should see output similar to the following.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-v0.3.0</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">gatewayClassName</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">hostname</span><span class="p">:</span><span class="w"> </span><span class="l">admin.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">admin-example-com-http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w"> </span>{}<span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-v0.3.0</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">auth-ingress-admin-example-com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hostnames</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="l">admin.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parentRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">admin-service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parents</span><span class="p">:</span><span class="w"> </span><span class="p">[]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">auth-ingress</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">basicAuth</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">secretRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">key</span><span class="p">:</span><span class="w"> </span><span class="l">auth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">basic-auth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">auth-ingress-admin-example-com</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
</ol>
<h3>Functional Tests<span class="hx-absolute -hx-mt-20" id="functional-tests-1"></span>
    <a class="subheading-anchor" href="#functional-tests-1"></a></h3><ol>
<li>
<p>Create a basic-auth secret.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">htpasswd -nbB admin testpass123 &gt; /tmp/htpasswd
</span></span><span class="line"><span class="cl">kubectl create secret generic basic-auth --from-file<span class="o">=</span>.htaccess<span class="o">=</span>/tmp/htpasswd -n default</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Apply the new converted objects.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway print --providers<span class="o">=</span>ingress-nginx --emitter<span class="o">=</span>kgateway <span class="p">|</span> kubectl apply -f -</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Verify the Gateway is accepted.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl get gateway nginx -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.conditions[?(@.type=="Accepted")].status}'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Ensure the HTTP route is attached.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl get httproute auth-ingress-admin-example-com -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.parents[0].conditions[?(@.type=="Accepted")].status}'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Get the Gateway IP.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl"><span class="nv">GATEWAY_IP</span><span class="o">=</span><span class="k">$(</span>kubectl get gateway nginx -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.addresses[0].value}'</span><span class="k">)</span> <span class="nb">echo</span> <span class="nv">$GATEWAY_IP</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Test without auth. This should fail with a 401.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl -I --resolve admin.example.com:80:<span class="nv">$GATEWAY_IP</span> http://admin.example.com/</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Test with auth.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl --resolve admin.example.com:80:<span class="nv">$GATEWAY_IP</span> -u <span class="s2">"admin:testpass123"</span> http://admin.example.com/</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
</ol>
<table>
  <thead>
      <tr>
          <th style="text-align: left;">Test</th>
          <th style="text-align: left;">Result</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;">Gateway accepted</td>
          <td style="text-align: left;">True</td>
      </tr>
      <tr>
          <td style="text-align: left;">HTTPRoute attached</td>
          <td style="text-align: left;">True</td>
      </tr>
      <tr>
          <td style="text-align: left;">TrafficPolicy attached</td>
          <td style="text-align: left;">True</td>
      </tr>
      <tr>
          <td style="text-align: left;">Request without auth</td>
          <td style="text-align: left;">401 Unauthorized</td>
      </tr>
      <tr>
          <td style="text-align: left;">Request with valid auth (admin:testpass123)</td>
          <td style="text-align: left;">200 OK</td>
      </tr>
  </tbody>
</table>
<h2>Use Case 3: CORS<span class="hx-absolute -hx-mt-20" id="use-case-3-cors"></span>
    <a class="subheading-anchor" href="#use-case-3-cors"></a></h2><ol>
<li>
<p>Apply the Ingress configuration below.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl apply -f - <span class="s">&lt;&lt;EOF
</span></span></span><span class="line"><span class="cl"><span class="s">apiVersion: networking.k8s.io/v1
</span></span></span><span class="line"><span class="cl"><span class="s">kind: Ingress
</span></span></span><span class="line"><span class="cl"><span class="s">metadata:
</span></span></span><span class="line"><span class="cl"><span class="s">  name: cors-ingress
</span></span></span><span class="line"><span class="cl"><span class="s">  namespace: default
</span></span></span><span class="line"><span class="cl"><span class="s">  annotations:
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/enable-cors: "true"
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/cors-allow-origin: "https://app.example.com,https://dashboard.example.com"
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/cors-allow-methods: "GET,POST,PUT,DELETE,OPTIONS"
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/cors-allow-headers: "Authorization,Content-Type,X-Requested-With"
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/cors-expose-headers: "X-Custom-Header,X-Request-ID"
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
</span></span></span><span class="line"><span class="cl"><span class="s">    nginx.ingress.kubernetes.io/cors-max-age: "7200"
</span></span></span><span class="line"><span class="cl"><span class="s">spec:
</span></span></span><span class="line"><span class="cl"><span class="s">  ingressClassName: nginx
</span></span></span><span class="line"><span class="cl"><span class="s">  rules:
</span></span></span><span class="line"><span class="cl"><span class="s">  - host: api.example.com
</span></span></span><span class="line"><span class="cl"><span class="s">    http:
</span></span></span><span class="line"><span class="cl"><span class="s">      paths:
</span></span></span><span class="line"><span class="cl"><span class="s">      - path: /v1
</span></span></span><span class="line"><span class="cl"><span class="s">        pathType: Prefix
</span></span></span><span class="line"><span class="cl"><span class="s">        backend:
</span></span></span><span class="line"><span class="cl"><span class="s">          service:
</span></span></span><span class="line"><span class="cl"><span class="s">            name: api-service
</span></span></span><span class="line"><span class="cl"><span class="s">            port:
</span></span></span><span class="line"><span class="cl"><span class="s">              number: 80
</span></span></span><span class="line"><span class="cl"><span class="s">EOF</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Run the <code>ingress2gateway</code> tool.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway print --providers<span class="o">=</span>ingress-nginx --emitter<span class="o">=</span>kgateway</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
<p>You should see output similar to the following.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-yaml"><span class="line"><span class="cl"><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">Gateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-v0.3.0</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">gatewayClassName</span><span class="p">:</span><span class="w"> </span><span class="l">kgateway</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">listeners</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">hostname</span><span class="p">:</span><span class="w"> </span><span class="l">admin.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">admin-example-com-http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">hostname</span><span class="p">:</span><span class="w"> </span><span class="l">api.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">api-example-com-http</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">protocol</span><span class="p">:</span><span class="w"> </span><span class="l">HTTP</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w"> </span>{}<span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-v0.3.0</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">auth-ingress-admin-example-com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hostnames</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="l">admin.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parentRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">admin-service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parents</span><span class="p">:</span><span class="w"> </span><span class="p">[]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">gateway.networking.k8s.io/generator</span><span class="p">:</span><span class="w"> </span><span class="l">ingress2gateway-dev</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">cors-ingress-api-example-com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">hostnames</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="l">api.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parentRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">nginx</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">rules</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">backendRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">api-service</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">port</span><span class="p">:</span><span class="w"> </span><span class="m">80</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">matches</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="nt">path</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">type</span><span class="p">:</span><span class="w"> </span><span class="l">PathPrefix</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">        </span><span class="nt">value</span><span class="p">:</span><span class="w"> </span><span class="l">/v1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">parents</span><span class="p">:</span><span class="w"> </span><span class="p">[]</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">auth-ingress</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">basicAuth</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">secretRef</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">key</span><span class="p">:</span><span class="w"> </span><span class="l">auth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">      </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">basic-auth</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">auth-ingress-admin-example-com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">ancestors</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nn">---</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.kgateway.dev/v1alpha1</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">TrafficPolicy</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">metadata</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">cors-ingress</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">namespace</span><span class="p">:</span><span class="w"> </span><span class="l">default</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">spec</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">cors</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">allowCredentials</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">allowHeaders</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">Authorization</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">Content-Type</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">X-Requested-With</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">allowMethods</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">GET</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">POST</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">PUT</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">DELETE</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">OPTIONS</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">allowOrigins</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">https://app.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">https://dashboard.example.com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">exposeHeaders</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">X-Custom-Header</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span>- <span class="l">X-Request-ID</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">maxAge</span><span class="p">:</span><span class="w"> </span><span class="m">7200</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">targetRefs</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span>- <span class="nt">group</span><span class="p">:</span><span class="w"> </span><span class="l">gateway.networking.k8s.io</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l">HTTPRoute</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">    </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l">cors-ingress-api-example-com</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w"></span><span class="nt">status</span><span class="p">:</span><span class="w">
</span></span></span><span class="line"><span class="cl"><span class="w">  </span><span class="nt">ancestors</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
</ol>
<h3>Functional Tests<span class="hx-absolute -hx-mt-20" id="functional-tests-2"></span>
    <a class="subheading-anchor" href="#functional-tests-2"></a></h3><ol>
<li>
<p>Apply the new converted objects.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">./ingress2gateway print --providers<span class="o">=</span>ingress-nginx --emitter<span class="o">=</span>kgateway <span class="p">|</span> kubectl apply -f -</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Verify the Gateway is accepted.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl get gateway nginx -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.conditions[?(@.type=="Accepted")].status}'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Confirm the HTTP route is attached.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">kubectl get httproute cors-ingress-api-example-com -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.parents[0].conditions[?(@.type=="Accepted")].status}'</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Capture the Gateway IP.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl"><span class="nv">GATEWAY_IP</span><span class="o">=</span><span class="k">$(</span>kubectl get gateway nginx -o <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.addresses[0].value}'</span><span class="k">)</span> <span class="nb">echo</span> <span class="nv">$GATEWAY_IP</span></span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Test the CORS preflight request. A 200 is expected.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl -I --resolve api.example.com:80:<span class="nv">$GATEWAY_IP</span> -X OPTIONS -H <span class="s2">"Origin: https://app.example.com"</span> -H <span class="s2">"Access-Control-Request-Method: POST"</span> -H <span class="s2">"Access-Control-Request-Headers: Authorization,Content-Type"</span> http://api.example.com/v1/</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Test with a disallowed origin.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl -I --resolve api.example.com:80:<span class="nv">$GATEWAY_IP</span> -X OPTIONS -H <span class="s2">"Origin: https://evil.example.com"</span> -H <span class="s2">"Access-Control-Request-Method: POST"</span> http://api.example.com/v1/</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
<li>
<p>Test with an allowed origin.</p>
<div class="hextra-code-block hx-relative hx-mt-6 first:hx-mt-0 hx-group/code">

<div><div class="highlight"><pre class="chroma" tabindex="0"><code class="language-bash"><span class="line"><span class="cl">curl -I --resolve api.example.com:80:<span class="nv">$GATEWAY_IP</span> -H <span class="s2">"Origin: https://app.example.com"</span> http://api.example.com/v1/</span></span></code></pre></div></div><div class="hextra-code-copy-btn-container hx-opacity-0 hx-transition group-hover/code:hx-opacity-100 hx-flex hx-gap-1 hx-absolute hx-m-[11px] hx-right-0 hx-top-0">
  <button class="hextra-code-copy-btn hx-group/copybtn hx-transition-all active:hx-opacity-50 hx-bg-primary-700/5 hx-border hx-border-black/5 hx-text-gray-600 hover:hx-text-gray-900 hx-rounded-md hx-p-1.5 dark:hx-bg-primary-300/10 dark:hx-border-white/10 dark:hx-text-gray-400 dark:hover:hx-text-gray-50" title="Copy code">
    <div class="copy-icon group-[.copied]/copybtn:hx-hidden hx-pointer-events-none hx-h-4 hx-w-4"></div>
    <div class="success-icon hx-hidden group-[.copied]/copybtn:hx-block hx-pointer-events-none hx-h-4 hx-w-4"></div>
  </button>
</div>
</div>
</li>
</ol>
<table>
  <thead>
      <tr>
          <th style="text-align: left;">Test</th>
          <th style="text-align: left;">Result</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;">Gateway accepted</td>
          <td style="text-align: left;">True</td>
      </tr>
      <tr>
          <td style="text-align: left;">HTTPRoutes attached</td>
          <td style="text-align: left;">Both True</td>
      </tr>
      <tr>
          <td style="text-align: left;">Policies attached</td>
          <td style="text-align: left;">Both True</td>
      </tr>
  </tbody>
</table>
<p><strong>CORS Tests</strong></p>
<table>
  <thead>
      <tr>
          <th style="text-align: left;">Test</th>
          <th style="text-align: left;">Result</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;">Preflight with allowed origin (<a href="https://app.example.com" rel="noopener" target="_blank">https://app.example.com</a>)</td>
          <td style="text-align: left;">200 OK with correct CORS headers</td>
      </tr>
      <tr>
          <td style="text-align: left;">Preflight with disallowed origin</td>
          <td style="text-align: left;">Passed through to backend (see note below)</td>
      </tr>
  </tbody>
</table>
<p><strong>Basic Auth Tests</strong></p>
<table>
  <thead>
      <tr>
          <th style="text-align: left;">Test</th>
          <th style="text-align: left;">Result</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td style="text-align: left;">Request without credentials</td>
          <td style="text-align: left;">401 Unauthorized</td>
      </tr>
      <tr>
          <td style="text-align: left;">Request with valid credentials (admin:testpass123)</td>
          <td style="text-align: left;">200 OK</td>
      </tr>
  </tbody>
</table>
<h2>Conclusion<span class="hx-absolute -hx-mt-20" id="conclusion"></span>
    <a class="subheading-anchor" href="#conclusion"></a></h2><p>As with all technology stacks, new pieces of the puzzle get released and sometimes replace existing implementations. Because of the maintenance needs around Ingress NGINX—it was hard to troubleshoot, difficult to standardize across projects, and only a handful of people were working on it—the new standard, Kubernetes Gateway API, was released with the goal of being an agnostic CRD to work with any vendor and any gateway. The concern with that was there was no clear migration step as thousands of engineers were using Ingress NGINX, so a tool to move from that to the new Kubernetes Gateway API standard was necessary. The specifications in this tool (emitter design and initial implementation) have recently moved to the upstream ingress2gateway project as well. In this blog post, you learned how to perform the migration on an object using the Ingress NGINX Controller to the new Gateway API standard.</p>
<h2>Keep in Touch<span class="hx-absolute -hx-mt-20" id="keep-in-touch"></span>
    <a class="subheading-anchor" href="#keep-in-touch"></a></h2><p>If you have any questions or just want to learn more about kgateway, feel free to reach out to us on <a href="https://cloud-native.slack.com/archives/C080D3PJMS4" rel="noopener" target="_blank">in the <code>#kgateway</code> channel in the CNCF Slack</a> or <a href="https://github.com/kgateway-dev/community/blob/main/README.md#community-meetings" rel="noopener" target="_blank">our community meetings</a>! There is a vibrant, open-source community in both places with people eager to chat and learn.</p>
