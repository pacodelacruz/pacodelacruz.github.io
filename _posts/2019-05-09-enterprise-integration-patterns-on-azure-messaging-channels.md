---
layout: post
title: Messaging Channels - Enterprise Integration Patterns on Azure
date: 2019-05-09 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Enterprise Integration Patterns, Azure iPaaS, Logic Apps, Service Bus, Event Grid, Azure Functions]
---
<p><img src="/assets/img/2019/04/Channel3.jpeg" alt="Channel 3" width="1920" style="width: 1920px;"></p>
<p>In the <u><a href="/enterprise-integration-patterns-on-azure-message-construction" rel="noopener" target="_blank">previous post</a> </u>of the series, I covered how application data are to be serialised and packaged into messages so they can be transmitted between applications. In this post, I’ll describe the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageChannel.html" rel="noopener" target="_blank"><strong>Messaging Channels</strong></a> patterns, which focus on solving the challenges of transmitting messages from a sender application to the intended receiver applications; and how these patterns can be implemented on the Azure Integration Services.&nbsp;<span>Those marked with an asterisk (*) are those which are not described&nbsp;</span><a href="https://www.enterpriseintegrationpatterns.com/books1.html" rel="noopener" target="_blank">in the original book</a><span>, but I suggest to consider.</span></p>
<!--more-->
<p><strong>Messaging Channels</strong> can be defined statically or dynamically. Depending on different factors, different types of channels can be used. The different channels are described in the following sections.&nbsp;</p>
<p>This is a part of a series describing how to implement the Enterprise Integration Patterns using the Azure Integration Services:</p>
<ol>
<li><a href="/enterprise-integration-patterns-on-azure-intro" rel=" noopener">Introduction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-message-construction">Message Construction</a></li>
<li>Messaging Channels (this)</li>
<li><a href="/enterprise-integration-patterns-on-azure-endpoints" rel="noopener" target="_blank">Messaging Endpoints</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-routing" rel="noopener" target="_blank">Message Routing</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-transformation" rel="noopener">Message Transformation</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-platform" rel="noopener" target="_blank">Platform Management</a></li>
</ol>
<p>The remaining posts will be published in the following weeks/months.</p>
<p>The patterns covered in this article are listed below.&nbsp;</p>
<ul>
<li><a href="#point-to-point-channel" rel=" noopener">Point-to-Point Channel</a></li>
<li><a href="#publish-subscribe-channel" rel=" noopener">Publish-Subscribe Channel</a></li>
<li><a href="#push-pull-channel" rel=" noopener">Push-Pull Channel (*)</a></li>
<li><a href="#push-push-channel" rel=" noopener">Push-Push Channel (*)</a></li>
<li><a href="#data-type-channel" rel=" noopener">Datatype Channel</a></li>
<li><a href="#invalid-message-channel" rel=" noopener">Invalid Message Channel</a></li>
<li><a href="#dead-letter-channel" rel=" noopener">Dead Letter Channel</a></li>
<li><a href="#guaranteed-delivery" rel=" noopener">Guaranteed Delivery</a></li>
<li><a href="#circuit-breaker" rel=" noopener">Circuit Breaker (*)</a></li>
<li><a href="#channel-adapter" rel=" noopener">Channel Adapter</a></li>
<li><a href="#messaging-bridge" rel=" noopener">Messaging Bridge</a></li>
<li><a href="#message-bus" rel=" noopener">Message Bus</a></li>
</ul>
<h2 id="point-to-point-channel">Point-to-Point Channel</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/PointToPointChannel.html" rel="noopener" target="_blank"><strong>Point-to-Point Channel</strong></a> is utilised when the sender is aware of the one receiver that is meant to receive and process the message. It’s common that <strong>Command Messages</strong> or <strong>Query Messages</strong>, which are tightly coupled to the receiver system require a <strong>Point-to-Point Channel</strong>.</p>
<p><strong>Implementation</strong>:</p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="633">
<p>A <strong>Point-to-Point</strong> channel can be implemented using <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions#queues" rel="noopener" target="_blank">Azure Service Bus Queues</a>. A Queue is meant to have only one consumer. Thus it’s common that the sender application that drops the message into the queue is aware of the receiver.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="publish-subscribe-channel">Publish-Subscribe Channel</h2>
<p>While the <strong>Point-to-Point Channel</strong> can be used to send messages to only one consumer, the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html" rel="noopener" target="_blank"><strong>Publish-Subscribe Channel</strong></a> allows to send one message to all available interested receivers (subscribers). In this pattern, the sender does not need to be aware of who is subscribing to the messages in the channel or if there are any active subscribers. The <strong>Publish-Subscribe Channel</strong> abstracts that from the sender. However, the channel must be aware of the subscribers, so that it can create a copy of every message for each of the active subscribers.</p>
<p>&nbsp;<strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="633">
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions#topics-and-subscriptions" rel="noopener" target="_blank">Azure Service Bus Topics and Subscriptions</a> can be used as a Publish-Subscribe Channel to transmit <strong>Document Messages</strong>. The sender drops the message into the topic, and a subscription is created for each subscriber.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="633">
<p>For <strong>Event Messages</strong>, <a href="https://docs.microsoft.com/en-us/azure/event-grid/concepts#topics" rel="noopener" target="_blank">Azure Event Grid topics</a> can be used. Similarly, a subscription is created for each interested receiver.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="push-pull-channel">Push-Pull Channel (*)</h2>
<p>A <strong>Push-Pull Channel (*)</strong> is a messaging channel which cannot push messages to the intended receiver applications. Thus, it is up to the receiver application to pull the messages from the channel. This type of channel fits well with the <strong>Polling Consumer</strong>. This pattern is not originally described in the book, but I believe it is important to be aware of this type of channels when designing message-based enterprise integration solutions.</p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="633">
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions" rel="noopener" target="_blank">Azure Service Bus Queues and Topics</a> require the receiver applications to pull messages from them.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="push-push-channel">Push-Push Channel (*)</h2>
<p>A <strong>Push-Push Channel (*)</strong> is a messaging channel which is able to push messages to the intended receiver applications. Thus, the receiver application must be able to receive the messages and be available. This <strong>Messaging Channel</strong> pattern fits well with the <strong>Event-Driven Consumer</strong>. Additionally, <strong>Push-Push Channels</strong> are ideal for <strong>Event Messages</strong>. This pattern is not originally described in the book, but I believe it is important to be aware of it when designing message-based enterprise integration solutions.</p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></p>
</td>
<td width="633">
<p><a href="https://azure.microsoft.com/en-au/services/event-grid/" rel="noopener" target="_blank">Azure Event Grid</a> is a <strong>Push-Push Channel</strong> ideal for <strong>Event Messages</strong>. Given that Event Grid pushes the messages to the intended receiver application, the application must be available. To support transient failures and solution resiliency, Event Grid offers configurable retries and dead-lettering.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="data-type-channel">Datatype Channel</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DatatypeChannel.html" rel="noopener" target="_blank"><strong>Datatype Channel</strong></a> suggests that, while designing and implementing our channels, we should aim to have one channel per Message Type. So that the receiver knows how to process the message without complex rules.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p>On Service Bus, it is a good practice to use separate queues or topics for different message types (datatypes).</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81"></strong></p>
</td>
<td width="633">
<p>On Event Grid, it is recommended to have different topics for different event types (data types).</p>
</td>
</tr>
</tbody>
</table>
<h2 id="invalid-message-channel">Invalid Message Channel</h2>
<p>When implementing messaging systems, we shouldn’t expect messages to be valid all the time. There can be many different reasons why a message in a <strong>Messaging Channel</strong> is not valid; from missing required values, to invalid values, to an invalid message structure, to invalid message types. In all these cases, there is no point on retrying the processing of the message, as it will keep failing the validations. The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/InvalidMessageChannel.html" rel="noopener" target="_blank"><strong>Invalid Message Channel</strong></a> is a channel to which a messaging channel or the receiver application can deliver those invalid messages in a graceful manner without losing them. &nbsp;</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="633">
<p>In Service Bus, there is a <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues" rel="noopener" target="_blank">Dead-Letter sub-queue</a> under queues or subscriptions, but not a separate Invalid Message sub-queue. However, we can make use of the <em><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues" rel="noopener" target="_blank">DeadLetterReason</a></em> and <em><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues" rel="noopener" target="_blank">DeadLetterErrorDescription</a></em> fields to specify the cause of sending the message to the sub-queue. Given that Service Bus does not provide <strong>Message Validation (*), </strong>this must occur on the receiver application, and that application would be responsible to move the message to the corresponding sub-queue while specifying the reason.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81"></strong></p>
</td>
<td width="633">
<p>When Event-Grid tries to deliver an event and receives an Http 400 (Bad Request) or an Http 413 (Request Entity Too Large) response code from the receiver, it <a href="https://docs.microsoft.com/en-us/azure/event-grid/delivery-and-retry#dead-letter-events" rel="noopener" target="_blank">immediately sends the event to the dead-letter endpoint</a>. The <strong>Message Validation (*)</strong> must occur on the receiver application, and that application must return the corresponding Http status code to Event Grid. At the time of writing, Event Grid only supports to write invalid messages to a blob, which cannot be considered a proper messaging channel. So it is up to the solution or the administrator to pull those events to resolve deliveries.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="dead-letter-channel">Dead Letter Channel</h2>
<p>In the previous pattern, we described how to handle messages that cannot be processed when the message is invalid. But, what if the message is valid but cannot be delivered at all to the intended receiver? This could happen for various reasons, from the receiver being unavailable, to a communication failure to the receiver, to not being able to identify the receiver, to a message being expired. When the channel cannot deliver the message, there should be a graceful way to remove the message from the queue or topic subscription and leave it somewhere else for a different type of processing or troubleshooting. This is the purpose of a <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DeadLetterChannel.html" rel="noopener" target="_blank"><strong>Dead Letter Channel</strong></a>.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="633">
<p>As mentioned above, Azure Service Bus provides a dead letter sub-queue and sub-subscriptions so messages that cannot be delivered can be moved there. Messages can also be moved to the dead letter if their <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues#exceeding-timetolive" rel="noopener" target="_blank"><em>TimeToLive</em> has exceeded</a> or when <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues#exceeding-maxdeliverycount" rel="noopener" target="_blank">all retries (<em>MaxDeliveryCount</em>) have been exhausted</a>.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81"></strong></p>
</td>
<td width="633">
<p>Event Grid supports holding events that can’t be delivered to the receiver application to a <a href="https://docs.microsoft.com/en-us/azure/event-grid/manage-event-delivery#set-dead-letter-location" rel="noopener" target="_blank">dead letter location</a>. At the time of writing, Event Grid only supports to write invalid messages to a blob, which cannot be considered a messaging channel. So it is up to the solution or the administrator to pull those events to resolve deliveries.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="guaranteed-delivery">Guaranteed Delivery</h2>
<p>Asynchronous messaging allows decoupling in time the sender from the receivers. When the receiver is not available or reachable through the network, the messaging system can store the message and retry delivering the message until the receiver is reachable or available. To do so, the messaging channel must not only have the message in memory but persist it on disk so the message can survive a crash in the channel. This behaviour is described by the <strong><a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/GuaranteedMessaging.html" rel="noopener" target="_blank">Guaranteed Delivery</a></strong> pattern.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81"></strong></p>
</td>
<td width="633">
<p>Event Grid <a href="https://docs.microsoft.com/en-us/azure/event-grid/delivery-and-retry#retry-schedule-and-duration">supports retries</a> to delivery events to the receiver applications. It implements exponential back off policy to avoid overwhelming unhealthy endpoints or minimise traffic when the endpoint is down for a long period. However, at the time of writing, it only retries up to 24 hours.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="633">
<p>Logic Apps and Service Bus can be used together to implement advanced options for guaranteed delivery. A Logic App workflow can be in charge of delivering the message, while Service Bus can persist the message until it is successfully delivered.</p>
<p>On Logic Apps a <em><a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-create-api-servicebus#add-trigger-or-action" rel="noopener" target="_blank">Peek-Lock</a></em> can be implemented on Service Bus messages. This means that a message is kept in the queue but locked until one of the following conditions:&nbsp;</p>
<ul>
<li>The Logic App workflow completes the messages if processed successfully,&nbsp;</li>
<li>The Logic App workflow abandons the message if a failure occurs, or</li>
<li>The lock expires.</li>
</ul>
<p>Currently, the <em>LockTime</em> can be set to up to five minutes. That means, that the workflow has up to 5 minutes to either complete the message or when more time is required, <em><a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync?view=azure-dotnet#Microsoft_Azure_ServiceBus_Core_MessageReceiver_RenewLockAsync_System_String_" rel=" noopener">Renew</a></em> the Lock. Once the message is abandoned or the lock expires, the message is again available on the queue or topic subscription and any other&nbsp;instance can process it. Every time a lock of a message is released, the <em>DeliveryCount</em> is increased by one, and the message will remain valid until it reaches the <em>MaxDeliveryCount</em> or its <em><a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.servicebus.message.expiresatutc?view=azure-dotnet" rel="noopener" target="_blank">ExpiresAtUtc</a></em><strong>. </strong></p>
<p>You can additionally, implement a retry policy on a Logic App action to send the message to the intended receiver. That retry policy has to consider the <em>LockTime</em>. For instance, if a send action times out after 2 minutes, you could only retry twice with an interval of less than a minute or implement a <em>RenewLock </em>between retries.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p>A similar implementation to the one described above could be done using <strong>Azure Functions</strong>. An advantage that Azure Functions offers when using the Service Bus Trigger binding, is that the message lock can be renewed while the Function is being executed and the message is not completed.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="circuit-breaker">Circuit Breaker (*)</h2>
<p>The <strong><a href="https://docs.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker" rel="noopener" target="_blank">Circuit Breaker</a></strong> pattern is not described in the Enterprise Integration Patterns Book, but made popular by Michael Nygard in his book <a href="https://pragprog.com/book/mnee/release-it" rel="noopener" target="_blank">Release It!</a> This pattern can prevent overwhelming downstream systems with messages that are likely to fail due to the unavailability or unhealthy status of the downstream system. By implementing this pattern, the solution should be able to detect failures on the receiver&nbsp;application&nbsp;and stop sending messages when they are unlikely to be processed successfully. At the same time, the solution should detect when the fault has been resolved so that it can resume sending the queued messages.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80"></strong></p>
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80"></strong></p>
</td>
<td width="633">
<p>This pattern is not a built-in capability of any of the Azure Integration Services. However, <a href="https://twitter.com/jeffhollan">Jeff Hollan</a>, Program Manager of the Azure Functions team, has <a href="https://hackernoon.com/reliable-event-processing-in-azure-functions-37054dc2d0fc">described a way to implement this pattern</a> using Logic Apps and Azure Functions. The solution&nbsp;described in his post relies on Redis cache as a shared state across all instances (<strong>Competing Consumers</strong>) processing the messages, and a Logic App as a <strong>Process Manager</strong> that can manage the state of the circuit (open or closed).</p>
<p>The solution described uses Event Hubs, but it can easily be implemented with Azure Service Bus.&nbsp;</p>
<p>&nbsp;</p>
</td>
</tr>
</tbody>
</table>
<h2 id="channel-adapter">Channel Adapter</h2>
<p>A <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelAdapter.html" rel="noopener" target="_blank"><strong>Channel Adapter</strong></a> is a simplified interface to the <strong>Messaging Channel</strong> that can be used by sender or receiver applications to connect to the messaging system.</p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80"></strong></p>
<p>&nbsp;</p>
</td>
<td width="633">
<p>Channel Adapters are meant to connect a particular application with the messaging system. The best way to do it in the Azure Integration Services when there is a Logic App&nbsp;<a href="https://docs.microsoft.com/en-us/azure/connectors/apis-list" rel=" noopener"><strong>Application Adapter</strong></a>&nbsp;(connector) available for the required application is to implement a Logic App that connects the application to Service Bus or Event Grid using the connectors.&nbsp;</p>
</td>
</tr>
<tr>
<td width="74"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80"></td>
<td width="633">
<p>Service Bus provides different client libraries for different programming languages, like <a href="https://docs.microsoft.com/en-us/dotnet/api/overview/azure/service-bus?view=azure-dotnet">.NET</a>, <a href="https://github.com/Azure/azure-service-bus-java">Java</a>, <a href="https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/">Javascript</a>, <a href="https://github.com/Azure/azure-service-bus-go">Go</a>, and <a href="https://docs.microsoft.com/en-us/python/api/overview/azure/servicebus?view=azure-python">Python</a>. These can be used when we can modify the applications participating in our solution using any of these programming languages.&nbsp;</p>
<p>Service Bus also provides <a href="https://community.dynamics.com/365/b/ajitpatra365crm/archive/2018/09/26/azure-service-bus-queue-integration-with-d365-part-1">integration with D365</a></p>
</td>
</tr>
<tr>
<td width="74"><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81"></td>
<td width="633">
<p>Event Grid also provides different client libraries, like <a href="https://docs.microsoft.com/en-us/azure/event-grid/sdk-overview#data-plane-sdks">.NET, Go, Java, Node, Python and Ruby</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="messaging-bridge">Messaging Bridge</h2>
<p>In some scenarios, enterprises use more than one messaging system in the same solution. A <strong><a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingBridge.html" rel=" noopener">Messaging Bridge</a></strong> can connect different messages channels reliably to move the messages from one channel to the other.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80"></strong></p>
</td>
<td width="633">
<p>Some companies use Azure Service Bus as a <strong>Messaging Channel</strong> on Azure and <a href="https://www.ibm.com/au-en/products/mq" rel="noopener" target="_blank">IBM MQ</a> on premises. Logic Apps provide a connector for both IBM MQ and Service Bus. You can use a Logic Apps and these two connectors to create a messaging bridge and connect both messaging channels.</p>
<p>Other queuing <strong>Messaging Channels, </strong>such as Apache ActiveMQ, RabbitMQ, or Apache Qpid use a standard protocol call <a href="https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol#AMQP_1.0_broker_implementations" rel="noopener" target="_blank">AMQP</a>. Unfortunately, at the time of writing, Logic Apps does offer an AMQP connector which could bridge from one AMQP broker to another or from Service Bus to an AMQP broker. If you would like this connector to be available, you can <a href="https://feedback.azure.com/forums/287593-logic-apps/suggestions/17093809-connector-for-rabbitmq-amqp" rel="noopener" target="_blank">upvote it here</a>.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81"></strong></p>
</td>
<td width="633">
<p>For Event Messages, you can implement events with the <a href="https://docs.microsoft.com/en-us/azure/event-grid/cloudevents-schema" rel="noopener" target="_blank">CloudEvents</a> schema and forward those events with the canonical metadata schema to other eventing Messaging Channels that support this schema and receive event messages via a http endpoint.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-bus">Message Bus</h2>
<p>The <strong><a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBus.html" rel=" noopener">Message Bus</a></strong> is a meta-pattern; which includes other messaging patterns, such as Publish-Subscribe Channels, Channel Adapter, Service Activators, Message Routers, Canonical Data Models, etc.; that acts as a middleware to connect multiple sender and receiver applications.</p>
<p>&nbsp;<strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80"></strong></p>
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80"></strong></p>
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81"></strong></p>
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td width="633">
<p>To implement this meta-pattern on Azure, we need a combination of all the Azure Integration Services, including Logic Apps, Azure Service Bus, Event Grid, Azure Functions, etc.</p>
</td>
</tr>
</tbody>
</table>
<h2>Wrapping Up</h2>
<p>In this post, I have covered how to implement the <strong>Messaging Channel</strong> patterns using the Azure Integration Services. Some of these patterns are already out-of-the-box features of the platform, and in other cases we need to create our own implementation. As mentioned previously, being aware of these patterns and the challenges they address, allow us to be much better prepared when architecting message-based enterprise integration solutions.</p>
<p>Happy integration!&nbsp;</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>