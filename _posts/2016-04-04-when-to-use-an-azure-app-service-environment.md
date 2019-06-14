---
layout: post
title: When to use an Azure App Service Environment?
date: 2016-04-04 22:12
author: Paco de la Cruz
comments: true
categories: [App Service Environment, Azure]
---

<p style="text-align:center;"><em>Cross-posted on <a href="https://blog.kloud.com.au/author/pacodelacruzag/">Kloud's blog</a>.
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>

Introduction
============

An [Azure App Service Environment](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-intro/) (ASE) is a premium [Azure App Service](https://azure.microsoft.com/en-us/services/app-service/) hosting environment which is dedicated, fully isolated, and highly scalable. It clearly brings advanced features for hosting Azure App Services which might be required in different enterprise scenarios. But being this a premium service, it comes with a premium price tag. Due to its cost, a proper business case and justification are to be prepared before architecting a solution based on this interesting PaaS offering on Azure.

When planning to deploy Azure App Services, an organisation has the option of creating an [Azure Service Plan](https://azure.microsoft.com/en-gb/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/) and hosting them there. This might be good enough for most of the cases. However, when higher demands of scalability and security are present, a dedicated and fully isolated [App Service Environment](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-intro/) might be necessary.

Below, I will summarise the information required to make a decision regarding the need of using an App Service Environment for hosting App Services. Please, when reading this post, consider that facts and data provided are based on Microsoft documentation at the time of writing, which will eventually change.

App Service Environment Pricing.
================================

To calculate the cost of an App Service Environment, we have to consider its architecture. An Azure App Service Environment is composed of two layers of [dedicated compute resources](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-intro/) and a [reserved static IP](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-reserved-public-ip/). Additionally, it requires a [Virtual Network](https://azure.microsoft.com/en-us/services/virtual-network/). The Virtual Network is [free of charge](https://azure.microsoft.com/en-us/pricing/details/virtual-network/) and reserved IP Addresses carry a [nominal charge](https://azure.microsoft.com/en-us/pricing/details/ip-addresses/). So the cost is mostly related to the compute resources. The ASE is composed of one front-end compute resource pool, as well as one to three worker compute resource pools.

The minimum implementation of an App Service Environment requires 2 x Premium P2 instances for the Front-End Pool and 2 x Premium P1 instances for the Worker Pool 1, with a total cost per annum [superior to $ 20,000 AUD](https://azure.microsoft.com/en-us/pricing/details/app-service/). This cost can easily escalate by scaling up or scaling out the ASE.

Having said that, the value and benefits must be clear enough so that the business can justify this investment.

The benefits of an Azure App Service Environment.
=================================================

To understand the benefits and advance features of an App Service Environment, we can compare what we get by deploying our Azure App Services on or without an App Service Environment, as show in the table below.

|  | **Without an App Service Environment** | **On an App Service Environment** |
| --- |  --- |  --- |
| **Isolation Level** | Compute resources are hosted on a multitenant environment. | All [compute resources](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-intro/) are fully isolated and dedicated exclusively to a single subscription. |
| **Compute resources specialisation** | There is no out-of-the-box compute resource layer specialisation. | [Compute resources](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-intro/) on an ASE are grouped in 2 different layers: Front-End Pool and Worker Pools (up to 3).The Front-End Pool is in charge of SSL termination and load balancing of app requests for the corresponding Worker Pools. Once the SSL has been off-loaded and the load balanced, the Worker Pool is in charge of processing all the logic of the App Services. The Front-End Pool is shared by all Worker Pools. |
| **Virtual Network (VNET) Integration** | A [Virtual Network](https://azure.microsoft.com/documentation/articles/virtual-networks-faq/) can be created and App Services [can be integrated to it](https://azure.microsoft.com/en-us/documentation/articles/web-sites-integrate-with-vnet/).The Virtual Network provides full control over IP address blocks, DNS settings, security policies, and route tables within the network.Classic "v1" and Resource Manager "v2" Virtual Networks can be used. | An ASE is always deployed in a regional Virtual Network. This provides the ability to have access to resources in a VNET without any additional configuration required.*[UPDATE] *Starting from mid-July 2016, ASEs now support "v2" ARM based virtual networks. |
| ***[UPDATE July 2016]*** **Accessible only via Site-to-Site or ExpressRoute VPN** | App Services are accessible via public Internet.  | *[UPDATE July 2016] ASEs support an [Internal Load Balancer (ILB)](https://azure.microsoft.com/en-us/documentation/articles/app-service-environment-with-internal-load-balancer/) which allows you to host your intranet or LOB applications on Azure and access them only via a Site-to-Site or ExpressRoute VPN. *  |
| ** Control over inbound and outbound traffic** | Inbound and outbound traffic control is not currently supported. | An ASE is [always deployed in a regional Virtual Network](http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-control-inbound-traffic/), thus [inbound and outbound network traffic can be controlled](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-control-inbound-traffic/) using a [network security group](https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/).*[UPDATE] *With updates of mid-July 2016, now ASEs can be [deployed into VNETs which use private address ranges](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-control-inbound-traffic/). |
| **Connection to On-Prem Resources** | [Azure App Service Virtual Network integration](https://azure.microsoft.com/en-us/documentation/articles/web-sites-integrate-with-vnet/) provides the capability to access on-prem resources via a VPN over public Internet. | In addition to [Azure App Service Virtual Network integration](https://azure.microsoft.com/en-us/documentation/articles/web-sites-integrate-with-vnet/), the ASE provides [the ability to connect](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-network-configuration-expressroute/) to on-prem resources via [ExpressRoute](https://azure.microsoft.com/en-us/documentation/articles/expressroute-introduction/), which provides a faster and more reliable and secure connectivity without going over public Internet.**Note**: ExpressRoute has its own [pricing model](https://azure.microsoft.com/en-us/pricing/details/expressroute/). |
| **Inspecting inbound web traffic and blocking potential attacks** | *[UPDATE Sept -- 2016]* A Web Application Firewall (WAF) service is available to App Services through [Application Gateway](https://azure.microsoft.com/en-us/documentation/articles/application-gateway-webapplicationfirewall-overview/).Application Gateway WAF has its own pricing model. | ASEs provide the ability to configure a [Web Application Firewall](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-web-application-firewall/) for inspecting inbound web traffic which can block SQL injections, cross-site scripting, malware uploads, application DDoS, and other attacks.**Note**: Web Application Firewall has its own [pricing model](https://azure.microsoft.com/en-us/marketplace/partners/barracudanetworks/waf/). |
| **Static IP Address** | By default, Azure App Services get assigned virtual IP addresses. However, these are shared with other App Services in that region.There is a way to [give an Azure Web App a dedicated inbound static IP](https://blogs.msdn.microsoft.com/benjaminperkins/2014/05/05/how-to-get-a-static-ip-address-for-your-microsoft-azure-web-site/) address.Nevertheless, there is no way to get a dedicated static outbound IP. Thus, an Azure App Service outbound IP cannot be securely whitelisted on on-prem or third-party firewalls. | ASEs provides a [static Inbound and Outbound IP Address](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-network-architecture-overview/) for all resources contained within it.App Services (Web App, Azure Web Jobs, API Apps, Mobile Apps and Logic Apps) can connect to third party application using a dedicated static outbound IP which can be whitelisted on on-prem or third-party firewalls. |
| **SLA** | App Services provide an [SLA of 99.95%](https://azure.microsoft.com/en-us/support/legal/sla/app-service/). | App Services deployed on an ASE provide an [SLA of 99.95%](https://azure.microsoft.com/en-us/support/legal/sla/app-service/). |
| **Scalability / Scale-Up** | App Services can be deployed on almost the [full range of pricing tiers](https://azure.microsoft.com/en-us/pricing/details/app-service/) from Free to Premium.However, Premium P4 is not available for App Services without an ASE. | App Services deployed on an ASE can only be deployed on [Premium](https://azure.microsoft.com/en-us/pricing/details/app-service/) instances, including Premium 4\. (8 cores, 14 GB RAM, 500 GB Storage) |
| **Scalability / Scale-Out** | App Services provisioned on a Standard App Service Plan can Scale-Out with up to 10 instances.App Services provisioned on a Premium App Service Plan can Scale-Out with up to 20 instances. | App Services deployed on an ASE can [scale out with up to 50 instances](https://azure.microsoft.com/en-us/documentation/articles/app-service-web-configure-an-app-service-environment/).An ASE can be configured to use up to 55 total compute resources. Of those 55, only 50 can be used to host workloads. |
| **Scalability / Auto Scale-Out** | App Services can be scaled-out [automatically](https://azure.microsoft.com/en-us/documentation/articles/insights-how-to-scale/). | App Services deployed on an ASE can be scaled-out [automatically](https://azure.microsoft.com/en-us/documentation/articles/app-service-environment-auto-scale/).However an auto Scale-Out buffer is required. See section below. |

Points to consider
------------------

As seen above, Azure App Service Environments provide advanced features which might be necessary in enterprise applications. However, there are some additional considerations to bear in mind when architecting solutions to be deployed on these environments.

|  | **Without an App Service Environment** | **On an App Service Environment** |
| --- |  --- |  --- |
| **Use of Front-End Pool** | Azure App Service provides [load-balancing](https://azure.microsoft.com/en-us/services/app-service/web/) out-of-the-box.Thus, there is no need to have a Front-End Pool for load balancing. | The Front-End Pool contains [compute resources](https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-intro/) responsible for SSL termination and load balancing of app requests within an App Service Environment.However, these compute resources cannot host workloads. So depending on your workload, the Front-End Pool, of at least 2 x Premium P2 instances, could be seen as an overhead. |
| **Fault-tolerance overhead** | SLA is provided without requiring additional compute resources. | To provide fault tolerance, [one or more additional compute resources](https://azure.microsoft.com/en-us/documentation/articles/app-service-web-configure-an-app-service-environment/) have to be allocated per Worker Pool. This compute resource is not available to be assigned a workload. |
| **Auto Scale-Out buffer** | Auto Scale-Out does not require a buffer. | Because [Scale-Out operations in an App Service Environment](https://azure.microsoft.com/en-us/documentation/articles/app-service-environment-auto-scale/) take some time to apply, a buffer of compute resources is required to be able to respond to the demands of the App Service.The size of the buffer is calculated using the Inflation Rate formula [explained in detailed here](https://azure.microsoft.com/en-us/documentation/articles/app-service-environment-auto-scale/).This means that the compute resources of the buffer are idle until a Scale-Out operation happens. In many cases this could be considered as an overhead.E.g. if auto Scale-Out is configured for an App Service (1 to 2 instances), when only one 1 instance is being used, there is an overhead of 2 compute resources. 1 for fault-tolerance (explained above) and 1 for Scale-Out buffer. |
| **Deployment** | App Services can be deployed using Azure Resource Manager templates. | App Service Environments can be deployed using Azure Resource Manager templates. *[UPDATE July 2016]* And after the update, ASEs now support ARM VNETs (v2).In addition, deploying an App Service Environment usually takes more than 3 hours. |

Conclusion
==========

So coming back to original the question, when to use an App Service Environment? When is right to deploy App Services on an App Service Environment and to pay the premium price? In summary:

- When higher scalability is required. E.g. more than 20 instances per App Service Plan or larger instances like Premium P4 **OR**
- When inbound and outbound traffic control is required to secure the App Service **OR**
- When connecting the App Service to on-prem resources via a secure channel (ExpressRoute) without going by public Internet is necessary **OR**
- *[Update July 2016]* When access to the App Services has to be restricted to be only via a Site-to-Site or ExpressRoute VPN **OR**
- *[Update Sept 2016] *<del>When inspecting inbound web traffic and blocking potential attacks is needed without using Web Roles **OR**</del>
- When a static outbound IP Address for the App Service is required.

**AND**

- Very important, when there is enough business justification to pay for it (including potential overheads like Front-End Pool, fault-tolerance overhead, and auto Scale-Out buffer)

What else would you consider when deciding whether to use an App Service Environment for your workload or not? Feel free to post your comments or feedback!

Thanks for reading!

<p style="text-align:center;"><em>Cross-posted on <a href="https://blog.kloud.com.au/author/pacodelacruzag/">Kloud's blog</a>.<br/>
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>
