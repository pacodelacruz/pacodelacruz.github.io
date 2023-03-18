---
layout: post
title: Messaging Routing - Enterprise Integration Patterns on Azure
date: 2020-09-09 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Enterprise Integration Patterns, Azure iPaaS, Logic Apps, Service Bus, Event Grid, Azure Functions]
---

<p><img src="/assets/img/2019/04/RoutingPatterns.jpg" alt="RoutingPatterns" width="1200" style="width: 1200px;"></p>
<p>In the <a href="/enterprise-integration-patterns-on-azure-endpoints">previous post</a> of the <a href="/enterprise-integration-patterns-on-azure-intro">series on the Enterprise Integration Patterns on Azure</a>, I explored the <strong>Messaging Endpoint</strong> patterns, which abstract integration interfaces from the application internals when building messaging-based integration solutions. In this post, I will cover the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageRoutingIntro.html" rel="noopener" target="_blank"><strong>Message Routing</strong></a> patterns on Azure, which provide guidelines to decouple the source from the intended receivers by implementing message routing in the integration layer.</p>
<!--more-->
<p>This post is a part of a series describing how to implement the Enterprise Integration Patterns using the <a href="/microsoft-azure-ipaas" rel="noopener" target="_blank">Azure Integration Services</a>:</p>
<ol>
<li><a href="/enterprise-integration-patterns-on-azure-intro" rel="noopener" target="_blank">Introduction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-message-construction" rel="noopener" target="_blank">Message Construction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank">Messaging Channels</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-endpoints" rel="noopener" target="_blank">Messaging Endpoints</a></li>
<li>Message Routing (this)</li>
<li><a href="/enterprise-integration-patterns-on-azure-transformation" rel="noopener">Message Transformation</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-platform" rel="noopener" target="_blank">Platform Management</a></li>
</ol>
<p>The remaining posts will be published in the following weeks/months.</p>
<p>The patterns covered in this article are listed below.</p>
<ul>
<li><a href="#pipes-and-filters" rel=" noopener">Pipes and Filters</a></li>
<li><a href="#content-based-router">Content-Based Router</a></li>
<li><a href="#recipient-list">Recipient List</a></li>
<li><a href="#message-filter">Message Filter</a></li>
<li><a href="#message-validator">Message Validator (*)</a></li>
<li><a href="#dynamic-router">Dynamic Router</a></li>
<li><a href="#load-balancer">Load Balancer (*)</a></li>
<li><a href="#splitter">Splitter</a></li>
<li><a href="#aggregator">Aggregator (Batching)</a></li>
<li><a href="#first-in-first-out">First-in, First-out (*)</a></li>
<li><a href="#resequencer">Re-sequencer</a></li>
<li><a href="#composed-message-processor">Composed Message Processor</a></li>
<li><a href="#scatter-gather">Scatter-Gather</a></li>
<li><a href="#routing-slip">Routing Slip</a></li>
<li><a href="#process-manager">Process Manager</a></li>
<li><a href="#message-broker">Message Broker</a></li>
</ul>
<h2 id="pipes-and-filters">Pipes and Filters</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html" rel="noopener" target="_blank"><strong>Pipes and Filters</strong></a> is an architectural style that allows us to divide a large process into a sequence of smaller and independent steps. Each step is called a <em>filter</em> and they are connected using pipes<em>.</em></p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>Service Bus or Event Grid can act as pipes.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps and Azure Functions can act as filters.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="content-based-router">Content-Based Router</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/ContentBasedRouter.html" rel="noopener" target="_blank"><strong>Content-Based Router</strong></a> pattern suggests that a router is in charge of routing messages into the corresponding channels based on their content. It can be used as part of the <a href="/enterprise-integration-patterns-on-azure-messaging-channels#publish-subscribe-channel" rel="noopener" target="_blank"><strong>Publish-Subscribe Channel</strong></a> pattern.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions#topics-and-subscriptions" rel="noopener" target="_blank">Azure Service Bus topics and subscriptions</a> can be used as a <a href="/enterprise-integration-patterns-on-azure-messaging-channels#push-pull-channel" rel="noopener" target="_blank"><strong>Push-Pull Channel</strong></a> to route messages based on their <a href="/enterprise-integration-patterns-on-azure-message-construction#message-header" rel="noopener" target="_blank"><strong>Message Header</strong> (*)</a>. Messages put into a topic can be routed to zero, one or more topic subscriptions. Different <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/topic-filters" rel="noopener" target="_blank">types of filters</a> can be used for routing.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p><a href="https://docs.microsoft.com/en-us/azure/event-grid/concepts#event-subscriptions">Event Grid subscriptions</a> provide content-based routing on a <a href="/enterprise-integration-patterns-on-azure-messaging-channels#push-push-channel" rel="noopener" target="_blank"><strong>Push-Push Channel</strong></a>. Event Grid provides <a href="https://docs.microsoft.com/en-us/azure/event-grid/event-filtering" rel="noopener" target="_blank">different options for filtering</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="recipient-list">Recipient List</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/RecipientList.html" rel="noopener" target="_blank"><strong>Recipient List</strong></a> pattern is similar to the <a href="#content-based-router"><strong>Content-Base Router</strong></a> pattern. However, instead of sending the message to a single recipient, the message is routed to multiple receivers. This recipient list is defined dynamically based on rules and the content of the message.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions#topics-and-subscriptions" rel="noopener" target="_blank">Azure Service Bus Topics and Subscriptions</a> can be used as a <a href="/enterprise-integration-patterns-on-azure-messaging-channels#push-pull-channel" rel="noopener" target="_blank"><strong>Push-Pull Channel</strong></a> to route messages based on their <a href="/enterprise-integration-patterns-on-azure-message-construction#message-header" rel="noopener" target="_blank"><strong>Message Header</strong> (*)</a>. The list of subscriptions that get the message is defined dynamically at run time based on filters.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p><a href="https://docs.microsoft.com/en-us/azure/event-grid/concepts#event-subscriptions">Event Grid Subscriptions</a> provide content-based routing on a <a href="/enterprise-integration-patterns-on-azure-messaging-channels#push-push-channel"><strong>Push-Push Channel</strong></a>. The list of subscriptions that get the message is defined dynamically at run time based on filters.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-filter">Message Filter</h2>
<p>A <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/Filter.html"><strong>Message Filter</strong></a> can be implemented so that messages that do not match a receiver’s criteria are discarded for that receiver. A filter can be stateless or stateful.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>When using the <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions#topics-and-subscriptions" rel="noopener" target="_blank">Azure Service Bus topics subscriptions</a>, messages that do not match any topic subscription filters are discarded. Filters in topic subscriptions are stateless.</p>
<p>Additionally, Service Bus supports <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/duplicate-detection" rel="noopener" target="_blank">message deduplication</a>. This feature is a stateful filter.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>When utilising <a href="https://docs.microsoft.com/en-us/azure/event-grid/concepts#event-subscriptions" rel="noopener" target="_blank">Event Grid subscriptions</a>, messages that do not match any subscription filters are discarded. These filters are stateless.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-validator">Message Validator (*)</h2>
<p>A type of <a href="#message-filter"><strong>Message Filter</strong></a> is the <strong>Message Validator (*)</strong>. In many cases, we need to validate if messages have a valid structure/data, and “dead letter” those which don’t. Even though this pattern is not described in the Enterprise Integration Patterns book, it is commonly implemented in messaging-based enterprise integration solutions.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Messages can be validated in Logic Apps workflows.</p>
<ul>
<li>The <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-perform-data-operations#parse-json-action" rel="noopener" target="_blank">Parse JSON</a> action allows validation of JSON messages based on a JSON schema.</li>
<li>The <a href="https://social.technet.microsoft.com/wiki/contents/articles/52867.logic-apps-message-validation-with-xml-json-and-flat-file-schemas-part-1.aspx#The_Logic_App_XML_Validation_Action" rel="noopener" target="_blank">XML Validation action</a> allows validation of XML messages.</li>
<li>The <a href="https://blogs.msdn.microsoft.com/david_burgs_blog/2017/02/14/going-deeper-with-flat-file-schema-authoring-for-biztalk-and-azure-logic-apps-flat-file-encode-decode-action/" rel="noopener" target="_blank">Flat File Decoding</a> action allows validation of flat files.</li>
</ul>
Additional information can be found in the references below:<br>
<ul>
<li><a href="https://social.technet.microsoft.com/wiki/contents/articles/52867.logic-apps-message-validation-with-xml-json-and-flat-file-schemas-part-1.aspx#The_Logic_App_XML_Validation_Action" rel="noopener" target="_blank">Logic Apps: Message Validation with XML, JSON and Flat-File Schemas (part 1)</a></li>
<li><a href="https://social.technet.microsoft.com/wiki/contents/articles/52909.logic-apps-message-validation-with-xml-json-and-flat-file-schemas-part-2.aspx" rel="noopener" target="_blank">Logic Apps: Message Validation with XML, JSON and Flat-File Schemas (part 2)</a></li>
<li><a href="https://blogs.msdn.microsoft.com/david_burgs_blog/2017/02/14/going-deeper-with-flat-file-schema-authoring-for-biztalk-and-azure-logic-apps-flat-file-encode-decode-action/" rel="noopener" target="_blank">Going deeper with Flat File schema authoring for BizTalk and Azure Logic Apps Flat File encode-decode action</a></li>
</ul>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>With Azure Functions, we can use a <a href="https://marcroussy.com/2019/06/14/http-trigger-function-request-validation/">JSON schema to validate a payload</a> or even use <a href="https://www.tomfaltesek.com/azure-functions-input-validation/" rel="noopener" target="_blank">FluentValidation</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="dynamic-router">Dynamic Router</h2>
<p>The <strong><a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DynamicRouter.html" rel="noopener" target="_blank">D</a></strong><span> pattern</span> suggests that potential receivers can create or update routing rules at run-time based on changing conditions.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Service Bus allows you to <a href="https://docs.microsoft.com/en-us/rest/api/servicebus/rules/createorupdate" rel="noopener" target="_blank">update the topic subscription rules</a> at any time. Subscribers could potentially modify these rules based on dynamic conditions.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="load-balancer">Load Balancer (*)</h2>
<p>The <strong>Load Balancer Pattern</strong> allows you to deliver messages to one of a set of endpoints using different load balancing policies. The difference between the <strong>Load Balancer (*)</strong> and <a href="/enterprise-integration-patterns-on-azure-endpoints#competing-consumers" rel="noopener" target="_blank"><strong>Competing Consumers</strong></a> is that the load is distributed by a dispatcher and not by the consumers. This pattern is useful to distribute load across multiple regions or to minimise the dispatch of messages to unavailable endpoints. The most common load balancing policies are round robin, random, based on performance, and failover.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Service Bus provides <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-geo-dr" rel="noopener" target="_blank">customer managed failover for regional (geo) disasters</a><span>.</span></p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>Event Grid provides <a href="https://docs.microsoft.com/en-us/azure/event-grid/geo-disaster-recovery" rel="noopener" target="_blank">server-side regional failover for disaster recovery</a><span>.</span></p>
</td>
</tr>
</tbody>
</table>
<h2 id="splitter">Splitter</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/Sequencer.html" rel="noopener" target="_blank"><strong>Splitter</strong></a> pattern proposes that messages that contain multiple elements or records can be broken out into a series of individual messages so that each one is processed independently.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps allow us to split a message array into independent messages using the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#trigger-multiple-runs">SplitOn property</a>. With this approach, each array item starts a new workflow instance. An alternative is to use the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-loops#foreach-loop">ForEach loop</a> structure within a workflow.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>With Azure Functions, a batched message can easily be broken out into individual messages using custom code.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="aggregator">Aggregator (Batching)</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/Aggregator.html" rel="noopener" target="_blank"><strong>Aggregator</strong></a> pattern suggests that related messages can be grouped so they can be processed as a whole. The <strong>Aggregator</strong> requires persistent storage to collect and store individual messages and a mechanism to define how messages are combined and when they are ready to be published.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps provides a way to <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-batch-process-send-receive-messages">batch related messages and specify criteria for releasing and processing those messages</a>. Messages are aggregated or batched based on a unique key or partition and can be released based on message count, batch message size, or a schedule.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="first-in-first-out">First-in, First-out (*)</h2>
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-sessions">Service Bus sessions</a> can be used to create a <a href="/enterprise-integration-patterns-on-azure-message-construction#message-sequence"><strong>Message Sequence</strong></a> construct. The next question is, how can we make sure that the messages are processed in the intended order once they reach the receiver? The <strong>First-in, First-out (*)</strong> (FIFO) pattern intends to describe this.</p>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/">Enterprise Integration Patterns</a> book does not describe this pattern as such. It describes how to construct a <a href="/enterprise-integration-patterns-on-azure-message-construction#message-sequence"><strong>Message Sequence</strong></a> and how to re-sequence them when required (<a href="#re-sequencer"><strong>Re-sequencer</strong></a>). But when messages are in the right order in the <a href="/enterprise-integration-patterns-on-azure-messaging-channels"><strong>Messaging Channel</strong></a>, how can we make sure that they are processed in the correct order?</p>
<p>Given that the FIFO pattern has an impact on the solution throughput and complexity, alternative approaches must be considered as suggested <a href="https://toonvanhoutte.wordpress.com/2018/07/24/fifo-in-integration-scenarios/">here</a>.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-sessions#first-in-first-out-fifo-pattern" rel="noopener" target="_blank">Service Bus with Sessions</a> supports the <strong>First-in, First-out</strong> pattern thanks to its <a href="/enterprise-integration-patterns-on-azure-messaging-channels#push-pull-channel"><strong>Push-Pull</strong></a> nature. However, you need to be aware that:</p>
<ul>
<li>Parallel processing can happen when <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-partitioning" rel="noopener" target="_blank">partitions</a> or <a href="/enterprise-integration-patterns-on-azure-endpoints#competing-consumers"><strong>Competing Consumers</strong></a> exist.</li>
<li>When sessions are enabled, sequential processing is only guaranteed per session.</li>
<li>The solution must be resilient to scenarios like <a href="https://en.wikipedia.org/wiki/Head-of-line_blocking" rel="noopener" target="_blank">head-of-line blocking</a>, or when messages that are part of the sequence fail.</li>
</ul>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps together with Service Bus can be used to implement the FIFO pattern as described <a href="https://social.technet.microsoft.com/wiki/contents/articles/40255.enforcing-ordered-delivery-using-azure-logic-apps-and-service-bus.aspx">here</a>.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Azure Functions together with Service Bus can be used to implement the FIFO pattern as described <a href="https://connectedcircuits.blog/2020/03/19/ensuring-ordered-delivery-of-messages-using-azure-functions/" rel="noopener" target="_blank">here</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="resequencer">Re-sequencer</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/Resequencer.html" rel="noopener" target="_blank"><strong>Re-sequencer</strong></a> pattern might be required when First-In, First-Out processing is required, but the channel cannot provide sequential delivery. This pattern requires an internal buffer to store the out-of-sequence messages until a complete sequence is obtained. Different approaches are required for bounded and unbounded sequences. The solution must also be resilient to potential scenarios such as <a href="https://en.wikipedia.org/wiki/Head-of-line_blocking">head-of-line blocking</a> and missing or failed messages.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps or Azure Durable Functions together with a persistence layer like <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities?tabs=csharp" rel="noopener" target="_blank">Azure Functions Durable Entities</a><span>, </span>Azure Storage, or Cosmos DB could be used. Custom logic would be required to put the messages in the right sequence before they are processed or delivered to the intended receiver.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="composed-message-processor">Composed Message Processor</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DistributionAggregate.html">Composed Message Processor</a> is a composed pattern that is comprised of a <a href="#splitter"><strong>Splitter</strong></a>, a <a href="#content-based-router"><strong>Content-Based Router</strong></a>, and finally an <a href="#aggregator"><strong>Aggregator</strong></a>.</p>
<p><strong>Implementation</strong></p>
<p><em>The implementation of each of the parts of this composed pattern is described previously in this article.</em></p>
<p><span style="color: #333333; font-family: inherit; font-size: 30px; background-color: transparent;">Scatter-Gather</span></p>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/BroadcastAggregate.html"><strong>Scatter-Gather</strong></a> pattern is useful when a message must be broadcast to multiple recipients and their responses must be reaggregated into a single message.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps allows you to send requests to multiple recipients and wait for their responses using the <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-native-webhook#add-an-http-webhook-action" rel="noopener" target="_blank">Webhook action</a>, as described <a href="/correlation-identifier-pattern-on-logic-apps" rel="noopener" target="_blank">here</a>. You can define a timeout to wait for the responses. The number of recipients must be known at design time.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Azure Durable Functions allows us to broadcast requests, wait for their responses, and aggregate them as described in this <a href="/azure-durable-functions-approval-workflow-with-sendgrid" rel="noopener" target="_blank">post</a>. A dynamic set of receivers can be defined at run time.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="routing-slip">Routing Slip</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/RoutingTable.html" rel="noopener" target="_blank"><strong>Routing Slip</strong></a> pattern is useful when we need to dynamically route a message through a series of processing steps which are unknown at design time. A routing slip is defined based on the message content and business rules, and can be attached to each message at run time.</p>
<p>One way to implement this pattern is to rely on the <a href="#pipes-and-filters">Pipes and Filters</a> architectural style and add the routing slip as a message header in an initial filter. Subsequent filters would process the message and pass it to the next filter based on the header state. Each filter must update the header.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps and Azure Functions can act as filters.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Service Bus can act as a pipe. The routing slip can be a Service Bus message header.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>Event Grid can act as a pipe. The routing slip can be an event message header.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="process-manager">Process Manager</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html" rel="noopener" target="_blank"><strong>Process Manager</strong></a> pattern allows us to route a message through multiple steps which are not necessarily sequential or known at design time. The <strong>Process Manager</strong> is commonly defined via a workflow or business rules and must keep a state of the processing sequence. This pattern can be subdivided into the <a href="https://docs.microsoft.com/en-us/azure/architecture/patterns/choreography" rel="noopener" target="_blank">Orchestration and Choreography</a> patterns. The Orchestration pattern is a <strong>Process Manager</strong> that is tightly coupled to the individual steps using <a href="/enterprise-integration-patterns-on-azure-message-construction#command-message" rel="noopener" target="_blank"><strong>Command Messages</strong></a>. In the Choreography pattern, the process manager typically uses <a href="/enterprise-integration-patterns-on-azure-message-construction#event-message" rel="noopener" target="_blank"><strong>Event Messages</strong></a> to communicate with the sub-tasks so that communication is not tightly coupled.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Logic Apps provides rich workflow capabilities to implement a stateful <strong>Process Manager</strong>.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>Azure Durable Functions provide workflow-as-code capabilities to implement a stateful <strong>Process Manager</strong>.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>When temporal decoupling is required, Service Bus can serve as a channel for command messages in an orchestration implementation.</p>
</td>
</tr>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="644">
<p>Event Grid can serve as a channel for event messages in a choreography implementation.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-broker">Message Broker</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBroker.html" rel="noopener" target="_blank">Message Broker</a> pattern allows us to fully decouple the message senders and receivers while maintaining central control over the flow of all message. The <strong>Message Broker</strong> can receive messages from multiple sources for multiple destinations and determine the correct destination based on business rules.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="75">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="644">
<p>A Logic App workflow can route messages to the corresponding channel based on the message content and business rules.</p>
</td>
</tr>
</tbody>
</table>
<h2>Wrapping Up</h2>
<p>In this post, we have covered the <strong>Messaging Routing</strong> patterns and how to leverage the Azure Integration Services to implement them. Understanding these patterns allows us to consider different approaches when architecting integration solutions. Stay tuned for the next instalment covering the <strong>Message Transformation</strong> patterns.</p>
<p>Happy integration!</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>