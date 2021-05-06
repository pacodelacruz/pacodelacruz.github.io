---
layout: post
title: Platform Management - Enterprise Integration Patterns on Azure
date: 2020-12-10 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Enterprise Integration Patterns, Azure iPaaS, Logic Apps, Service Bus, Event Grid, Azure Functions]
---

<p><img src="/assets/img/2019/04/Management.jpg" alt="Management" width="1000" style="width: 1000px;"></p>
<p>When implementing modern application integration solutions, we rely on platforms that abstract many of the challenges inherent to these types of solutions. While the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DynamicRouter.html" rel="noopener" target="_blank">Enterprise Integration Patterns</a> previously covered in this series describe architectural and implementation patterns, the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/SystemManagementIntro.html" rel="noopener" target="_blank"><strong>Platform Management</strong></a> patterns focus on the operational aspects of an integration platform. In this post, we cover how the Platform Management patterns can be implemented on Azure.</p>
<!--more-->
<p>This is the last post of the series describing how to implement the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DynamicRouter.html" rel="noopener" target="_blank">Enterprise Integration Patterns</a> using the <a href="/microsoft-azure-ipaas" rel="noopener" target="_blank">Azure Integration Services</a>:</p>
<ol>
<li><a href="/enterprise-integration-patterns-on-azure-intro" rel="noopener" target="_blank">Introduction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-message-construction" rel="noopener" target="_blank">Message Construction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank">Messaging Channels</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-endpoints" rel="noopener" target="_blank">Messaging Endpoints</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-routing" rel="noopener" target="_blank">Message Routing</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-transformation" rel="noopener" target="_blank">Message Transformation</a></li>
<li>Platform Management (this)</li>
</ol>
<p>The patterns covered in this article are listed below.</p>
<ul>
<li><a href="#control-bus">Control Bus</a></li>
<li><a href="#detour">Detour</a></li>
<li><a href="#wire Tap">Wire Tap</a></li>
<li><a href="#message-history">Message History</a></li>
<li><a href="#message-store">Message Store</a></li>
<li><a href="#tracing-correlation-identifier">Tracing Correlation Identifier (*)</a></li>
<li><a href="#message-resubmission">Message Resubmission (*)</a></li>
<li><a href="#repair-and-resubmit">Repair and Resubmit (*)</a></li>
<li><a href="#smart-proxy">Smart Proxy</a></li>
<li><a href="#monitoring-events-and-alerts">Monitoring Events and Alerts (*)</a></li>
<li><a href="#test-message">Test Message</a></li>
<li><a href="#channel-purger">Channel Purger</a></li>
</ul>
<h2 id="control-bus">Control Bus</h2>
<p>Integration solutions are distributed in nature. Different components, including channels (pipes), processors (filters), and process managers, can reside in multiple networks or locations. To manage and support an integration solution, we need to understand not only what the different components are but also their health status, metrics, alerts, exception messages, etc. The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/ControlBus.html" rel="noopener" target="_blank">Control Bus</a> is meant to provide this visibility.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Generic%20code.png" alt="Generic code" width="80" style="width: 80px;"></strong></td>
<td width="644">
<p>A custom <a href="https://docs.microsoft.com/en-us/azure/azure-portal/azure-portal-dashboards" rel="noopener" target="_blank">Azure Dashboard</a> can be created that includes charts, metrics, log queries, and links to the relating resources. Azure resource groups could also be used to group relating components.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Generic%20code.png" alt="Generic code" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p><a href="https://www.serverless360.com/blog/what-are-composite-applications-in-serverless360" rel="noopener" target="_blank">Serverless360</a> is a third-party tool that allows you to define composite applications for management and monitoring purposes.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="detour">Detour</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/Detour.html" rel="noopener" target="_blank"><strong>Detour</strong></a> pattern proposes that messages can be routed to an alternate path at run time using a <a href="/enterprise-integration-patterns-on-azure-routing#content-based-router" rel="noopener" target="_blank"><strong>Context-Based Router</strong></a> and toggled using the <strong>Control Bus</strong>. This alternate path might be a set of <a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank" style="font-weight: bold;">Pipes and Filters</a> with additional steps.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;">
<p><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></p>
</td>
<td width="644">
<p><strong>Context-based routing</strong>: Service Bus topics or Event Grid topics can be used to implement context-based routing.</p>
</td>
</tr>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Generic%20code.png" alt="Generic code" width="80" style="width: 80px;"></strong></td>
<td width="644">
<p><strong>Detour toggle</strong>: Azure Storage or any other persistence layer could be used to toggle the detour.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="81" style="width: 81px;"></strong></p>
<p><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="81" style="width: 81px;"></p>
</td>
<td width="644">
<p><strong>Context update</strong>: Logic Apps or Azure Functions can change the message context by updating the message header property used for routing based on the detour toggle state.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="wire Tap">Wire Tap</h2>
<p>When all messages that travel across a channel must be inspected for monitoring or troubleshooting, a <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/WireTap.html" rel="noopener" target="_blank"><strong>Wire Tap</strong></a> can be implemented so a copy of all messages is sent to a secondary channel.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></td>
<td width="644">
<p>When using Service Bus topics, an additional subscription could be implemented so that a <a href="/enterprise-integration-patterns-on-azure-endpoints#polling-consumer" rel="noopener" target="_blank"><strong>Polling Consumer</strong></a> persists all messages for monitoring or troubleshooting purposes.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></td>
<td width="644">
<p>When using Event Grid topics, an additional subscription could be implemented so that an <a href="/enterprise-integration-patterns-on-azure-endpoints#event-driven-consumer" rel="noopener" target="_blank"><strong>Event-Driven Consumer</strong></a> persists all messages for monitoring or troubleshooting purposes.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-history">Message History</h2>
<p>In some scenarios, where multiple <a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a> process a message without having an orchestrating <a href="/enterprise-integration-patterns-on-azure-routing#process-manager" rel="noopener" target="_blank"><strong>Process Manager</strong></a>, it can be difficult to troubleshoot a message without knowing what steps it has gone through. The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageHistory.html" rel="noopener" target="_blank"><strong>Message History</strong></a> pattern suggests that history metadata can be added to a message as part of the <a href="/enterprise-integration-patterns-on-azure-transformation#message-envelope" rel="noopener" target="_blank"><strong>Message Envelope</strong></a>.</p>
<p>One way to implement this pattern is to rely on the&nbsp;<a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a>&nbsp;architectural style and add the details of each pipe and filter as history metadata to the Message Envelope in each filter.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td style="width: 97px;">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="81" style="width: 81px;"></strong></p>
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="81" style="width: 81px;"></strong></p>
</td>
<td style="width: 537px;">
<p>Logic Apps and Azure Functions can act as filters.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 97px;"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></td>
<td style="width: 537px;">
<p>Service Bus can act as a pipe. The message history can be handled as a Service Bus message header or in a <a href="/enterprise-integration-patterns-on-azure-transformation#message-envelope" rel="noopener" target="_blank"><strong>Message Envelope</strong></a>.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 97px;"><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></td>
<td style="width: 537px;">
<p>Event Grid can act as a pipe. The message history can be added to the <a href="/enterprise-integration-patterns-on-azure-transformation#message-envelope" rel="noopener" target="_blank"><strong>Message Envelope</strong></a>.</p>
<p>&nbsp;</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-store">Message Store</h2>
<p>For monitoring, troubleshooting, or analysis purposes, we can use a <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageStore.html" rel="noopener" target="_blank"><strong>Message Store</strong></a> to persist relevant information of the messages that are being processed by the integration platform. Depending on the business requirements, cost, storage space, and performance implications, metadata, the full payload, or both could be stored.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>Logic Apps stateful workflows persist all inputs and outputs of every action of the workflow. When dealing with sensitive data, <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-securing-a-logic-app#obfuscate" rel="noopener" target="_blank">obfuscation</a> could be applied.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>With Azure Functions, we can implement <a href="/correlated-structured-logging-on-azure-functions" rel="noopener" target="_blank">custom structured logging</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="tracing-correlation-identifier">Tracing Correlation Identifier (*)</h2>
<p>Previously, we discussed the <a href="/enterprise-integration-patterns-on-azure-message-construction#correlation-identifier" rel="noopener" target="_blank"><strong>Correlation Identifier</strong></a> pattern, which is useful to correlate messages in a <a href="/enterprise-integration-patterns-on-azure-message-construction#request-reply" rel="noopener" target="_blank"><strong>Request-Reply</strong></a> implementation. However, when we are implementing the <strong>Message Store</strong> pattern across distributed components, it is important that we can correlate them. We can include a <strong>Tracing Correlation Identifier</strong> to be able to associate different log records and follow the processing flow of a single message across distributed <a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a>. This pattern is not discussed in the Enterprise Integration Patterns, but I believe it is quite useful and common.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>When using stateful workflow in Logic Apps, a <strong>Tracing Correlation Identifier</strong> is maintained automatically. You can add custom metadata to your tracing events as <a href="/business-activity-monitoring-on-azure-logic-apps" rel="noopener" target="_blank">described here</a>. Once you have implemented custom tracing events, then you can public custom queries and dashboards as <a href="/publishing-custom-queries-of-logic-apps-execution-logs" rel="noopener" target="_blank">described here</a>.</p>
</td>
</tr>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>With Azure Functions and Application Insights, we can implement <a href="/correlated-structured-logging-on-azure-functions" rel="noopener" target="_blank">correlated structured logging</a> or rely on the automatic correlation of Application Insights and leverage its timeline view.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-resubmission">Message Resubmission (*)</h2>
<p>Previously, we described the <a href="/enterprise-integration-patterns-on-azure-messaging-channels#dead-letter-channel" rel="noopener" target="_blank"><strong>Dead Letter Channel</strong></a> pattern, which provides a graceful way to remove and troubleshoot messages from the queue or topic subscription when these could not be delivered successfully due to issues not intrinsic to the message. When the reason for dead lettering was transient, these messages could be resubmitted into the main queue or topic for a retry. This pattern is not described in the Enterprise Integration Patterns book, however, it is quite common in integration solutions.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Generic%20code.png" alt="Generic code" width="80" style="width: 80px;"></strong></td>
<td width="644">
<p>Regardless of the <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank"><strong>Messaging Channel</strong></a> in use, explore the option of resending the message from the source when possible. With this approach, the dead-lettered message must then be removed as a separate operation.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></td>
<td width="644">
<p>When using Service Bus as the <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank"><strong>Messaging Channel</strong></a>, we can use different tools for resubmitting a message from the dead-letter sub-queue to the main queue or topic.</p>
<p><a href="https://github.com/paolosalvatori/ServiceBusExplorer/blob/develop/docs/documentation.md" rel="noopener" target="_blank"><strong>Service Bus Explorer</strong></a> (Windows-based client)<strong>:</strong> It is a free Windows tool that allows resubmitting a message in a dead-letter queue. More information on the <a href="https://github.com/paolosalvatori/ServiceBusExplorer/wiki/Repair-and-Resubmit-Message" rel="noopener" target="_blank">documentation</a>.</p>
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/explorer" rel="noopener" target="_blank"><strong>Portal-based Service Bus Explorer</strong></a> - It supports data operations on the queues, topics, and subscriptions (and their dead letter sub-entities) from the Azure portal. A message can be read from the dead-letter queue and copied to generate a new message to the queue or topic.</p>
<p>When resubmitting a message, the considerations below must too be taken into account:</p>
<p>If duplicate detection is enabled, a new <code>MessageId</code> system property might be required depending on the duplicate detection history configuration.</p>
<p>For topics, when a message is resubmitted, all subscription filters would get a copy of the message. To avoid sending redundant messages to non targeted receivers, the appropriate idempotence mechanisms or message metadata and filters must be in place.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="repair-and-resubmit (*)">Repair and Resubmit (*)</h2>
<p>When a message has been dropped to the <a href="/enterprise-integration-patterns-on-azure-messaging-channels#invalid-message-channel" rel="noopener" target="_blank"><strong>Invalid Message Channel</strong></a>, it means that it cannot be processed as is; it requires some updates before it is being resubmitted. If the message cannot be fixed on the source system and resent, we need to explore the option of repairing and resubmitting the message on the integration platform. This pattern is not discussed in the Enterprise Integration Patterns, but worth exploring.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>When using Service Bus as the <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank"><strong>Messaging Channel</strong></a>, we can use different tools for repairing and resubmitting a message in the dead-letter sub-queue.</p>
<p><a href="https://github.com/paolosalvatori/ServiceBusExplorer/blob/develop/docs/documentation.md" rel="noopener" target="_blank"><strong>Service Bus Explorer</strong></a> (Windows-based client): As discussed above, this tool allows resubmitting a message in the dead-letter queue. Once the message has been selected, the message body, system properties, and custom properties can be edited before being resubmitted. More information on the <a href="https://github.com/paolosalvatori/ServiceBusExplorer/wiki/Repair-and-Resubmit-Message" rel="noopener" target="_blank">documentation</a>.</p>
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/explorer" rel="noopener" target="_blank"><strong>Portal-based Service Bus Explorer</strong></a> – As described above, it allows reading messages from the dead-letter queue and copy them to generate and submit a new message to the queue or topic.</p>
<p>See the considerations in the previous section when resubmitting messages.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="smart-proxy">Smart Proxy</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/SmartProxy.html" rel="noopener" target="_blank"><strong>Smart Proxy</strong></a> pattern describes an implementation of the <a href="/enterprise-integration-patterns-on-azure-message-construction#request-reply" rel="noopener" target="_blank"><strong>Request-Reply</strong></a> pattern where multiple requestors can send requests and expect responses to their own independent queues, while the replier expects to have a single queue for all requests and responses. The Smart Proxy is a set of <a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a> in which a pipe is in charge of translating the <a href="/enterprise-integration-patterns-on-azure-message-construction#return-address" rel="noopener" target="_blank"><strong>Return Address</strong></a> so that the replier can work with just a pair of queues, while the requestors keep their own.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>Service Bus queues can act as pipes</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="81" style="width: 81px;"></strong></p>
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>Azure Functions or Logic Apps can act as filters. They would require a persistence layer to be able to map the <a href="/enterprise-integration-patterns-on-azure-message-construction#return-address" rel="noopener" target="_blank"><strong>Return Address</strong></a>. &nbsp;</p>
</td>
</tr>
</tbody>
</table>
<h2 id="monitoring-events-and-alerts">Monitoring Events and Alerts (*)</h2>
<p>When managing an integration platform, we often want to be notified when certain events occur. For instance, an integration processed failed, there are messages in a dead-letter sub-queue, etc. This pattern is not described in the Enterprise Integration Pattern. However, I believe it is noteworthy.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>When using Logic Apps, <a href="https://docs.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps#set-up-monitoring-alerts" rel="noopener" target="_blank">monitoring alerts</a> can be implemented.</p>
</td>
</tr>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>When using Azure Functions, monitoring alerts can be implemented using Azure Monitor or Application Insights.</p>
</td>
</tr>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>When using Service Bus, <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-metrics-azure-monitor#message-metrics" rel="noopener" target="_blank">metrics</a> can be used to establish alerts, including dead-lettered message count.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="test-message">Test Message</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/TestMessage.html" rel="noopener" target="_blank"><strong>Test Message</strong></a> pattern suggests that when we need to test integration solutions end-to-end, we can design our solution in a way that we can inject test messages in a pipe, let the test message go through the series of <a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a>, and monitor the test message at the end without impacting the receiver applications.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>Service Bus can be used as a <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank"><strong>Messaging Channel</strong></a> or pipe. Metadata must be added in the <a href="/enterprise-integration-patterns-on-azure-message-construction#message-header" rel="noopener" target="_blank"><strong>Message Header</strong></a> or <a href="/enterprise-integration-patterns-on-azure-transformation#envelope-wrapper" rel="noopener" target="_blank"><strong>Envelope Wrapper</strong></a> to signal all <a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a> when a message is a test. The last pipe should be a topic, in which the subscription for the receiver does not consider test messages and a test subscription receives only test messages.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></td>
<td width="644">
<p>Event Grid can be used as a <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank"><strong>Messaging Channel</strong></a> or pipe. Metadata must be added in the <a href="/enterprise-integration-patterns-on-azure-transformation#envelope-wrapper" rel="noopener" target="_blank"><strong>Envelope Wrapper</strong></a> to signal all <a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a> when a message is a test. The last pipe should be a topic in which the subscription for the receiver does not consider test messages and a test subscription forwards only test messages.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="channel-purger">Channel Purger</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelPurger.html" rel="noopener" target="_blank" style="font-weight: bold;">Channel Purger</a> pattern suggests that a <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank"><strong>Messaging Channel</strong></a> should provide tools or means to be purged. This might not be recommended in production environments, but it is useful when developing or testing integration solutions.</p>
<p><strong>Implementation</strong></p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed;" cellpadding="4">
<tbody>
<tr>
<td width="75"><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="81" style="width: 81px;"></strong></td>
<td width="644">
<p>When using Service Bus, the Windows-based client <a href="https://github.com/paolosalvatori/ServiceBusExplorer/blob/develop/docs/documentation.md" rel="noopener" target="_blank">Service Bus Explorer</a> can be used to purge queues, dead-letter sub-queues or topic subscriptions. Additionally, when possible a queue or topic subscription can be deleted and recreated.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></td>
<td width="644">
<p>When using Event Grid, a topic could be deleted to be purged.</p>
</td>
</tr>
</tbody>
</table>
<h2>Wrapping up</h2>
<p>In this post, we have covered the Platform Management of the Enterprise Integration Patterns and how to leverage Azure to implement them. With this post, the series comes to an end. I hope you have found the whole series useful and these posts have provided some handy information for when you are architecting or implementing integration solutions on Azure. Thanks for reading and making it this far!</p>
<p>Happy integration!</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>