---
layout: post
title: When to Use an Azure App Service Environment v2 (App Service Isolated)
date: 2017-08-08 23:05
author: Paco de la Cruz
comments: true
category: Architecture
tags: [App Service, App Service Isolated, Architecture, Azure, Azure App Service Environment]
---
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/08/080817_1153_WhentoUsean1.png" alt="" /></p>

<h2>Introduction</h2>
The <a href="https://docs.microsoft.com/en-us/azure/app-service/app-service-environment/intro">Azure App Service Environment</a> (ASE) is a premium feature offering of the Azure App Services which is fully isolated, highly scalable, and runs on a customer's virtual network. On an ASE you can host Web Apps, API Apps, Mobile Apps and Azure Functions. The first generation of the App Service Environment (ASE v1) was released in late 2015. Two significant updates were launched after that; in July 2016, they added the option of having an <a href="https://docs.microsoft.com/en-us/azure/app-service-web/app-service-environment-with-internal-load-balancer">Internal Load Balancer</a>, and in August of that year, an <a href="https://docs.microsoft.com/en-us/azure/app-service-web/app-service-app-service-environment-web-application-firewall">Application Gateway with an Web Application Firewall</a> could be configured for the ASE. After all the feedback Microsoft had been receiving on this offering, they started working on the second generation of the App Service Environment (ASE v2); and on July of 2017, it's been <a href="https://azure.microsoft.com/en-gb/blog/announcing-app-service-isolated-more-power-scale-and-ease-of-use/">released as Generally Available</a>.

In my previous job, I wrote the post "<a href="https://blog.kloud.com.au/2016/04/05/when-to-use-an-azure-app-service-environment/">When to Use an App Service Environment</a>", which referred to the first generation (ASE v1). I've decided to write an updated version of that post, mainly because that one has been one of my posts with more comments and questions and I know the App Service Environment, also called App Service Isolated, will continue to grow in popularity. Even though the ASE v2 has been simplified, I still believe many people would have questions about it or would want to make sure that they have no other option but to pay for this premium feature offering when they have certain needs while deploying their solutions on the Azure PaaS.

When you are planning to deploy Azure App Services, you have the option of creating them on a multi-tenant environment or on your own isolated (single-tenant) App Service Environment. If you want to understand in detail what they mean by "multi-tenant environment" for Azure App Services, I recommend you to read <a href="https://msdn.microsoft.com/en-us/magazine/mt793270.aspx">this article</a>. When they refer to a "Scale-Unit" in that article, they are talking about this multi-tenant shared infrastructure. You could picture an App Service Environment having a very similar architecture, but with all the building blocks dedicated to you, including the Front-End, File Servers, API Controllers, Publisher, Data Roles, Database, Web Workers, etc.

In this post, I will try to summarise when is required to use an App Service Environment (v2), and in case you have an App Service Environment v1, why it makes a lot of sense to migrate it to the second generation.
<h2>App Service Environment v2 Pricing</h2>
Before getting too excited about the functionality and benefits of the ASE v2 or App Service Isolated, it's important to understand its pricing model.

Even though, they have abstracted much of the complexity of the ASE in the second generation, we still need to be familiar with the architecture of this feature offering to properly calculate the costs of the App Service Environment v2. To calculate the total cost of your ASE, you need to consider an <strong>App Service Environment Base Fee </strong>and the cost of the<strong> Isolated Workers</strong>.

The <strong>App Service Environment Base Fee</strong> covers the cost of the of all the infrastructure required to run your single-tenant and isolated Azure App Services; including load balancing, high-availability, publishing, continuous delivery, app settings shared across all instances, deployment slots, management APIs, etc. None of your assemblies or code are executed in the instances which are part of this layer. Then, the <strong>Isolated Workers</strong> are the ones executing your Web Apps, API Apps, Mobile Apps or Functions. You decide the size and how many Isolated workers you want to spin up, thus the cost of the worker layer. Both layers are charged by the hour. Below, the prices for the Australian Regions in Australian Dollars are shown.

<img class="alignnone wp-image-1174" src="https://www.mexia.com.au/wp-content/uploads/2017/08/ASE-Pricing-Australia-Highlighted.png" alt="" width="701" height="473" />

In Australia, the App Service Environment base fee is above $ 1700 AUD per month, and the Isolated I1 instance is close to $ 500 AUD per month. This means that the entry-level of an ASE v2 with one Isolated Worker costs around $ 2,200 AUD per month or above $ 26,000 AUD per year, which is very similar to the price of the ASE v1 in this region. This cost can easily escalate by scaling up or scaling out the ASE. It's noteworthy that prices vary from region to region. For instance, according to the <a href="https://azure.microsoft.com/en-us/pricing/calculator/">Azure pricing calculator</a>, at the time of writing, the <strong>prices for the Australian Regions are around 35% more expensive than those in the West US 2</strong>. To calculate your own costs, in your region and in your currency, check the <a href="https://azure.microsoft.com/en-us/pricing/calculator/">pricing calculator</a>.

