---
layout: post
title: Microsoft Azure Integration Platform as a Service (iPaaS) – Logic Apps and its Azure allies [Update]
date: 2018-02-02 08:00
author: Paco de la Cruz
comments: true
category: Azure iPaaS
tags: [API Management, Azure, Azure Functions, Azure iPaaS, Service Bus]
---
<h2>Introduction</h2>
A year ago, <a href="/microsoft-azure-ipaas/" target="_blank" rel="noopener noreferrer">I wrote a post</a> about this very same topic, the Microsoft Azure Integration Platform as a Service (iPaaS). At that time, the core iPaaS product offering, Azure Logic Apps, was roughly 6 months old, since it’s <a href="https://blogs.microsoft.com/firehose/2016/07/27/azure-logic-apps-microsofts-ipaas-offering-is-now-generally-available/" target="_blank" rel="noopener noreferrer">generally available launch</a>. Since the launch date, we’ve seen an impressive release cadence from the product team, an ever-growing list of connectors from other Microsoft product teams and third-party providers, and a considerable growth of the user-base.

In this post, I’ll talk about the current state of Azure Logic Apps and the other Azure services which are relevant when building application and process integration solutions. <span style="background-color:transparent;">It’s worth mentioning that there is another platform which targets hybrid data integration solutions called </span><a style="background-color:transparent;" href="https://azure.microsoft.com/en-au/services/data-factory/" target="_blank" rel="noopener noreferrer">Azure Data Factory</a><span style="background-color:transparent;">, however, that’s an </span><a style="background-color:transparent;" href="https://en.wikipedia.org/wiki/Extract,_transform,_load" target="_blank" rel="noopener noreferrer">ETL platform</a><span style="background-color:transparent;">, and won’t be discussed in this post.</span>
<h2>What's an Integration Platform as a Service (iPaaS)</h2>
According to <a href="https://www.gartner.com/it-glossary/information-platform-as-a-service-ipaas/" target="_blank" rel="noopener noreferrer">Gartner</a>, an “Integration Platform as a Service (iPaaS) is a suite of cloud services enabling development, execution and governance of integration flows connecting any combination of on premises and cloud-based processes, services, applications and data within individual or across multiple organizations.” Its capabilities <a href="https://www.gartner.com/doc/3645397/magic-quadrant-enterprise-integration-platform" target="_blank" rel="noopener noreferrer">usually include</a>:
<ul>
	<li>Communication Protocol Connectors (HTTP, SFTP, AS2, etc.)</li>
	<li>Application connectors, including SaaS and on-premises apps</li>
	<li>Ability to handle data formats, like JSON or XML, and data standards like EDIFACT, HL7, etc.</li>
	<li>Orchestration</li>
	<li>Routing</li>
	<li>Data validation and transformation</li>
	<li>Monitoring and Management tools</li>
	<li>Full life-cycle API Management</li>
	<li>Development and solution life-cycle management tools</li>
</ul>
Microsoft’s core iPaaS product offering is <a href="https://azure.microsoft.com/en-us/services/logic-apps/" target="_blank" rel="noopener noreferrer">Azure Logic Apps</a>. However, we could argue that one of the key differentiators and advantages of Microsoft within the iPaaS market, is how Logic Apps is enriched with the whole Azure ecosystem, which also keeps getting richer and better.
<h2>Azure Services to Build Integration Solutions</h2>
Based on Gartner’s definition of iPaaS, I’ll describe the different Azure services which we can leverage to develop, implement and manage enterprise-class integration solutions. The figure below, shows the different capabilities of an iPaaS, and which Azure product offerings we can use to implement them.

