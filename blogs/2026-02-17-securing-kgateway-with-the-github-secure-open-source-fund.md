---
title: "Securing kgateway with the GitHub Secure Open Source Fund"
url: "/blog/github-secure-open-source/"
date: "Tue, 17 Feb 2026 14:35:00 +0000"
author: ""
feed_url: "https://kgateway.dev/blog/index.xml"
---
<p>Earlier this year, <a href="https://github.com/kgateway-dev/kgateway" rel="noopener" target="_blank">kgateway</a> was selected to participate in the <a href="https://github.com/open-source/github-secure-open-source-fund" rel="noopener" target="_blank">GitHub Secure Open Source Fund</a>, an initiative that provides maintainers with a three-week education sprint focused on the latest tooling, best practices, and strategies for securing open source software projects.</p>
<p>Being part of this program was both an honor and an opportunity. The GitHub Secure Open Source Fund provided expert insights, structured guidance, and a collaborative environment where we learned alongside other open source maintainers who share the same commitment to strengthening the security of the ecosystem.</p>
<h2>Building on a Strong Foundation<span class="hx-absolute -hx-mt-20" id="building-on-a-strong-foundation"></span>
    <a class="subheading-anchor" href="#building-on-a-strong-foundation"></a></h2><p>Kgateway is a Kubernetes-native ingress controller and next-generation API gateway that builds on top of the Envoy proxy and implements the Kubernetes Gateway API. As such, security has always been a priority for kgateway. As part of the CNCF ecosystem, we emphasize licensing clarity, community governance, and responsible disclosure practices.</p>
<p>One key takeaway from the program was that security isn’t just about detecting issues; it’s about building processes that make fixing them routine and predictable.</p>
<p>Before the program began, we had already put important foundations in place, especially to handle an influx of AI-generated contributions:</p>
<ul>
<li>Community standards, including an AI-generated code policy section.</li>
<li>GitHub Copilot running automated reviews on pull requests.</li>
</ul>
<p>However, the Secure Open Source Fund helped us formalize, automate, and strengthen our approach in meaningful ways.</p>
<h2>What We Improved<span class="hx-absolute -hx-mt-20" id="what-we-improved"></span>
    <a class="subheading-anchor" href="#what-we-improved"></a></h2><p>The kgateway team improved security processes related to vulnerability reporting, code scanning, and repository hygiene.</p>
<h3>🔐 Formalized Vulnerability Reporting<span class="hx-absolute -hx-mt-20" id="-formalized-vulnerability-reporting"></span>
    <a class="subheading-anchor" href="#-formalized-vulnerability-reporting"></a></h3><p>We created a <a href="https://github.com/kgateway-dev/kgateway/blob/main/SECURITY.md" rel="noopener" target="_blank"><code>SECURITY.md</code> file</a> that clearly documents how to report vulnerabilities and what contributors can expect from our disclosure process. Alongside this, we refined our security incident response documentation to ensure we have a well-defined and actionable response plan. For more information, see the <a href="https://kgateway.dev/docs/envoy/latest/reference/vulnerabilities/" rel="noopener" target="_blank">docs</a>.</p>
<h3>🔎 Enabled gosec as a Required Check<span class="hx-absolute -hx-mt-20" id="-enabled-gosec-as-a-required-check"></span>
    <a class="subheading-anchor" href="#-enabled-gosec-as-a-required-check"></a></h3><p>We activated <a href="https://github.com/securego/gosec" rel="noopener" target="_blank"><code>gosec</code></a> static analysis scanning and made it a required check on every pull request. While enabling it, we:</p>
<ul>
<li>Fixed type conversion issues</li>
<li>Addressed file permission concerns</li>
<li>Cleaned up findings surfaced by the scanner</li>
</ul>
<p>For a project extending the Gateway API with Kubernetes custom resources using Kubebuilder, kgateway introduces new types like TrafficPolicy and GatewayParameters. Catching unsafe type conversions, invalid references, or misconfigured RBAC early is critical. Static analysis with gosec provides an additional layer of guardrails, complementing Kubebuilder’s CRD validation, before changes ever reach production clusters.</p>
<p>When <code>gosec</code> fails locally or in CI on a pull request, it prints the specific rule ID, affected file, and line number, and a brief explanation of the issue, so contributors can quickly identify and remediate the finding.</p>
<p>This not only improved the codebase immediately but also ensured future contributions meet a higher security bar.</p>
<h3>🧹 Cleaned Up Repository Secrets<span class="hx-absolute -hx-mt-20" id="-cleaned-up-repository-secrets"></span>
    <a class="subheading-anchor" href="#-cleaned-up-repository-secrets"></a></h3><p>We audited and removed unused secrets in our repository environments, reducing risk and improving overall repository hygiene.</p>
<h2>Why This Matters<span class="hx-absolute -hx-mt-20" id="why-this-matters"></span>
    <a class="subheading-anchor" href="#why-this-matters"></a></h2><p>The GitHub Secure Open Source Fund does more than support individual projects; it strengthens the entire open-source ecosystem by investing in its security foundation.</p>
