---
layout: post
title: Load Balancing Public and Private Traffic to Azure API Management across Multiple Regions
date: 2020-07-08 10:00
author: Paco de la Cruz
comments: true
category: Azure API Management
tags: [API Management, Load Balancing, Azure Networking]
---

<base target="_blank"/>

<p><img src="/assets/img/2020/07/00-green-red-light.jpg" alt="Green red light" width="636" style="width: 636px;"></p>
<p>In different projects, I have had to implement load balancing for <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region" rel="noopener" target="_blank">multi-region</a> deployments of Azure API Management. API Management with multi-region deployments, allows you to enable a built-in external load balancer. This means that public traffic is routed to a <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region#-use-custom-routing-to-api-management-regional-gateways" rel="noopener" target="_blank">regional gateway based on the lowest latency</a> without the need for additional configuration or the help of any other service.</p>
<!--more-->
<p>However, it is quite common that organisations want to use API Management for both internal and external consumers, exposing just a subset of the APIs to the public internet. <a href="https://docs.microsoft.com/en-us/azure/application-gateway/overview" rel="noopener" target="_blank">Azure Application Gateway</a> enables those scenarios. Additionally, it provides a <a href="https://docs.microsoft.com/en-us/azure/web-application-firewall/ag/ag-overview" rel="noopener" target="_blank">Web Application Firewall to protect APIs from malicious attacks</a>. When organisations integrate API Management with Application Gateway, the built-in external load balancer can no longer be used.</p>
<p>The simplest way to implement load balancing across multiple regions for public traffic to API Management exposed via Application Gateway is to use <a href="https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview">Azure Traffic Manager</a>. However, what to do when private traffic, for instance, coming from on-premises, must also be load-balanced to the multiple regions of API Management?&nbsp; According to this <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-using-with-vnet#--routing" rel="noopener" target="_blank">document</a>:</p>
<blockquote>
<p>For multi-region API Management deployments configured in internal virtual network mode, users are responsible for managing the load balancing across multiple regions, as they own the routing.</p>
</blockquote>
<p>At the time of writing, Azure does not have a highly available service offering for load-balancing private HTTP traffic across regions with health probing capabilities. There is a <a href="https://feedback.azure.com/forums/217313-networking/suggestions/33924592-traffic-manager-allow-internal-routing-option" rel="noopener" target="_blank">feature request</a> in the networking feedback forums that raises this gap. Thus, currently, there is no simple approach to implement this. Some months back, I <a href="https://github.com/MicrosoftDocs/azure-docs/issues/39015" rel="noopener" target="_blank">raised an issue</a> on <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region" rel="noopener" target="_blank">this documentation page</a> due to the lack of clarity on the topic.</p>
<p>In this post, I will explain how we can implement a highly available load balancing for both public and private HTTP Traffic to Azure API Management when integrated with Application Gateway and deployed on multiple regions. The approach I show here relies on having public health-probe endpoints exposed via Application Gateway.</p>
<h1>Discarded Approaches</h1>
<p>Before I explain how this could be implemented, I would like to cover the approaches that were discarded and the rationale behind that. Even though they are certainly not feasible, it might be worth reviewing them here as these alternatives could be raised as part of a technical decision process. This section could potentially help you to save some time and avoid exploring alternatives with a dead-end.</p>
<p>There are some load balancing service offerings on Azure that target internal traffic. However, they could not be considered for the reasons outlined below.</p>
<ul>
<li><a href="https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview" rel="noopener" target="_blank">Azure Load Balancer</a>– It operates at layer 4 (Transport Layer) of the&nbsp;OSI Network Stack and supports TCP and UDP protocols.&nbsp;However, it targets Virtual Machines and Scale Sets only. It cannot be used for PaaS services like API Management. Furthermore, it targets load balancing of regional traffic and is deployed within a single region, which means a single point of failure.</li>
<li><a href="https://docs.microsoft.com/en-us/azure/application-gateway/overview" rel="noopener" target="_blank">Azure Application Gateway</a> – It is a layer 7 (Application Layer) web traffic load balancer that can make routing decisions based on HTTP request attributes. Application Gateway supports any public and private IP address as back ends, including Azure API Management or Azure Web Apps. However, it is deployed within a single region. Thus, by itself is not resilient to a regional outage or disaster.</li>
</ul>
<p>If you want to read more about these options and how they compare to Traffic Manager, refer to <a href="https://devblogs.microsoft.com/premier-developer/azure-load-balancing-solutions-a-guide-to-help-you-choose-the-correct-option/" rel="noopener" target="_blank">this article</a>.</p>
<p>Another option that was discarded was using the Windows DNS service. While this service can be geo-distributed and supports <a href="https://docs.microsoft.com/en-us/windows-server/networking/dns/deploy/app-lb" rel="noopener" target="_blank">Application Load-Balancing</a> with round-robin or basic weighted routing, it does not have health probing capabilities, so traffic could potentially be directed to unavailable or unhealthy endpoints.</p>
<p>Microsoft has documented two approaches for using Azure Traffic Manager to failover private endpoints on Azure with some limitations as summarised below.</p>
<ul>
<li><a href="https://docs.microsoft.com/en-us/archive/blogs/mihansen/using-azure-traffic-manager-for-private-endpoint-failover-manual-method" rel="noopener" target="_blank">Using Azure Traffic Manager for Private Endpoint Failover - Manual Method</a> - This approach works well when only one region is meant to be active at a time (active-passive scenarios) and a manual failover is acceptable. However, it does not work well on active-active scenarios. If you are paying for a multi-instance deployment of API Management and/or receiving traffic from multiple regions, you most likely want to be able to leverage all the instances running.</li>
<li><a href="https://docs.microsoft.com/en-us/archive/blogs/mihansen/using-azure-traffic-manager-for-private-endpoint-failover-automation" rel="noopener" target="_blank">Using Azure Traffic Manager for Private Endpoint Failover – Automation</a> - This approach could work for active-active scenarios. However, its main drawback is that it requires a custom monitoring solution. Then, you would need to think of making sure that this monitoring component is highly available and resilient to a regional outage or disaster.</li>
</ul>
<p>Now, let us continue to discuss how to implement a highly available load balancing of public and private traffic to API Management across multiple regions.</p>
<h1>Prerequisites</h1>
<p>If you are planning to implement this in your organisation, there are some prerequisites that you need to bear in mind.</p>
<ul>
<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region" rel="noopener" target="_blank">Multi-region deployment of <strong>API Management</strong></a><span><strong>.</strong></span> This is somehow obvious, assuming this is the reason you are reading this article.</li>
<li><a href="https://docs.microsoft.com/en-us/azure/application-gateway/overview" rel="noopener" target="_blank"><strong>Application Gateway</strong></a> in both regions. As I mentioned before, this approach relies on having health probe endpoints available via the public internet. Application Gateway is the one in charge of routing and exposing some of the private endpoints into the internet while keeping certain APIs only accessible via the private network.</li>
<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway" rel="noopener" target="_blank"><strong>API Management with Application Gateway integration</strong></a> set up. You can find more information on <a href="https://techcommunity.microsoft.com/t5/azure-paas-developer-blog/integrating-api-management-with-app-gateway-v2/ba-p/1241650" rel="noopener" target="_blank">this article</a>.</li>
<li><a href="https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview" rel="noopener" target="_blank"><strong>Traffic Manager</strong></a> to implement the load balancing and health probing.</li>
<li>Write access to the <strong>private DNS Zone </strong>to create private DNS records.</li>
<li>Write access to the <strong>public DNS registrar </strong>to create public DNS records.</li>
</ul>
<h1>Relevant Concepts</h1>
<p>Before we get into the technical details of how this approach works, I recommend you make sure that you understand the concepts below.</p>
<ul>
<li><a href="https://en.wikipedia.org/wiki/Domain_Name_System" rel="noopener" target="_blank">DNS</a> – You don’t need to be aware of all the intricacies of how DNS resolution translates a <a href="https://en.wikipedia.org/wiki/Hostname" rel="noopener" target="_blank">hostname</a> into an IP address, but you do need to understand the basic principles of a private DNS zone and a public DNS service and registrar.</li>
<li><a href="https://en.wikipedia.org/wiki/Split-horizon_DNS" rel="noopener" target="_blank">Split-horizon DNS</a> - What you need to be aware of is that the same hostname can be configured to have different IP addresses in the public DNS registrar and in the private DNS zone. This way, private traffic gets directed to a private endpoint, while public traffic to a public endpoint.</li>
</ul>
<h1>Scenario</h1>
<p>In this hypothetical scenario, we want HTTP requests to the domain <code>api.pacodelacruz.io</code> to be directed to one of the available and healthy API Management endpoints. API Management is deployed in two regions and has an internal load balancer (ILB). Application Gateway is used to expose a subset of APIs to the public.</p>
<h1>Custom Public Traffic Load Balancing</h1>
<p>As mentioned above, a <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region" rel="noopener" target="_blank">multi-region</a> deployment of API Management premium provides a built-in external load balancer for public HTTP Traffic that routes requests to a <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region#-use-custom-routing-to-api-management-regional-gateways" rel="noopener" target="_blank">regional gateway based on the lowest latency</a>. However, you can implement custom routing and load balancing <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region#-use-custom-routing-to-api-management-regional-gateways" rel="noopener" target="_blank">using Traffic Manager</a>. Custom traffic routing is required when API Management is integrated with Application Gateway.</p>
<p>The conceptual architecture diagram below depicts the public traffic load-balancing scenario.</p>
<p><img src="/assets/img/2020/07/11-public-arch-1.png" width="636" style="width: 636px;" alt="Public Arch"></p>
<p>The flow of the public HTTP traffic would be as described in the sequence diagram shown below.</p>
<p><img src="/assets/img/2020/07/12-public-flow-1.png" width="594" style="width: 594px;" alt="Public Flow"></p>
<p>The public client’s flow can be summarised as follows:</p>
<ul>
<li>The public client is trying to reach the hostname, e.g. <code>api.pacodelacruz.io</code></li>
<li>The client checks whether the hostname mapping to an IP address is already in its cache. Otherwise, it goes to the public DNS to resolve it.</li>
<li>The public DNS record points to the Traffic Manager endpoint via a hostname.</li>
<li>Traffic Manager then routes the traffic based on the routing method configured on the profile and the availability, performance, and health of the endpoints. It then returns a hostname, in this scenario either <code>api1.pacodelacruz.io</code> or <code>api2.pacodelacruz.io</code></li>
<li>Then the public DNS, in turn, resolves the returned hostname to the regional Application Gateway’s hostname and then to its public IP address.</li>
<li>The regional Application Gateway’s IP address is returned to the client and now the public client can make the request to the corresponding API Management instance through App Gateway.</li>
</ul>
<p>For the sequence described above to work, we need the configuration below:</p>
<p><strong>Public DNS</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="170">
<p><strong>Hostname</strong></p>
</td>
<td width="431">
<p><strong>Pointing to</strong></p>
</td>
</tr>
<tr>
<td width="170">
<p><code>api.pacodelacruz.io</code></p>
</td>
<td width="431">
<p>Azure Traffic Manager’s hostname, e.g. <code>apipacodelacruzio.trafficmanager.net</code></p>
</td>
</tr>
<tr>
<td width="170">
<p><code>api1.pacodelacruz.io</code></p>
</td>
<td width="431">
<p>FQDN of the Application Gateway in the primary region, e.g. <code>pacodelacruzio-appgw-ase.australiasoutheast.cloudapp.azure.com</code></p>
</td>
</tr>
<tr>
<td width="170">
<p><code>api2.pacodelacruz.io</code></p>
</td>
<td width="431">
<p>FQDN of the Application Gateway in the secondary region, e.g. <code>pacodelacruzio-appgw-aes.australiaeast.cloudapp.azure.com</code></p>
</td>
</tr>
</tbody>
</table>
<p><strong>Traffic Manager Profile</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="170">
<p><strong>Setting</strong></p>
</td>
<td width="431">
<p><strong>Value</strong></p>
</td>
</tr>
<tr>
<td width="170">
<p>Endpoint types</p>
</td>
<td width="431">
<p>external endpoint</p>
</td>
</tr>
<tr>
<td width="170">
<p>Endpoint 1</p>
</td>
<td width="431">
<p><code>api1.pacodelacruz.io</code></p>
</td>
</tr>
<tr>
<td width="170">
<p>Endpoint 2</p>
</td>
<td width="431">
<p><code>api2.pacodelacruz.io</code></p>
</td>
</tr>
<tr>
<td>
<p>Endpoint monitor settings / Custom Header settings</p>
</td>
<td>
<p><code>Host:api.pacodelacruz.io</code></p>
</td>
</tr>
</tbody>
</table>
<p>You might be wondering why we need <code>api1.pacodelacruz.io</code> and <code>api2.pacodelacruz.io</code> hostnames when we can route traffic from Traffic Manager directly to the Application Gateway hostnames. This will become evident when we see the configuration required for private traffic.</p>
<p>The Traffic Manager's endpoint monitor settings / custom header settings allows keeping the original hostname (<code>api.pacodelacruz.io</code>) in the health probe requests while also using the existing TLS certificate without having to add unnecessary aliases.&nbsp;</p>
<p>Details on how to integrate App Gateway with an API Management deployed within a VNET are detailed in this <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway" rel="noopener" target="_blank">article</a>. App Gateway needs to use the API Management regional endpoints for the back end pool and health probing endpoints as described <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region#-use-custom-routing-to-api-management-regional-gateways" rel="noopener" target="_blank">here</a>. API Management, in turn, will need to route API calls to regional backend services as defined <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-deploy-multi-region#-route-api-calls-to-regional-backend-services" rel="noopener" target="_blank">here</a>.</p>
<h1>Private Traffic Load Balancing</h1>
<p>Once we have the custom load balancing of public traffic set up via Traffic Manager, we can proceed to configure the private traffic load balancing. A conceptual architecture diagram is depicted in the figure below.</p>
<p><img src="/assets/img/2020/07/21-private-arch.png" width="861" style="width: 861px;" alt="Private Arch"></p>
<p>The flow of the private HTTP traffic is as depicted in the sequence diagram shown below.</p>
<p><img src="/assets/img/2020/07/22-private-flow.png" width="831" style="width: 831px;" alt="Private Flow"></p>
<p>The private client’s flow can be summarised as follows:</p>
<ul>
<li>The private client is trying to reach the hostname, e.g. <code>api.pacodelacruz.io</code></li>
<li>The client checks whether the hostname mapping to an IP address is already in its cache. Otherwise, it goes to the private DNS to resolve it.</li>
<li>The private DNS might have it in cache or can query the public DNS.</li>
<li>The public DNS record points to the Traffic Manager endpoint via a hostname.</li>
<li>Traffic Manager then routes the traffic based on the routing method configured on the profile and the availability, performance, and health of the endpoints. It then returns a hostname, in this scenario either <code>api1.pacodelacruz.io</code> or <code>api2.pacodelacruz.io</code></li>
<li>Then the private DNS, in turn, resolves the returned hostname to the regional API Management IP address. This is where the <a href="https://en.wikipedia.org/wiki/Split-horizon_DNS" rel="noopener" target="_blank">DNS split-horizon</a> is configured.</li>
<li>Now the private client can make the request to the corresponding API Management instance directly via the private network.</li>
</ul>
<p>For the sequence described above to work, we need the configuration below:</p>
<p><strong>Private DNS</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="170">
<p><strong>Hostname</strong></p>
</td>
<td width="431">
<p><strong>Pointing to</strong></p>
</td>
</tr>
<tr>
<td width="170">
<p><code>api1.pacodelacruz.io</code></p>
</td>
<td width="431">
<p>Internal IP address of the primary regional API Management instance</p>
</td>
</tr>
<tr>
<td width="170">
<p><code>api2.pacodelacruz.io</code></p>
</td>
<td width="431">
<p>Internal IP address of the secondary regional API Management instance</p>
</td>
</tr>
</tbody>
</table>
<p>All the remaining setup was done when configuring the custom public traffic load balancing as described above.</p>
<h2>Considerations</h2>
<p>As mentioned earlier, this approach for load-balancing private traffic only works if you have public health probe endpoints. When this is not the case, i.e. you have a multi-region deployment of Azure API Management with ILB only and this is not exposed to the public internet via Application Gateway, a different approach or workaround would be required.</p>
<p>One simple option is that you implement two sets of Azure Functions to expose the health status of internal endpoints to the public internet.&nbsp;</p>
<ul>
<li><strong>Internal probing Azure Functions</strong> - Timer triggered Azure Functions deployed within the VNET, one per region, whose purpose would be to check the health of the private APIs. These functions can then store the health status of each of the regional endpoints in a storage account.</li>
<li><strong>External health status endpoints – </strong>HTTP triggered Azure Functions deployed outside the VNET, one per region. These functions would read the health status of each internal endpoint from the storage account and return a corresponding HTTP status code. E.g. HTTP 200 for a healthy endpoint and HTTP 500 for an unhealthy endpoint.</li>
</ul>
<p>This way, the Traffic Manager could rely on the publicly exposed health probe endpoints without exposing internal APIs.</p>
<h1>Wrapping Up</h1>
<p><img src="/assets/img/2020/07/31-all-arch.png" alt="All Arch" width="1031" style="width: 1031px;"></p>
<p>Throughout this post, I have described how you can implement load balancing of public and private traffic to an API Management deployed to multiple regions and exposed via Application Gateway. As you have seen, once you have API Management integrated with Application Gateway, implementing load-balancing for both public and private traffic, you just have to add a Traffic Manager profile and add certain records on both the public DNS registrar and the private DNS zone.</p>
<p>If you want to be able to load-balance private traffic only without exposing health probe endpoints to the public internet, consider casting your vote to this <a href="https://feedback.azure.com/forums/217313-networking/suggestions/33924592-traffic-manager-allow-internal-routing-option" rel="noopener" target="_blank">feature request</a>, so that this is provided as a built-in networking feature on Azure.</p>
<p>I hope you have found this post useful!</p>
<p>Happy load balancing!</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>