Moreover, the <strong>App Service Environment Base Fee</strong> is calculated based on the default configuration, which uses I1 instances for the ASE Front-End, and with the scaling rule of adding one Front End instance for every 15 worker instances, as described in the Front End scale configuration page shown below. If you keep this configuration, the App Service Environment Base Fee will stay the same, regardless of the number and size of workers. However, we can scale-up the Front End instances to I2 or I3 or reduce the number of workers per Front End instance. This would have an impact on the App Service Environment Base Fee. To calculate the extra cost, you would need to add the cost of every additional core on top of the base configuration. Before changing the Front-End scaling configuration, bear in mind that the Front End instances act only as a layer seven-load balancer (round robin) and perform SSL termination. All the compute of your App Services is executed in the worker instances.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/08/080817_1153_WhentoUsean3.png" alt="" />

With these price tag, the value and benefits of the ASE must be clear enough so that we can justify the investment to the business.
<h2>The benefits of the Azure App Service Isolated or App Service Environment v2.</h2>
To understand the benefits and advance features of an App Service Environment v2, it's worth comparing what we get with this premium offering with what we get by deploying an Azure App Service on a multi-tenant environment. This comparison is shown in the table below.
<table style="border-collapse:collapse;" border="0"><colgroup> <col style="width:415px;" /> <col style="width:416px;" /> <col style="width:416px;" /></colgroup>
<tbody valign="top">
<tr>
<td style="padding-left:14px;padding-right:14px;border:solid .5pt;"></td>
<td style="padding-left:14px;padding-right:14px;border-top:solid .5pt;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Multi-tenant environment</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:solid .5pt;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>App Service Isolated /</strong>
<strong> App Service Environment v2</strong></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Virtual Network (VNET) Integration</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

Azure App Services can be <a href="https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-integrate-with-vnet">integrated to an Azure Virtual Network</a>.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

An ASE is always deployed in the customer's Virtual Network.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Static Inbound IP Address</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

By default, Azure App Services get assigned a virtual IP addresses. However, this is shared with other App Services in that region.

You can <a style="font-family:inherit;font-size:inherit;" href="https://docs.microsoft.com/en-us/azure/app-service-web/app-service-web-tutorial-custom-ssl">bind a IP-based SSL certificate</a><span style="font-family:inherit;font-size:inherit;"> to your App Service, which will give you a dedicated public inbound IP address.</span></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

ASEs provide a static virtual inbound IP address (VIP). This VIP can be public or private, depending on whether configured with and Internal Load Balancer (ILB) or not.

More information on the ASE network architecture <a href="https://docs.microsoft.com/en-us/azure/app-service-web/app-service-app-service-environment-network-architecture-overview">here</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Static Outbound IP Address</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">No.

The outbound IP address of an App Service is not static, but it can be <a href="https://blogs.msdn.microsoft.com/waws/2017/02/01/how-do-i-determine-the-outbound-ip-addresses-of-my-azure-app-service/">any address within a certain range, which is not static either</a>.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

ASEs provide a static public outbound IP address. More information <a href="https://docs.microsoft.com/en-us/azure/app-service-web/app-service-app-service-environment-network-architecture-overview">here</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Connecting to Resources On-Premises</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

<span style="font-family:inherit;font-size:inherit;">Azure App Service VNET integration provides the capability to access resources on-premises via a VPN over the public Internet.
</span>

Additionally, <a href="https://www.mexia.com.au/the-new-azure-hybrid-connections/">Azure Hybrid Connections</a> can be used to connect to resources on-premises without requiring major firewall or network configurations.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

In addition to VPN over the public internet and Hybrid Connections support, an ASE provides the ability to connect to resources on-premises via ExpressRoute, which provides a faster, more reliable and secure connectivity without going over the public Internet.

<strong>Note</strong>: ExpressRoute has its own <a href="https://azure.microsoft.com/en-au/pricing/details/expressroute/">pricing model</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Private access only</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">No.

App Services are always accessible via the public internet.

One way to restrict access to your App Service is using <a href="https://azure.microsoft.com/en-us/blog/ip-and-domain-restrictions-for-windows-azure-web-sites/">IP and Domain restrictions</a>, but the App Service is still reachable from the internet.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

An ASE can be deployed with an Internal Load Balancer, which will lock down your App Services to be accessible only from within your VNET or via ExpressRoute or Site-to-Site VPN.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Control over inbound and outbound traffic</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">No.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

An ASE is always deployed on a subnet within a VNET. Inbound and outbound traffic can be <a href="https://docs.microsoft.com/en-us/azure/app-service-web/app-service-app-service-environment-control-inbound-traffic">controlled using a network security group</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Web Application Firewall</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