<p>For kgateway, this experience helped us:</p>
<ul>
<li>Formalize and document our security processes.</li>
<li>Automate enforcement of secure coding practices.</li>
<li>Connect with other maintainers facing similar security challenges.</li>
<li>Engage our community more transparently around security.</li>
</ul>
<p>Security is not a one-time milestone, but rather an ongoing commitment. This program accelerated our progress and reinforced our dedication to building kgateway as a secure, reliable project for the community. Because many of our maintainers are also involved in other open source projects such as <a href="https://github.com/agentgateway/agentgateway" rel="noopener" target="_blank">agentgateway</a>, we are also applying the lessons learned from this security initiative to other projects.</p>
<p>If you maintain an open source project, start small: add a <code>SECURITY.md</code> file, enable dependency scanning, and audit unused secrets. Small steps compound quickly.</p>
<h2>Acknowledgements<span class="hx-absolute -hx-mt-20" id="acknowledgements"></span>
    <a class="subheading-anchor" href="#acknowledgements"></a></h2><p>We are immensely grateful to GitHub and everyone involved in the Secure Open Source Fund for their support and expertise.</p>
<p>We want to reiterate our thanks to the following partners who supported the Secure Open Source Fund.</p>
<p>Funding Partners: Alfred P. Sloan Foundation, American Express, Chainguard, Datadog, Herodevs, Kraken, Mayfield, Microsoft, Shopify, Stripe, Superbloom, Vercel, Zerodha, 1Password</p>
<p>Ecosystem Partners: Atlantic Council, Ecosyste.ms, CURIOSS, Digital Data Design Institute Lab for Innovation Science, Digital Infrastructure Insights Fund, Microsoft for Startups, Mozilla, OpenForum Europe, Open Source Collective, OpenUK, Open Technology Fund, OpenSSF, Open Source Initiative, OpenJS Foundation, University of California, OWASP, Santa Cruz OSPO, Sovereign Tech Agency, SustainOSS</p>
<p>And as always, thanks to our kgateway community! If you haven&rsquo;t already, come join us in the <code>#kgateway</code> channel on the CNCF Slack and share your security best practices.</p>
<div class="hextra-cards hx-mt-4 hx-gap-4 hx-grid not-prose">
<a class="hextra-card hx-group hx-flex hx-flex-col hx-justify-start hx-overflow-hidden hx-rounded-lg hx-border hx-border-gray-200 hx-text-current hx-no-underline dark:hx-shadow-none hover:hx-shadow-gray-100 dark:hover:hx-shadow-none hx-shadow-gray-100 active:hx-shadow-sm active:hx-shadow-gray-200 hx-transition-all hx-duration-200 hover:hx-border-gray-300 hx-bg-transparent hx-shadow-sm dark:hx-border-neutral-800 hover:hx-bg-slate-50 hover:hx-shadow-md dark:hover:hx-border-neutral-700 dark:hover:hx-bg-neutral-900" href="https://github.blog/open-source/maintainers/securing-the-ai-software-supply-chain-security-results-across-67-open-source-projects/" rel="noreferrer" target="_blank"><span class="hextra-card-icon hx-flex hx-font-semibold hx-items-start hx-gap-2 hx-pt-4 hx-px-4 hx-text-gray-700 hover:hx-text-gray-900 dark:hx-text-neutral-200 dark:hover:hx-text-neutral-50"><svg fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" stroke-linecap="round" stroke-linejoin="round"></svg>GitHub blog post</span><div class="hextra-card-subtitle hx-line-clamp-3 hx-text-sm hx-font-normal hx-text-gray-500 dark:hx-text-gray-400 hx-px-4 hx-mb-4 hx-mt-2">Learn more about the Secure Open Source Fund projects</div></a>
<a class="hextra-card hx-group hx-flex hx-flex-col hx-justify-start hx-overflow-hidden hx-rounded-lg hx-border hx-border-gray-200 hx-text-current hx-no-underline dark:hx-shadow-none hover:hx-shadow-gray-100 dark:hover:hx-shadow-none hx-shadow-gray-100 active:hx-shadow-sm active:hx-shadow-gray-200 hx-transition-all hx-duration-200 hover:hx-border-gray-300 hx-bg-transparent hx-shadow-sm dark:hx-border-neutral-800 hover:hx-bg-slate-50 hover:hx-shadow-md dark:hover:hx-border-neutral-700 dark:hover:hx-bg-neutral-900" href="https://slack.cncf.io/" rel="noreferrer" target="_blank"><span class="hextra-card-icon hx-flex hx-font-semibold hx-items-start hx-gap-2 hx-p-4 hx-text-gray-700 hover:hx-text-gray-900 dark:hx-text-neutral-200 dark:hover:hx-text-neutral-50"><svg fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" stroke-linecap="round" stroke-linejoin="round"></svg>CNCF Slack channel</span></a>
</div>
