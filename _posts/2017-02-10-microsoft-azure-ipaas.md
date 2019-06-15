---
layout: post
title: Microsoft Azure Integration Platform as a Service (iPaaS) – Logic Apps, Azure Functions, API Management and Service Bus working together
date: 2017-02-10 23:44
author: Paco de la Cruz
comments: true
category: Azure iPaaS
tags: [API Management, Azure, Azure Functions, Azure iPaaS, Logic Apps, Microsoft iPaaS, Service Bus]
---

<h1>Introduction</h1>
If you work for an established organisation going through <a href="http://www.cio.com/article/3063620/it-strategy/digital-transformation-why-its-important-to-your-organization.html">Digital Transformation</a> or in a modern company born in the digital era, the chances are that IT is required to implement integration solutions more than ever before. Whether an organisation is <a href="http://economics.mit.edu/files/11347">an incumbent or a new entrant</a>, they need to leverage the power of technology to lead the change or embrace it. Monolithic systems will not be capable of meeting the needs of digital organisations, creating the need to interconnect existing systems with best-of-breed cloud and distributed apps and to expose all these systems to other systems, business partners or consumer apps through easy to consume <a href="https://en.wikipedia.org/wiki/Application_programming_interface">APIs</a>.

Microsoft provides different technologies on Azure which enable us to build robust application, data and process integration solutions. One of the core offerings for Azure integration is Logic Apps – however there are other Azure PaaS offerings which when used with Logic Apps form the robust building blocks of first-class integration solutions. These technologies together provide a very rich and fully-managed Integration Platform as a Service (iPaaS). In this post, I will try to describe these Azure technologies that we can leverage to make our life easier, and perhaps even more fun, when implementing enterprise integration solutions.
<h1>Azure Services to build integration solutions</h1>
Out of the very large and growing list of Azure services, there are some that we can utilise to build our integration solutions, including <a href="https://azure.microsoft.com/en-us/services/logic-apps/">Logic Apps</a>, <a href="https://azure.microsoft.com/en-us/services/functions/">Azure Functions</a>, <a href="https://azure.microsoft.com/en-us/services/api-management/">API Management</a>, and <a href="https://azure.microsoft.com/en-us/services/service-bus/">Service Bus Messaging</a>. I will briefly describe each of them below.
<h2>Logic Apps</h2>
<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-what-are-logic-apps">Logic Apps</a> is a very robust and powerful platform to orchestrate and automate integration workflows. The visual designer together with all the <a href="https://docs.microsoft.com/en-us/azure/connectors/apis-list">available connectors</a>, and deployment and management tools make it a great tool for this kind of scenarios. Logic Apps also provides the concept of <a href="http://martinfowler.com/articles/serverless.html">serverless</a> computing, in which you just focus on what you want to achieve, without worrying at all about servers, patching, scaling, etc.
<h3>Workflow and Orchestration Capabilities</h3>
Logic Apps workflows, which can be easily designed and implemented graphically (via the Azure Portal or <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-deploy-from-vs">Visual Studio</a>), are based on the <a href="https://docs.microsoft.com/en-us/rest/api/logic/definition-language">Workflow Definition Language</a>. These workflows provide rich ways to process and manipulate data than can be obtained or pushed via different <a href="https://docs.microsoft.com/en-us/azure/connectors/apis-list">connectors</a>. Below there is a figure which shows a simple Logic App workflow with different actions, a condition and a loop.

<img src="/assets/img/2017/03/030617_1239_microsoftaz1.png" alt="" />

In Logic Apps workflows we can implement <a href="https://docs.microsoft.com/en-us/rest/api/logic/actions-and-triggers">different actions</a>, like calling an HTTP endpoint, calling Azure API Apps, calling WebHooks, waiting, calling a Logic App nested workflow, implementing conditions and loops, query arrays, terminating a workflow, or returning an HTTP Response.
<h3>Triggers, Input and Outputs</h3>
Logic Apps provides a growing list of <a href="https://docs.microsoft.com/en-us/azure/connectors/apis-list">connectors</a> that allow us to trigger workflows and get data from and push data to many different protocols, SaaS applications, other Azure and Power Apps Services, and on-premises systems. Below there is a snapshot of the 100+ connectors available at the time of writing (few of them are still in preview).

<strong>Protocol Connectors
</strong>

<img src="/assets/img/2017/03/030617_1239_microsoftaz2.png" alt="" />
<p style="text-align:justify;"><strong>Azure Services and Power Apps Connectors
</strong></p>
<img src="/assets/img/2017/03/030617_1239_microsoftaz3.png" alt="" />
<p style="text-align:justify;"><strong>SaaS Connectors
</strong></p>
<p style="text-align:justify;"><img src="/assets/img/2017/03/030617_1239_microsoftaz4.png" alt="" /></p>
<p style="text-align:justify;"><strong>B2B, XML, EDI and AS2 Connectors
</strong></p>
<p style="text-align:justify;"><img src="/assets/img/2017/03/030617_1239_microsoftaz5.png" alt="" /></p>
<p style="text-align:justify;"><strong>Hybrid Connectors
</strong></p>
<p style="text-align:justify;"><img src="/assets/img/2017/03/030617_1239_microsoftaz6.png" alt="" /></p>
<p style="text-align:justify;">As we can see, with all these connectors, Logic Apps allows us to very easily and quickly connect to many different protocols, apps, and also to other Azure Services. Additionally, the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-overview">Enterprise Integration Pack</a> brings some of the BizTalk Server functionality to Logic Apps and makes really easy to implement B2B integrations based on AS2 and EDI.</p>

<h2>Azure Functions</h2>
Another Azure service that is very useful when building integration solutions is Azure Functions. <a href="https://azure.microsoft.com/en-us/documentation/articles/functions-overview/">Azure Functions</a> are the evolution of <a href="https://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/">Azure WebJobs</a>, which allow us to create microservices or small pieces of code that can run synchronously or asynchronously as part of composite and distributed cloud solutions. Azure Functions are built on top of the <a href="https://github.com/Azure/azure-webjobs-sdk">WebJobs SDK</a> but with some additional benefits and capabilities, including the option of being deployed on a serverless-style <a href="https://azure.microsoft.com/en-us/documentation/articles/functions-scale/">Dynamic Service Plan</a>, in which you pay per consumption only, and also the capability of being triggered via HTTP. Additionally, with Functions you can deploy from very simple to quite complex pieces of code developed on different languages, such as <strong>bash (.sh)</strong>,<strong> batch (.bat / .cmd)</strong>,<strong> C#</strong>,<strong> F#</strong>, <strong>Node.Js</strong>,<strong> PHP</strong>,<strong> PowerShell</strong>, and<strong> Python</strong>.
<h3>Triggers, Inputs and Outputs</h3>
Azure Functions support <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings">different triggers, input and output bindings</a>, which I summarise in the table below. These bindings make Functions very easy to be called and allow them to get data from and push data to other microservices, Logic Apps or other systems or services.
<div>
<table style="border-collapse:collapse;" border="0"><colgroup> <col style="width:260px;" /> <col style="width:71px;" /> <col style="width:59px;" /> <col style="width:73px;" /></colgroup>
<tbody valign="top">
<tr>
<td style="padding-left:9px;padding-right:9px;border:solid .5pt;"><strong>Type / Service</strong></td>
<td style="padding-left:9px;padding-right:9px;border-top:solid .5pt;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Trigger</strong></td>
<td style="padding-left:9px;padding-right:9px;border-top:solid .5pt;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Input</strong></td>
<td style="padding-left:9px;padding-right:9px;border-top:solid .5pt;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"><strong>Output</strong></td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer">Schedule</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook">HTTP Call</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob">Azure Blob Storage</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs">Azure Event Hubs</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue">Azure Storage Queues</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-service-bus">Azure Service Bus Messaging</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-table">Azure Storage Tables</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-mobile-apps">Azure Mobile Apps Tables</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-documentdb">Azure DocumentDB</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-notification-hubs">Azure Notification Hubs</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-twilio">Twilio SMS Message</a></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;">SendGrid Emails (Not fully documented yet)</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;">Files in Cloud File Storage SaaS, such as Box, DropBox, OneDrive, FTP and SFTP (Not fully documented yet)</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">*</td>
</tr>
</tbody>
</table>
</div>
<h3>Processing</h3>
Integration Processes usually require custom data and message validation, enrichment, transformation or routing. These can simply be done via custom code on Azure Functions.
<h2>Service Bus Messaging</h2>
<a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions">Azure Service Bus Messaging</a> provides reliable message queuing and publish-subscribe messaging capabilities. Azure Service Bus Queues and Topics allow us to decouple in time upstream and downstream systems or different interrelated services. <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions">Service Bus Queues</a> provide ordered message delivery to competing consumers, while <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions">Service Bus Topics</a> enable publish-subscribe messaging scenarios where multiple consumers can process the same message.

The following Service Bus features make it a key component of most integration solutions on Azure: <strong>
</strong>
<ul>
	<li>temporal decoupling;<strong>
</strong></li>
	<li>the ability to load balance message processing using competing consumers;<strong>
</strong></li>
	<li>the capability of implementing Publish/Subscribe architecture via Service Bus Topics;<strong>
</strong></li>
	<li>and the fact that Functions and Logic Apps can very easily read and write messages from and to Service Bus.<strong>
</strong></li>
</ul>
<h2>API Management</h2>
<a href="https://docs.microsoft.com/en-au/azure/api-management/">API Management</a> is an Azure Service which functions as a API Gateway or Front-End for backend APIs in the cloud or on-prem. In addition, it provides a Developer portal which helps to speed up the adoption and use of the implemented APIs. Some of the benefits of API Management are:
<ul>
	<li>Ability to secure APIs by implementing different authentication and authorisation methods, like <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-protect-backend-with-aad">Azure AD</a> or use of <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-mutual-certificates">client certificates</a></li>
	<li>Managing <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-create-or-invite-developers">consumer accounts</a></li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-using-with-vnet">VNET connectivity</a> to connect to other secured resources</li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-sample-cache-by-key">Caching</a> to improve API response times</li>
	<li>Ability to <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-sample-flexible-throttling">throttle requests</a> based on the caller or the product</li>
	<li>API call <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies">routing or forwarding</a></li>
	<li>Transforming requests, by <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies">changing the method</a> or implementing <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-transformation-policies">richer transformation policies</a></li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-api-inspector">Tracing calls</a>, or <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies">logging requests via Event Hubs</a></li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-get-started">Monitoring and analysing usage and health</a> of APIs</li>
	<li>Having a <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-key-concepts">developer's portal</a> which allows to publicise APIs and speed up adoption and the on-boarding process</li>
	<li>Ability to monetise APIs</li>
</ul>
Thanks to all these features, API Management can be leveraged on many integration solutions which require to expose RESTful APIs and require any kind of mediation.
<h1>When to use each of these technologies and how they complement each other</h1>
Now that I have described the Azure iPaaS offerings, it's worth analysing when we could use each of these technologies and how they complement each other. I summarise this in the table below.
<div>
<table style="border-collapse:collapse;" border="0"><colgroup> <col style="width:118px;" /> <col style="width:307px;" /> <col style="width:354px;" /></colgroup>
<tbody valign="top">
<tr style="background:#f2f2f2;">
<td style="padding-left:9px;padding-right:9px;border:solid .5pt;"><span style="font-size:9pt;"><strong>Technology</strong></span></td>
<td style="padding-left:9px;padding-right:9px;border-top:solid .5pt;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"><span style="font-size:9pt;"><strong>When to use it</strong></span></td>
<td style="padding-left:9px;padding-right:9px;border-top:solid .5pt;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"><span style="font-size:9pt;"><strong>Use together with</strong></span></td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><span style="font-size:9pt;">Logic Apps</span></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;">To implement and orchestrate visually designed integration workflows.
</span></li>
	<li><span style="font-size:9pt;">To orchestrate distributed microservices.
</span></li>
	<li><span style="font-size:9pt;">To leverage the 100+ connectors to interact with different protocols, SaaS systems and services, other Azure services, and on-premises systems.
</span></li>
	<li><span style="font-size:9pt;">To implement cloud-based B2B integrations with AS2 and EDI. </span></li>
</ul>
</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;"><strong>Functions</strong> - so custom logic can be implemented in microservices to be orchestrated by LogicApps.
</span></li>
	<li><span style="font-size:9pt;"><strong>Service Bus </strong>-<strong>
</strong>to decouple in time different microservices and steps in the integration process. <strong>
</strong></span></li>
	<li><span style="font-size:9pt;"><strong>API Management</strong> - for those HTTP triggered apps when some of the capabilities of API Management are required.</span></li>
</ul>
</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><span style="font-size:9pt;">Azure Functions</span></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;">To implement code-based microservices or processing.</span></li>
</ul>
</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;"><strong>Logic Apps </strong>-<strong>
</strong>so different microservices can be orchestrated.
</span></li>
	<li><span style="font-size:9pt;"><strong>Azure Service Bus Messaging</strong> - to decouple in time different microservices.
</span></li>
	<li><span style="font-size:9pt;"><strong>API Management</strong> - for those HTTP triggered functions when some of the capabilities of API Management are required.</span></li>
</ul>
</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><span style="font-size:9pt;">Service Bus</span></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;">To decouple in time upstream systems from downstream systems or different microservices.
</span></li>
	<li><span style="font-size:9pt;">To implement multiple consumers on a Publish/Subscribe pattern.
</span></li>
	<li><span style="font-size:9pt;">To allow load distribution with multi-instance competing consumers. </span></li>
</ul>
</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;"><strong>Functions and Logic Apps </strong>-<strong>
</strong>to decouple in time different microservices and steps in the integration process. <strong>
</strong></span></li>
</ul>
</td>
</tr>
<tr>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:solid .5pt;border-bottom:solid .5pt;border-right:solid .5pt;"><span style="font-size:9pt;">API Management</span></td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;">When any of the API Management features is required, for example: securing backend APIs, API response caching, request throttling, request routing, request transformation, API calls tracing or logging, usage and health analytics, or providing a rich portal for developers. </span></li>
</ul>
</td>
<td style="padding-left:9px;padding-right:9px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;">
<ul>
	<li><span style="font-size:9pt;"><strong>Functions and Logics Apps </strong>- that are<strong>
</strong>triggered by an HTTP call and require some kind of mediation. </span></li>
</ul>
</td>
</tr>
</tbody>
</table>
</div>
<h1>Summary</h1>
Azure provides a very robust Integration Platform as a Service (iPaaS), which is based on Logic Apps and can be complemented with Azure Functions, Service Bus Messaging and API Management

<img src="/assets/img/2017/03/030617_1239_microsoftaz7.png" alt="" />

The breadth and capabilities of many different Azure technologies and how they complement each other is what differentiates Azure against other iPaaS vendors. We can leverage many different services to build first-class integration solutions. <strong>Logic Apps</strong> is the core engine to implement these. Logic Apps can connect to many different protocols, SaaS apps and Services, to other Azure and Power App Services, to on-premises systems and to B2B trading partners via a growing list of connectors. Logic Apps integration workflows can easily be extended with custom code implemented as microservices on <strong>Azure Functions</strong>. In order to decouple in time these integration processes, we can leverage <strong>Service Bus Messaging</strong> services. And in case we need to expose our integration services as RESTful APIs, we might want to make use of all the features and capabilities of <strong>API Management</strong>. Additionally, these integration solutions can be enhanced by other Azure services, such as Cognitive Services to, for example get the sentiment from social media feeds or Azure Active Directory for authenticating and authorising calls to our APIs. All of this with all the PaaS magic and the powerful DevOps capabilities of Azure.

Thanks for reading, and please share your thoughts or queries on this topic below <span style="font-family:Wingdings;">J</span>

Happy clouding!

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