<img class="alignnone size-full wp-image-1139" src="/assets/img/2018/02/01-ipaas-components.png" alt="01 iPaaS Components" width="2768" height="1378" />
<h2>Orchestration, Data Handling and Transformation, and Routing.</h2>
Logic Apps workflows allow us to orchestrate our integrations solutions. They provide a workflow visual designer (via the Azure Portal or <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-deploy-from-vs" target="_blank" rel="noopener noreferrer">Visual Studio</a>) to design, build and manage our integration workflows. These workflows are <a href="http://martinfowler.com/articles/serverless.html" target="_blank" rel="noopener noreferrer">serverless</a>, which means that we can focus on the functionality and business value, while all the infrastructure and scaling are fully abstracted and we pay only for what we use. With Logic Apps we can implement long running workflows, as described in previous posts using <a href="/correlation-identifier-pattern-on-logic-apps/" target="_blank" rel="noopener noreferrer">the Webhook action</a>, and the <a href="/logic-apps-correlation-and-message-dependency-management-on-logic-apps-with-service-bus" target="_blank" rel="noopener noreferrer">Service Bus connector</a>. A figure of a Logic App workflow, which implements some conditions, a loop and some connectors, is shown below.

<img class="alignnone size-full wp-image-1140" src="/assets/img/2018/02/02-workflow.png" alt="02 workflow" width="1498" height="2282" />

Behind the scenes, Logic Apps workflows are based on the <a href="https://docs.microsoft.com/en-us/rest/api/logic/definition-language" target="_blank" rel="noopener noreferrer">Workflow Definition Language</a>. Within the Logic App, we can <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-xml-validation" target="_blank" rel="noopener noreferrer">validate messages</a> and transform messages using <a href="/transforming-json-objects-in-logic-apps/" target="_blank" rel="noopener noreferrer">Data Operations</a>, <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform" target="_blank" rel="noopener noreferrer">Liquid</a>, or <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-transform" target="_blank" rel="noopener noreferrer">XSLTs for XML documents</a>. For Routing, we can use different Logic Apps <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-use-logic-app-features#conditions-run-steps-only-after-meeting-a-condition" target="_blank" rel="noopener noreferrer">features</a> to control the workflow flow, and we can also use <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions" target="_blank" rel="noopener noreferrer">Azure Service Bus Topics Subscriptions</a> for Pub/Sub scenarios.

Additionally, we can <a href="/business-rules-on-azure-logic-apps-with-liquid-templates/" target="_blank" rel="noopener noreferrer">externalise Business Rules from Logic Apps workflows</a>.
<h2>Connectors</h2>
In addition to the Logic Apps workflows, the ever-growing <a href="https://docs.microsoft.com/en-us/azure/connectors/apis-list" target="_blank" rel="noopener noreferrer">list of connectors</a>, is another of the great features of this platform. These connectors allow us to trigger workflows and get data from and push data to many diverse apps on the cloud and on-premises through different protocols. Additionally, the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-connection" target="_blank" rel="noopener noreferrer">on-premises data gateway</a>, allows hybrid connectivity for some of the connectors. Below there is a snapshot of the 170+ connectors available at the time of writing. Please bear in mind that this list is always changing and that some connectors are still in preview and might not be available in all regions.
<table style="height:91px;background-color:#eeeeee;margin-left:auto;margin-right:auto;" border="0" width="90%" cellpadding="6">
<tbody>
<tr>
<td style="width:50%;vertical-align:top;text-align:left;"><strong>Protocol Connectors
</strong>
<ul>
	<li>FTP</li>
	<li>HTTP</li>
	<li>HTTP with Azure AD</li>
	<li>HTTP with Swagger</li>
	<li>RSS</li>
	<li>SFTP</li>
	<li>SMTP</li>
	<li>SOAP-to-REST</li>
	<li>WebHook</li>
	<li>AS2</li>
</ul>
</td>
<td style="width:50%;vertical-align:top;text-align:left;"><strong>Hybrid Connectors</strong>
<ul>
	<li>BizTalk</li>
	<li>DB2</li>
	<li>File System</li>
	<li>Informix</li>
	<li>MQ</li>
	<li>MySQL</li>
	<li>Oracle DB</li>
	<li>PostgreSQL</li>
	<li>REST</li>
	<li>SAP</li>
	<li>SharePoint</li>
	<li>SOAP (to REST and pass-through)</li>
	<li>SQL Server</li>
	<li>Teradata</li>
</ul>
</td>
</tr>
<tr>
<td style="width:50%;text-align:left;vertical-align:top;"><strong>Azure Connectors</strong>
<ul>
	<li>Azure AD</li>
	<li>Azure API Management</li>
	<li>Azure App Services</li>
	<li>Azure Application Insights</li>
	<li>Azure Automation</li>
	<li>Azure Blob Storage</li>
	<li>Azure Container Instance</li>
	<li>Azure Data Lake</li>
	<li>Azure Data Factory</li>
	<li>Azure Event Grid</li>
	<li>Azure File Storage</li>
	<li>Azure Functions</li>
	<li>Azure Kusto</li>
	<li>Azure Logic Apps</li>
	<li>Azure ML</li>
	<li>Azure Resource Manager</li>
	<li>Azure Security Center</li>
	<li>Azure SQL Data Warehouse</li>
	<li>Azure Storage Queues</li>
	<li>Azure Table Storage</li>
	<li>Computer Vision API</li>
	<li>Common Data Service</li>
	<li>Content Moderator</li>
	<li>Cosmos DB</li>
	<li>Custom Vision</li>
	<li>Event Hubs</li>
	<li>Face API</li>
	<li>LUIS</li>
	<li>QnA Maker</li>
	<li>Service Bus</li>
	<li>SQL Server</li>
	<li>Text Analytics</li>
	<li>Video Indexer</li>
</ul>
</td>
<td style="width:50%;text-align:left;vertical-align:top;"><strong>Other Microsoft Connectors</strong>
<ul>
	<li>Bing Maps</li>
	<li>Bing Search</li>
	<li>Dynamics 365</li>
	<li>Dynamics 365 for Financials</li>
	<li>Dynamics Nav</li>
	<li>Microsoft Forms</li>
	<li>Microsoft Kaizala</li>
	<li>Microsoft StaffHub</li>
	<li>Microsoft Teams</li>
	<li>Microsoft To-Do</li>
	<li>Microsoft Translator</li>
	<li>MSN Weather</li>
	<li>Office 365 Excel</li>
	<li>Office 365 Groups</li>
	<li>Office 365 Outlook</li>
	<li>Office 365 Video</li>
	<li>OneDrive</li>
	<li>OneDrive for Business</li>
	<li>OneNote</li>
	<li>Outlook Customer Manager</li>
	<li>Outlook Tasks</li>
	<li>Outlook.com</li>
	<li>Project Online</li>
	<li>Power BI</li>
	<li>SharePoint</li>
	<li>Skype for Business</li>
	<li>VSTS</li>
	<li>Yammer</li>
</ul>
</td>
</tr>
<tr>
<td style="width:50%;vertical-align:top;text-align:left;"><strong>Third-Party SaaS Connectors</strong>
<ul>
	<li>10to8</li>
	<li>Adobe Creative Cloud</li>
	<li>Apache Impala</li>
	<li>Appfigures</li>
	<li>Asana</li>
	<li>Aweber</li>
	<li>Basecamp3</li>
	<li>Benchmark Email</li>
	<li>Bitbucket</li>
	<li>Bitly</li>
	<li>Blogger</li>
	<li>Box</li>
	<li>Buffer</li>
	<li>Calendly</li>
	<li>Campfire</li>
	<li>Capsule CRM</li>
	<li>Chatter</li>
	<li>Cognito Forms</li>
	<li>D&amp;B Optimizer</li>
	<li>Derdack Signl4</li>
	<li>DocFusion</li>
	<li>Docparser</li>
	<li>DocuSign</li>
	<li>Dropbox</li>
	<li>Easy Redmine</li>
	<li>Elastic Forms</li>
	<li>Enadoc</li>
	<li>Eventbrite</li>
	<li>Facebook</li>
	<li>FlowForma</li>
	<li>FreshBooks</li>
	<li>Freshdesk</li>
	<li>Freshservice</li>
	<li>GitHub</li>
	<li>Gmail</li>
	<li>Google Calendar</li>
	<li>Google Drive</li>
	<li>Google Sheets</li>
	<li>Google Tasks</li>
	<li>GoToMeeting</li>
	<li>GoToTraining</li>
	<li>GoToWebinar</li>
	<li>Harvest</li>
	<li>HelloSign</li>
	<li>HipChat</li>
	<li>iAuditor</li>
	<li>Infobip</li>
	<li>Infusionsoft</li>
	<li>Inoreader</li>
	<li>insightly</li>
	<li>Instagram</li>
	<li>Instapaper</li>
	<li>Intercom</li>
	<li>Jira</li>
</ul>
</td>
<td style="width:50%;vertical-align:top;text-align:left;">
<ul>
	<li>JotForm</li>
	<li>Kintone</li>
	<li>LeanKit</li>
	<li>LiveChat</li>
	<li>Lithium</li>
	<li>MailChimp</li>
	<li>Mandrill</li>
	<li>Marketing Content Hub</li>
	<li>Metatask</li>
	<li>Muhimbi PDF</li>
	<li>MySQL</li>
	<li>Nexmo</li>
	<li>Oracle Database</li>
	<li>Pager Duty</li>
	<li>Parserr</li>
	<li>Paylocity</li>
	<li>Pinterest</li>
	<li>Pipedrive</li>
	<li>Pitney Bowes Data Validation</li>
	<li>Pivotal Tracker</li>
	<li>Planner</li>
	<li>Plivo</li>
	<li>Plumsail Documents</li>
	<li>Plumsail Forms</li>
	<li>Plumsail SP</li>
	<li>PostgreSQL</li>
	<li>Redmine</li>
	<li>Salesforce</li>
	<li>SendGrid</li>
	<li>ServiceNow</li>
	<li>Slack</li>
	<li>Smartsheet</li>
	<li>SparkPost</li>
	<li>Stripe</li>
	<li>SurveyMonkey</li>
	<li>Tago</li>
	<li>Teamwork Projects</li>
	<li>Teradata</li>
	<li>Todoist</li>
	<li>Toodledo</li>
	<li>Trello</li>
	<li>Twilio</li>
	<li>Twitter</li>
	<li>Typeform</li>
	<li>UserVoice</li>
	<li>Vimeo</li>
	<li>WebMerge</li>
	<li>WordPress</li>
	<li>Workday HCM</li>
	<li>Workday Finance</li>
	<li>Wunderlist</li>
	<li>YouTube</li>
	<li>Zendesk</li>
	<li>Zoho</li>
</ul>
</td>
</tr>
</tbody>
</table>
A key component of Logic Apps connectivity is the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-connection" target="_blank" rel="noopener noreferrer">on-premises data gateway</a>, which allows the connectors listed above as "Hybrid" to connect to on-premises resources. The data gateway is an agent that can be installed on a server on-premises or on a VM on an Azure VNET. All the interchanges between the data gateway and Logic Apps are outgress (from on-premises to Azure) via a managed Service Bus Relay with encrypted channels. The data gateway can be installed on more than one VM to provide high-availability.

<img class="alignnone size-full wp-image-1145" src="/assets/img/2018/02/03-on-prem-data-gateway.png" alt="03 On Prem data Gateway" width="1218" height="537" />
<h2>Serverless Compute (Custom Code and Custom Connectors)</h2>
As shown in the previous sections, Logic Apps provide a lot of connectors and functionality for message processing, however, in some scenarios, we might need to write custom code. With Azure Functions, we can write custom nano-services in C#, F#, Javascript or any other of the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/supported-languages" target="_blank" rel="noopener noreferrer">supported languages</a>, and we can invoke those functions from our Logic Apps synchronously via HTTP or asynchronously via Azure Service Bus or Azure Event Grid, as described below.

We can use <a href="https://azure.microsoft.com/en-gb/services/functions/" target="_blank" rel="noopener noreferrer">Azure Functions</a> as part of our custom message processing or to send or receive messages to or from an application for which there is no out-of-the-box connector.

Additionally, we can use Azure Functions <a href="https://docs.microsoft.com/en-us/azure/app-service/app-service-hybrid-connections" target="_blank" rel="noopener noreferrer">Hybrid connections</a> to securely connect to resources on-premises via outbound calls to Azure Service Bus Relay.
<h2>Messaging and Eventing</h2>
Integration solutions usually require a way to send and receive messages in an asynchronous way. Additionally, modern computing is all about events, and increasingly, integration solutions have to react to or push events.

<a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions" target="_blank" rel="noopener noreferrer">Azure Service Bus Messaging</a> provides reliable message queuing and publish-subscribe messaging capabilities. Azure Service Bus Queues and Topics allow us to decouple in time upstream and downstream systems or different interrelated services. <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions#queues" target="_blank" rel="noopener noreferrer">Service Bus Queues</a> provide ordered message delivery to competing consumers, while <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions#topics-and-subscriptions" target="_blank" rel="noopener noreferrer">Service Bus Topics</a> enable publish-subscribe messaging scenarios where multiple consumers can process the same message.

The following Service Bus features make it a key component of most integration solutions on Azure:
<ul>
	<li>temporal decoupling</li>
	<li>message processing load balancing (competing consumers)</li>
	<li>Publish/Subscribe pattern via Service Bus Topics;</li>
</ul>
<a href="https://azure.microsoft.com/en-us/services/event-grid/" target="_blank" rel="noopener noreferrer">Azure Event Grid</a> is a fully-managed event-routing platform on Azure which we can leverage as part of our integration solutions.  Logic Apps can react to events on Event Grid coming from the different <a href="https://docs.microsoft.com/en-us/azure/event-grid/overview#event-publishers" target="_blank" rel="noopener noreferrer">publishers</a> or publish events for other <a href="https://docs.microsoft.com/en-us/azure/event-grid/overview#event-handlers" target="_blank" rel="noopener noreferrer">handlers</a> to consume.

While Service Bus provides rich and robust messaging capabilities, like transactions, ordering, sessions, dead-lettering, enqueue time, long time to live and deduplication, among others; Event Grid offers hyper-scale event routing, with filtering, routing, built-in Azure publishers, with a push-push model, but a short time to live and no ordering. Additionally, due to these limitations, Event Grid is not meant for critical or transactional messages, but for events, which might still point to its source, e.g. the event of a blob being created containing the URL of the actual blob.
<h2>API Mediation and Management</h2>
API Mediation and Management is a core funcitonality of integration solutions which require to expose RESTful APIs.

<a href="https://docs.microsoft.com/en-au/azure/api-management/" target="_blank" rel="noopener noreferrer">Azure API Management</a> is an Azure service which functions as a API gateway for backend APIs in the cloud or on-premises. Some of the benefits of API Management are:
<ul>
	<li>Securing APIs with different authentication and authorisation methods, like <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-protect-backend-with-aad" target="_blank" rel="noopener noreferrer">Azure AD</a> or use of <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-mutual-certificates" target="_blank" rel="noopener noreferrer">client certificates</a></li>
	<li>A <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-key-concepts#a-namedeveloper-portal-a-developer-portal" target="_blank" rel="noopener noreferrer">developer’s portal</a> to publish APIs and speed up the on-boarding process</li>
	<li>Managing <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-create-or-invite-developers" target="_blank" rel="noopener noreferrer">consumer accounts</a></li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-using-with-vnet" target="_blank" rel="noopener noreferrer">VNET connectivity</a> to connect to other secured resources and on-premises APIs</li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-sample-cache-by-key" target="_blank" rel="noopener noreferrer">Caching</a> to improve API response times</li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-sample-flexible-throttling" target="_blank" rel="noopener noreferrer">Throttling requests</a></li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies#ForwardRequest" target="_blank" rel="noopener noreferrer">Routing or forwarding</a></li>
	<li>Transforming requests, by <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies#SetRequestMethod" target="_blank" rel="noopener noreferrer">changing the method</a> or implementing <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-transformation-policies" target="_blank" rel="noopener noreferrer">richer transformation policies</a></li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-api-inspector" target="_blank" rel="noopener noreferrer">Tracing calls</a>, or <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies#log-to-eventhub" target="_blank" rel="noopener noreferrer">logging requests via Event Hubs</a></li>
	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-get-started#a-nameview-analytics-aview-analytics" target="_blank" rel="noopener noreferrer">Monitoring and analysing usage and health</a> of APIs</li>
	<li><a href="https://blogs.msdn.microsoft.com/apimanagement/2016/08/25/how-to-mock-responses-in-api-management/" target="_blank" rel="noopener noreferrer">Mocking responses</a></li>
	<li>Monetising APIs</li>
</ul>
<span style="background-color:transparent;">Furthermore, </span><a style="background-color:transparent;" href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-proxies" target="_blank" rel="noopener noreferrer">Azure Functions Proxies</a><span style="background-color:transparent;"> provide a small subset of what API Management does, which can be leveraged as a light API Gateway for HTTP-triggered Logic Apps, including:</span>
<ul>
	<li>Azure Functions Proxies can be secured in a very similar way to <a href="https://docs.microsoft.com/en-us/azure/app-service/app-service-authentication-overview" target="_blank" rel="noopener noreferrer">App Services</a>.</li>
	<li>Modify <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-proxies#modify-requests-responses" target="_blank" rel="noopener noreferrer">requests and responses</a>, which also allows us to mock APIs.</li>
	<li>Consolidating multiple and disperse APIs into simpler URL routes.</li>
</ul>
Depending on the level of features we require, we can go with the lightweight Azure Functions Proxies or with the full API Management for our integration solutions.
<h2>Monitoring and Management</h2>
Azure Logic Apps provide <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms" target="_blank" rel="noopener noreferrer">built-in monitoring tools</a> that allow you to check the run history, trigger history, status, performance, etc. You can also install the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms#install-the-logic-apps-management-solution-in-oms" target="_blank" rel="noopener noreferrer">Logic Apps Management Solution</a> on OMS, which gives you a very rich aggregated view and charts of all your logic apps that are being monitored. The OMS Logic Apps Management Solution view is shown in the figure below.

<img class="alignnone size-full wp-image-1141" src="/assets/img/2018/02/11-oms-solution.png" alt="11 OMS Solution" width="2176" height="1088" />

Additionally, you can build custom activity monitoring dashboards and publish custom queries based on diagnostics sent to <a href="https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-overview" target="_blank" rel="noopener noreferrer">Azure Log Analytics</a>. More information on my previous posts:
<ul>
	<li><a href="/business-activity-monitoring-on-azure-logic-apps/" target="_blank" rel="noopener noreferrer">Business Activity Monitoring on Azure Logic Apps with Azure Log Analytics</a> and</li>
	<li><a href="/publishing-custom-queries-of-logic-apps-execution-logs/" target="_blank" rel="noopener noreferrer">Publishing Custom Queries of Logic Apps Execution Logs</a></li>
</ul>
A sample of what you can build in the figure as follows.

<img class="alignnone size-full wp-image-1142" src="/assets/img/2018/02/12-bam-dashboard.gif" alt="12 BAM Dashboard" width="1696" height="656" />
<h2>Development and Solution Life-cycle Management</h2>
To develop some of the iPaaS Solution components we can use the Azure Portal, however, when we think about the whole CI/CD pipeline, it’s better to work with Visual Studio and with Visual Studio Team Services (VSTS) for source control, build and release defections. More information about CI/CD for Logic Apps on my previous post:
<ul>
	<li><a href="/preparing-azure-logic-apps-for-cicd/" target="_blank" rel="noopener noreferrer">Preparing Azure Logic Apps for CI/CD using ARM Templates</a></li>
</ul>
<h2>Handy References</h2>
If you want to know more about Logic Apps, you might find the following links handy
<ul>
	<li style="list-style-type:none;">
<ul>
	<li>Logic Apps Twitter <a href="https://twitter.com/logicappsio" target="_blank" rel="noopener noreferrer">@logicappsio</a></li>
	<li><a href="https://aka.ms/logicapps-docs" target="_blank" rel="noopener noreferrer">Logic Apps Documentation</a></li>
	<li><a href="https://aka.ms/logicappsblog" target="_blank" rel="noopener noreferrer">Logic Apps Blog</a></li>
	<li><a href="https://aka.ms/logicappslive" target="_blank" rel="noopener noreferrer">Logic Apps Webcasts</a></li>
	<li><a href="https://blogs.msdn.microsoft.com/logicappsupdate/" target="_blank" rel="noopener noreferrer">Logic Apps Release update</a></li>
	<li><a href="http://aka.ms/logicappsroadmap" target="_blank" rel="noopener noreferrer">Logic Apps Roadmap</a></li>
	<li><a href="https://aka.ms/logicapps-wish" target="_blank" rel="noopener noreferrer">Logic Apps Feature requests</a></li>
</ul>
</li>
</ul>
&nbsp;
<h2>Wrapping Up</h2>
In this post, I’ve discussed the current state of the Microsoft Azure Integration Platform as a Service (iPaaS) components, being Logic Apps the main player and complemented by other Azure product offerings. As mentioned previously, one of the main advantages of Microsoft in the iPaaS market is the whole Azure ecosystem and how we can build integration solutions leveraging the capabilities of all these different platforms. And we need to consider not only the technologies described here, but also all those which can be utilised through the connectors; such as <a href="https://azure.microsoft.com/en-gb/services/cognitive-services/" target="_blank" rel="noopener noreferrer">Azure Cognitive Services</a> and <a href="https://azure.microsoft.com/en-us/services/machine-learning-studio/" target="_blank" rel="noopener noreferrer">Azure Machine Learning</a>.

Additionally, many of these components offer per-execution price or affordable entry-level pricing options. This enables us to start small when building enterprise-class integration solutions and grow from there.

I hope you’ve found this post useful, and please feel free to ask any questions or add your comments below.

Happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