Starting from mid-July 2017, Azure Application Gateway with Web Application Firewall <a href="https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-web-app-overview">supports App Services in a multi-tenant environment</a>. More info on how to configure it <a href="https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-web-app-powershell">here</a>.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Yes.

An <a href="https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-web-application-firewall-portal">Azure Application Gateway with Web Application Firewall</a> can be configured to protect App Services on an ASE by preventing SQL injections, session hijacks, cross-site scripting attacks, and other attacks.

<strong>Note</strong>: The Application Gateway with Web Application Firewall has its own <a href="https://azure.microsoft.com/en-au/pricing/details/application-gateway/">pricing model</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>SLA</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">99.95%

No SLA is provided for Free or Shared tiers.

App Services starting from the Basic tier provide an <a href="https://azure.microsoft.com/en-au/support/legal/sla/app-service/v1_4/">SLA of 99.95%</a>.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">99.95%

App Services deployed on an ASE provide an <a href="https://azure.microsoft.com/en-au/support/legal/sla/app-service/v1_4/">SLA of 99.95%</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Instance Sizes / Scale-Up</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Full range.

App Services can be deployed on almost the <a href="https://azure.microsoft.com/en-au/pricing/details/app-service/">full range of tiers</a> from Free to Premium v2.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">3 sizes.

Workers on an ASE v2 support <a href="https://azure.microsoft.com/en-au/pricing/details/app-service/">three sizes (Isolated)</a></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Scalability / Scale-Out</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Maximum instances:

Basic: 3

Standard: 10

Premium: 20</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">ASE v2 supports up to 100 Isolated Worker instances.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Deployment Time</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Very fast.

The deployment of New App Services on the multi-tenant environment is rather fast, usually less than 2 minutes.

This can vary.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Slower.

The deployment of a New App Service Environment can take between 60 and 90 minutes (Tested on the Australian Regions)

This can vary.

<span style="font-family:inherit;font-size:inherit;">This is important to consider, particularly in cold DR scenarios.</span></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Scaling out Time</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Very fast.

Scaling out an App Service usually takes less than 2 minutes.

This can vary.</td>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">Slower.

Scaling out in an App Service Environment can take between 30 and 40 minutes (Tested on the Australian Regions).

This can vary.

This is something to consider when configuring auto-scaling.</td>
</tr>
</tbody>
</table>
<h2>Reasons to migrate your ASE v1 to an ASE v2</h2>
If you already have an App Service Environment v1, there are many reasons to migrate to the second generation, including:
<ul>
	<li><strong>More horsepower</strong>: With the ASE v2, you get Dv2-based machines, with faster cores, SSD storage, and twice the memory per core when compared to the ASE v1. You are practically getting double performance per core.</li>
	<li><strong>No stand-by workers for fault-tolerance: </strong>To provide fault-tolerance, the ASE v1 requires you to have one stand-by worker for every 20 active workers on each worker pool. You have to pay for those stand-by workers. ASE v2 has abstracted that for you, and you don't need to pay for those.</li>
	<li><strong>Streamlined scaling</strong>: If you want to implement auto-scaling on an ASE v1, you have to manage scaling not only at the App Service Plan level, but at the Worker Pool level as well. For that, you have to use a <a href="https://docs.microsoft.com/en-us/azure/app-service/app-service-environment-auto-scale">complex inflation rate formula</a>, which requires you to have some instances to be waiting and ready for whenever an auto-scale condition kicks off. This has its own cost implications. ASEs v2 allow you to auto-scale your App Service Plan the same way you do it with your multi-tenant App Services, without the complexity of managing worker pools and without paying for waiting instances.</li>
	<li><strong>Cost saving</strong>: Because you are getting an upgraded performance, you should be able to host the same workloads using half as much in terms of cores. Additionally, you don't need to pay for fault-tolerance or auto-scaling stand-by workers.</li>
	<li><strong>Better experience and abstraction</strong>: Deployment and scaling of the ASE v2 is much simpler and friendlier than it was with the first generation.</li>
</ul>
<h2>Wrapping Up</h2>
So coming back to original the question, when to use an App Service Environment? When is it required and would make sense to pay the premium price of the App Service Environment?
<ul>
	<li>When we need to restrict the App Services to be accessible only from within the VNET or via Express Route or Site-to-Site VPN, OR</li>
	<li>When we require to control inbound and outbound traffic to and from our App Services OR</li>
	<li>When we need a connection between the App Services and resources on-premises via a secure and reliable channel (ExpressRoute) without going via the public Internet OR</li>
	<li>When we require much more processing power, i.e. scaling out to more than 20 instances OR</li>
	<li>When a static outbound IP Address for the App Service is required.</li>
</ul>
What else would you consider when deciding whether to use an App Service Environment for your workload or not? Feel free to post your comments or feedback below!

Happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
