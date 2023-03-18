---
layout: post
title: Messaging Endpoints - Enterprise Integration Patterns on Azure
date: 2019-06-05 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Enterprise Integration Patterns, Azure iPaaS, Logic Apps, Service Bus, Event Grid, Azure Functions]
---

<img src="/assets/img/2019/04/Endpoints.jpg" alt="Endpoints" width="5847" style="width: 5847px;">
<p>In the <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank">previous post</a> of the series, I described how messages can be transmitted from a sender application to intended receiver applications through <strong>Messaging Channels</strong>, and how the Azure Integration Services can be leveraged to implement these Enterprise Integration Patterns. <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageEndpoint.html"><strong>Messaging Endpoints</strong></a> are the application touch points that abstract the application internals and can be used to either receive/extract messages from a source application or send messages to a target application when building integration solutions. In this post, I describe the <strong>Messaging Endpoint</strong> patterns and how we can utilise the Azure Integration Services to implement them.</p>
<!--more-->
<p>This post is a part of a series describing how to implement the Enterprise Integration Patterns using the Azure Integration Services:</p>
<ol>
<li><a href="/enterprise-integration-patterns-on-azure-intro" rel="noopener" target="_blank">Introduction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-message-construction" rel="noopener" target="_blank">Message Construction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank">Messaging Channels</a></li>
<li>Messaging Endpoints (this)</li>
<li><a href="/enterprise-integration-patterns-on-azure-routing" rel=" noopener">Message Routing</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-transformation" rel="noopener" target="_blank">Message Transformation</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-platform" rel="noopener" target="_blank">Platform Management</a></li>
</ol>
<p>The remaining posts will be published in the following weeks/months. Those patterns highlighted with an asterisk (*) are those that are not described in the original <a href="https://www.enterpriseintegrationpatterns.com/books1.html" rel="noopener" target="_blank">Enterprise Integration Patterns book</a>, but that I suggest considering.</p>
<p><span style="background-color: #ffffff;">The patterns covered in this article are listed below.&nbsp;</span></p>
<ul>
<li><a href="#application-adapter">Application Adapter (*)</a></li>
<li><a href="#messaging-mapper">Messaging Mapper</a></li>
<li><a href="#messaging-gateway">Messaging Gateway</a></li>
<li><a href="#transactional-client">Transactional Client</a></li>
<li><a href="#polling-consumer">Polling Consumer (Message Pull)</a></li>
<li><a href="#event-driven-consumer">Event-Driven Consumer (Message Push)</a></li>
<li><a href="#competing-consumers">Competing Consumers</a></li>
<li><a href="#throttled-consumer">Throttled Consumer (*)</a></li>
<li><a href="#singleton-consumer">Singleton Consumer (*)</a></li>
<li><a href="#selective-consumer">Selective Consumer</a></li>
<li><a href="#message-dispatcher">Message Dispatcher</a></li>
<li><a href="#durable-subscriber">Durable Subscriber</a></li>
<li><a href="#idempotent-receiver">Idempotent Receiver</a></li>
<li><a href="#stale-message">Stale Message (*)</a></li>
<li><a href="#service-window">Service Window (*)</a></li>
<li><a href="#service-activator">Service Activator</a></li>
</ul>
<h2 id="application-adapter">Application Adapter (*)</h2>
<p>In my <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank">previous post</a>, we explored the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelAdapter.html"><strong>Channel Adapter</strong></a>, which abstracts the complexities of a <strong>Messaging Channel</strong>, so that applications can connect to it more easily. The Azure Integration Services offer <strong>Application Adapters (*)</strong> or <a href="https://docs.microsoft.com/en-us/azure/connectors/apis-list" rel="noopener" target="_blank">connectors</a> that abstract applications from the integration solutions so that messages can be received, pulled or delivered to different applications with no code required. This pattern is not originally described in the Enterprise Integration Patterns, however, it’s common in modern integration platforms.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>Logic Apps provide a growing <a href="https://docs.microsoft.com/en-us/connectors/" rel="noopener" target="_blank">list of connectors</a> that allow you to connect to many different applications without writing code. Some of these connectors allow applications to trigger Logic Apps workflows, other connectors enabling pulling messages from the application, and many of these connectors allow you to deliver messages to the receiver application.</p>
<p>Many of these connectors interact with the application’s exposed APIs. However, in other cases, we might need to bridge a messaging channel into a file-based protocol, due to the lack of a connector or API on the application side.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="messaging-mapper">Messaging Mapper</h2>
<p>A <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingMapper.html" rel="noopener" target="_blank"><strong>Messaging Mapper</strong></a> is in charge of serialising an object on the sender application into a message that can be sent to the <strong>Messaging Channel</strong> and vice versa on the receiver application. This message could be structured as a JSON object, as an XML message, or even as a proprietary format, such as an IDoc in SAP. A <strong>Messaging Mapper</strong> is required when the application data is not stored in a way that can be directly sent through a <strong>Messaging Channel</strong>. For instance, when the data is stored in a relational database, or in a hierarchy of objects. These data must be translated into a message that can be transmitted through a channel. In some cases, particularly in cloud-native applications where data is stored in document-based databases, this mapping might not be required.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>A <strong>Messaging Mapper</strong> is usually implemented at the application API. The Application API is aware of the Business Objects and can expose those business objects as domain-specific self-contained messages. Additionally, as mentioned in the section above, a Logic App connector (<strong>Application Adapter (*)</strong>) can abstract the complexities of the application API so that a <strong>Messaging Mapper</strong> results very easy to implement on an Enterprise Application Integration solution.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="messaging-gateway">Messaging Gateway</h2>
<p>A <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingGateway.html" rel="noopener" target="_blank"><strong>Messaging Gateway</strong></a> encapsulates internal objects and exposes domain-specific methods or messages. It usually implements the <strong>Messaging Mapper</strong> described above exposing different methods.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>Usually, this abstraction is on the application’s API, either sender or receiver. This layer exposes the application data and functionality in an abstracted way (e.g. Salesforce APIs).&nbsp;<br>Logic Apps connectors can provide an additional layer of abstraction on top of these APIs.&nbsp;</p>
</td>
</tr>
</tbody>
</table>
<h2 id="transactional-client">Transactional Client</h2>
<p>A <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/TransactionalClient.html" rel="noopener" target="_blank"><strong>Transactional Client</strong></a> is the implementation of boundaries of an atomic transaction so that messages are not lost in the process of being sent or received. The transaction scope can also go further, for instance:</p>
<ul>
<li><strong>Database to Message</strong>: so that the object is not marked as processed or deleted on the sender application’s database until it has been successfully sent to the <strong>Messaging Channel</strong>.</li>
<li><strong>Message to Database</strong>: so that the message is not deleted from the <strong>Messaging Channel</strong> until it has been committed to the target Database in the receiver application.</li>
<li><strong>Message Groups:</strong> In this case, a transaction won’t be completed until the whole set of messages are received. Otherwise, it should not be committed. This approach could be required when implementing the <strong>Message Sequence</strong></li>
<li><strong>Receive-Send Message Pairs</strong>: in this approach, a message being received cannot be completed, until a correlated message is sent. This approach might be relevant when implementing the <strong>Request-Reply</strong> pattern on the receiver application.</li>
<li><strong>Send-Receive Message Pairs</strong>: in this scenario, a message being sent cannot be completed, until a correlated message is received. This approach could be required when implementing the <strong>Request-Reply</strong> pattern on the sender application.</li>
</ul>
<p>In some cases, the sender or receiver application does not support transactions. When transactions are not supported, a <strong>Process Manager</strong> can be implemented so a compensation operation is performed. You can find more information on this pattern in the <a href="https://docs.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction" rel="noopener" target="_blank">Microsoft docs</a>.</p>
<p><strong>Implementation </strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>As described above, there are different types of transactions that can be implemented on a message-based enterprise integration solution. Depending on the type, a different implementation would be required.</p>
<ul>
<li><strong>Database to Message</strong>: this is usually implemented at the sender application API and abstracted using a Logic App connector.&nbsp;</li>
<li><strong>Message to Database</strong>: this can easily be implemented using Service Bus, a Logic App workflow, and a Logic App connector. The Logic App workflow trigger would be a <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-create-api-servicebus" rel="noopener" target="_blank">Peek-Lock on the Service Bus queue or topic subscription</a>. Later, the workflow will try to deliver the message to the receiver application. Only if the delivery is successful, the message is completed on Service Bus, otherwise, the message will be abandoned so that a new instance of the workflow can retry later. This pattern was described in detail in the <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank">previous post</a>, in the <strong>Guaranteed Delivery</strong> pattern.</li>
<li><strong>Message Groups</strong>: A Logic App workflow can be implemented to receive all messages within a session from a Service Bus queue or topic subscription and only commit the transaction if all messages have been received and processed successfully.</li>
<li><strong>Receive-Send Message Pairs: </strong>To implement this pattern, a Logic App workflow could be utilised, so a message is not completed on a queue or topic subscription until a correlated message is successfully sent to another queue or topic.</li>
<li><strong>Send-Receive Message Pairs: </strong>This pattern can be implemented on a Logic App workflow, implementing the <strong>Request-Reply</strong> pattern, either using the webhook action, or Service Bus sessions, and not committing or completing the request message, until the reply is received.</li>
</ul>
</td>
</tr>
</tbody>
</table>
<h2 id="polling-consumer">Polling Consumer (Message Pull)</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/PollingConsumer.html" rel="noopener" target="_blank"><strong>Polling Consumer</strong></a> pattern is applied when the message is pulled to be processed. This pattern can be implemented on both, the sender and the receiver application sides.</p>
<ul>
<li><strong>Polling Consumer on the Receiver Application</strong>. In this pattern, the receiver application controls when to consume a message from the <strong>Messaging Channel</strong> by checking if messages are available. This pattern fits well with the <strong>Push-Pull Channel (*).</strong></li>
<li><strong>Polling Consumer on the Sender Application</strong>. This pattern is implemented when the messages cannot be pushed from the sender application to the <strong>Messaging Channel.</strong> However, an <strong>Application Adapter (*)</strong> can poll the sender application for new messages and then send them to the corresponding <strong>Messaging Channel</strong>.</li>
</ul>
<p>Depending on the capabilities and flexibility of the application and the <strong>Channel Adapter</strong> or the <strong>Application Adapter (*)</strong>, a <strong>Polling Consumer </strong>can be stateful or stateless, as described below.</p>
<ul>
<li><strong>Stateless Polling Consumer</strong>: In this approach, the consumer cannot or does not need to keep a state for polling. For instance, a <strong>Polling Consumer</strong> that gets all available records from a table, and once a message has been put into the channel, deletes it from that table or marks it as processed. Usually, this happens within a transaction. The advantage of this approach is that it does not require the <strong>Polling Consumer</strong> to keep a state. However, a disadvantage is that it requires the sender application to keep a state concerning the integration process; which is not always possible or ideal.</li>
<li><strong>Stateful Polling Consumer</strong>: In this pattern, the consumer keeps a state for polling. For instance, keeping a polling watermark which indicates the last time a poll was performed. This way, the next poll only gets messages which were created or updated after that polling watermark. The main advantage of this approach is that it does not require the sender application to keep a status concerning to the integration process. However, it requires the <strong>Polling Consumer</strong> to keep a state. Additionally, for highly available solutions, this state must be maintained in a distributed way.</li>
</ul>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>Some Logic App adapters support the <strong>Stateless Polling Consumer</strong> pattern. For instance, the <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-create-api-servicebus#add-trigger-or-action" rel="noopener" target="_blank">Service Bus Trigger</a> is able to receive all available messages and complete them once they have been processed successfully.</p>
<p>Likewise, some adapters support the <strong>Stateful Polling Consumer</strong> pattern. The <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-create-api-sqlazure#add-sql-trigger" rel="noopener" target="_blank">SQL Connector</a>, for instance, can poll for inserts or updates on a SQL Table without requiring to keep an integration state on the source table. It does this by keeping a trigger state that contains either the <em>RecordId</em> for inserts or the <em>RowVersion</em> for updates. Additionally, the file connector can be triggered when a new file is dropped into a folder. And because the connector does not delete the files as part of the trigger step, the workflow must keep a trigger state (<em>LastModifiedDate</em> of most recently updated file), so when a new poll occurs, only new or updated files trigger the workflow.</p>
<p>The Stateful Polling Consumer pattern is described in detail in <a href="/polling-consumer-pattern-using-azure-logic-apps">this post</a>.</p>
<p>Some Logic App adapters support <strong>both the Stateful and Stateless approach</strong>. For instance, the SQL Connector could also call a stored procedure that gets records and either delete them or update them after a successful processing. Likewise, when using the File Connector, we can have a workflow which is triggered recurrently, with an action to get all available files in the folder, and once the files have been processed successfully, we can delete them from the folder; so that the next instance won’t process them again (destructive read).</p>
</td>
</tr>
</tbody>
</table>
<h2 id="event-driven-consumer">Event-Driven Consumer (Message Push)</h2>
<p>In some scenarios, the source is able to push messages when an event occurs. Similarly to the previous pattern, the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/EventDrivenConsumer.html" rel="noopener" target="_blank">Event-Driven Consumer</a> can be implemented on the sender and the receiver application.</p>
<ul>
<li><strong>Event-Driven Consumer on the Receiver Application side</strong>: This pattern can be applied when the receiver application is open to accept messages from the <strong>Messaging Channel</strong> directly without requiring to poll.</li>
<li><strong>Event-Driven Consumer on the Sender Application side: </strong>This pattern can be applied when a <strong>Messaging Channel</strong> can receive messages directly.</li>
</ul>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></td>
<td width="633">
<p>On the receiver application side, this pattern can be implemented using Event Grid. The receiver application must expose a HTTP API that can receive events</p>
</td>
</tr>
<tr>
<td width="74"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>On the sender application side, this pattern can be implemented using Event Grid or Service Bus. Both accept messages being pushed via a HTTP request or the SDK.</p>
</td>
</tr>
</tbody>
</table>
<p><strong>Polling Consumer vs Event-Driven Consumer&nbsp; </strong></p>
<p>The table below shows some differences between the <strong>Polling Consumer</strong> and the <strong>Event-Driven Consumer</strong> patterns</p>
<table>
<tbody>
<tr>
<td width="160">
<p>&nbsp;</p>
</td>
<td width="255">
<p>Polling Consumer</p>
</td>
<td width="208">
<p>Event-Driven Consumer</p>
</td>
</tr>
<tr>
<td width="160">
<p>Trigger</p>
</td>
<td width="255">
<p>The target (consumer) triggers the action by polling, usually based on a schedule.</p>
</td>
<td width="208">
<p>The source triggers the action by pushing the message when an event occurs.</p>
</td>
</tr>
<tr>
<td width="160">
<p>Durability</p>
</td>
<td width="255">
<p>Messages can be stored on the source until the target is ready to consume them.</p>
</td>
<td width="208">
<p>Messages can get lost if the intended receiver is not reachable and retries get exhausted.</p>
</td>
</tr>
<tr>
<td width="160">
<p>Timeliness</p>
</td>
<td width="255">
<p>Depending on the schedule, there could be a delay between a message is emitted and consumed.&nbsp;</p>
</td>
<td width="208">
<p>Usually, messages can be processed in near real-time.</p>
</td>
</tr>
<tr>
<td width="160">
<p>Ordering</p>
</td>
<td width="255">
<p>Under certain conditions, polling could allow in-order processing.</p>
</td>
<td width="208">
<p>In-order processing is usually not supported.</p>
</td>
</tr>
<tr>
<td width="160">
<p>Cost Effectiveness</p>
</td>
<td width="255">
<p>When trying to poll frequently to minimise delays, polling can result in a waste of resources</p>
</td>
<td width="208">
<p>Processing is only required when messages are available.</p>
</td>
</tr>
<tr>
<td width="160">
<p>Throttling or Concurrency Control</p>
</td>
<td width="255">
<p>The polling consumer can dictate the frequency and volume of messages to pull.</p>
</td>
<td width="208">
<p>Throttling without the risk of losing messages can be harder to implement.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="competing-consumers">Competing Consumers</h2>
<p>When consuming messages from a queue, having one single processing worker may result in messages being piled up on the channel, which can impact the throughput. Having <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/CompetingConsumers.html" rel="noopener" target="_blank"><strong>Competing Consumers</strong></a> processing messages in parallel from one channel allows reducing bottlenecks and increasing throughput. Having multiple consumers working in parallel might bring the risk of processing the same message multiple times. To avoid this, two approaches can be used:</p>
<ul>
<li><strong>Destructive Read</strong>: this means that once a message is grabbed by one of the consumers, the message is immediately removed from the channel. This approach is very simple, however, not very resilient. Messages could be lost if a failure occurs.</li>
<li><strong>Non</strong>-<strong>destructive Read: </strong>to minimise the risk of losing messages in the case of failures, a message can be kept locked in the channel after being grabbed, so no other consumers can take it. Then, the message can be removed once it has been successfully processed, the lock can be renewed if the processing is taking longer than the lock duration, or the message can be unlocked if a transient failure occurs so it can be processed again by any of the <strong>Competing Consumers</strong>.</li>
</ul>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>Logic Apps implements the <strong>Competing Consumer</strong> pattern with Service Bus out-of-the-box. By default, the workflow will scale-out to multiple instances to process multiple messages in parallel. Logic Apps allow you to implement a <strong>destructive read</strong> approach with the Auto-complete Service Bus Trigger, and the <strong>non-destructive read</strong> with the Peek-lock Service Bus trigger.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p>Azure Functions also implement the <strong>Competing Consumers</strong> pattern with Service Bus out-of-the-box. By default, the function will scale-out to multiple instances to process multiple messages in parallel.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="throttled-consumer">Throttled Consumer (*)</h2>
<p>In cloud-native integration solutions, having <strong>Competing Consumers</strong> is almost always the norm. On Azure Integration Services, Logic Apps or Azure Functions will promptly scale out based on the number or incoming messages. However, in some cases, we might want to have a limited number of instances of our processing workers consuming the available messages. This requirement is common when we need to avoid hammering a receiver application with more requests than this can handle. The <strong>Throttled Consumer (*)</strong> pattern is not described in the original book of Enterprise Integration Patterns; however, it can solve common challenges when implementing Enterprise Integration Solutions. You can find more information on this pattern in the <a href="https://docs.microsoft.com/en-us/azure/architecture/patterns/throttling" rel="noopener" target="_blank">Microsoft docs</a>.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>Azure Logic Apps allows you to have <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#runtime-configuration-settings" rel="noopener" target="_blank">concurrency control</a> so that you can limit them to have only a limited number of instances of a workflow running at the same time. Other messages will be waiting until the running instance completes its execution.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td width="74"><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>At the time of writing, Azure Functions does support controlling the degree of parallelism but with some limitations, as described in the following posts: <a href="https://github.com/Azure/azure-functions-host/issues/1207" rel="noopener" target="_blank">(1)</a>, <a href="https://github.com/Azure/azure-functions-host/issues/912" rel="noopener" target="_blank">(2)</a>, and <a href="https://docs.microsoft.com/en-au/azure/azure-functions/functions-app-settings#website_max_dynamic_application_scale_out" rel="noopener" target="_blank">(3)</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="singleton-consumer">Singleton Consumer (*)</h2>
<p>A type of the <strong>Throttled Consumer</strong> pattern is required when the receiver application must or can only handle one message at a time. In this case, we need to implement a <strong>Singleton Consumer (*)</strong>. The <strong>Singleton Consumer (*)&nbsp;</strong>might be required in different scenarios.</p>
<ul>
<li><strong>In-order processing</strong>: If we know that messages in a queue are sequenced and we need to process them in that order, we will need to make sure that only one worker is processing those messages at a certain time. However, relying only on the first-in first-out (FIFO) approach on queues has limitations for ordered processing. For instance, we need to make sure that partitions are aligned with the sequencing requirements, and we must be aware that the head-of-queue message can block the whole queue if its processing is taking long, especially when using retries.</li>
<li><strong>Processing limitation</strong>: Some applications can only handle one message at a time. This might not be true for cloud applications, but can be the case in legacy applications.</li>
</ul>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p>Azure Logic Apps allows you to have <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#runtime-configuration-settings" rel="noopener" target="_blank">concurrency control</a> so that you can limit them to have only one instance of a workflow running at the same time. Other messages will be waiting until the running instance completes its execution.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p>At the time of writing, Azure Functions does support controlling the degree of parallelism but with some limitations, as described in the following posts: <a href="https://github.com/Azure/azure-functions-host/issues/1207" rel="noopener" target="_blank">(1)</a>, <a href="https://github.com/Azure/azure-functions-host/issues/912" rel="noopener" target="_blank">(2)</a>, and <a href="https://docs.microsoft.com/en-au/azure/azure-functions/functions-app-settings#website_max_dynamic_application_scale_out" rel="noopener" target="_blank">(3)</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="selective-consumer">Selective Consumer</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageSelector.html" rel="noopener" target="_blank"><strong>Selective Consumer</strong></a> pattern can be implemented when the receiver application does not want to consume all messages available on the <strong>Messaging Channel</strong>. Usually, properties in the <strong>Message Header (*)</strong> are used by the <strong>Selective Consumer </strong>to determine whether the message is relevant to that consumer.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p>In Service Bus, <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/topic-filters" rel="noopener" target="_blank">Topics with Subscriptions</a> can be used. The selective rules are implemented at the subscription using <strong>Message Header</strong> properties. In a <strong>Publish-Subscriber Channel</strong> pattern, subscriptions would be used to identify the messages intended for receiver applications.</p>
</td>
</tr>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"><br></strong></p>
</td>
<td width="633">
<p>Azure Event Grid also provides topics for <strong>Event Messages</strong> that allow <a href="https://docs.microsoft.com/en-us/azure/event-grid/event-filtering" rel="noopener" target="_blank">directing messages</a> in a selective manner to different consumers.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="message-dispatcher">Message Dispatcher</h2>
<p>A <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageDispatcher.html" rel="noopener" target="_blank"><strong>Message Dispatcher</strong></a> can be required when the <strong>Publish-Subscribe Channel</strong>, <strong>Datatype Channel</strong>, or <strong>Selective Consumer</strong> patterns cannot be implemented or are not sufficient to distribute and route the messages to the intended receivers. The <strong>Message Dispatcher</strong> acts as a coordinator to implement advanced rules to send messages to the consumers.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p>When Service Bus topics subscriptions or Event Grid topics are not sufficient to direct messages to the corresponding consumers, a <strong>Message Dispatcher</strong> can be implemented as a Logic App workflow. Additionally, when there is a requirement to implement the dispatching business rules to identify the receiver in a decoupled way from the Logic App workflow, these can be defined using Liquid Templates as described <a href="/business-rules-on-azure-logic-apps-with-liquid-templates">in this post</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="durable-subscriber">Durable Subscriber</h2>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DurableSubscription.html"><strong>Durable Subscriber</strong></a> pattern allows that messages in a <strong>Messaging Channel</strong> are not lost when the intended receiver is not available, by persisting the messages.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>Service Bus provides durability so that receiver applications can pull messages when they are back online. The Message <em><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-expiration" rel="noopener" target="_blank">TimeToLive</a> </em>property is to be set accordingly.</p>
</td>
</tr>
<tr>
<td width="74"><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></td>
<td width="633">
<p>Event Grid also provides <a href="https://docs.microsoft.com/en-us/azure/event-grid/delivery-and-retry" rel="noopener" target="_blank">durability for messages</a>. It implements <strong>Guaranteed Delivery</strong> (push) but within a limit of 24 hours.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="idempotent-receiver">Idempotent Receiver</h2>
<p>The <strong>Guaranteed Delivery</strong> pattern is meant to resubmit a message when they were not delivered successfully. However, it is possible that in some scenarios the message is received by the receiver successfully, but the corresponding acknowledgement is not received or processed on the sender side. This would lead to the submission of a redundant message. Given that possibility, particularly in distributed systems when retries may happen in many different layers, the ability to discard duplicate messages is crucial. The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html" rel="noopener" target="_blank"><strong>Idempotent Receiver</strong></a> pattern allows a receiver to safely receive the same message multiple times without undesired side effects.</p>
<p>This approach requires that each message has a unique identifier and that either the <strong>Messaging Channel</strong> or the receiver application keeps a log of the identifiers of the messages previously processed; so that if a duplicate message arrives, it can be safely discarded.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74">
<p><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"><br></strong></p>
</td>
<td width="633">
<p><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/duplicate-detection" rel="noopener" target="_blank">Service Bus supports deduplication</a> based on the <em><a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.servicebus.message.messageid?view=azure-dotnet">MessageId</a> </em>property<em>.</em> The duplicate detection history can be kept for up to seven days. When enabling deduplication, we need to consider scenarios where we purposely need to resubmit a message to the channel with the same identifier, which would require us to create a new identifier for the resubmission.</p>
</td>
</tr>
<tr>
<td width="74"><img src="/assets/img/2019/04/Generic%20code.png" alt="Generic code" width="80" style="width: 80px;"></td>
<td width="633">
<p>When not using Service Bus, we would need to implement our own log and validation.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="stale-message">Stale Message (*)</h2>
<p>In many scenarios deduplication works well. However, there are other scenarios where deduplication is not enough, particularly when we are processing <strong>Document Messages</strong> that contain the full state of an entity and in-sequence processing is important, but the <strong>Messaging Channel</strong> cannot guarantee in-order delivery. Think of an Employee entity that has two updates, one after the other; and both messages have a full snapshot of the Employee entity. If we only applied deduplication and messages arrived out-of-sequence, the receiver application would process both messages in the wrong order and the final state of the employee would be out-of-date.</p>
<p>When the full state of the entity is contained in the <strong>Document Message</strong>, and the full change history is not required on the receiver application, processing every single message for a particular entity is not always required. In these scenarios, where eventual consistency per entity is preferred over the processing of every single entity event, we could implement the <strong>Stale Message</strong> (*) pattern to identify whether a message received is newer than the persisted state in the receiver application. A <strong>Stale Message (*)</strong> is a message that is received and does not contain newer information for a particular entity. This can happen because the message was received out-of-sequence, or because the message has been processed successfully previously and received again due to a redundant retry. This pattern was not described in the Enterprise Integration Patterns book, however, it can be used to solve the challenges described above.</p>
<p>I am suggesting to add this pattern to the <strong>Messaging Endpoints</strong> group, due to its relation to the <strong>Idempotent Receiver</strong> pattern; however, it could well be classified as a <strong>Message Routing</strong> pattern, given that it is also related to the <strong>Resequencer</strong> and <strong>Message Validation (*)</strong> patterns.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Generic%20code.png" alt="Generic code" width="80" style="width: 80px;"></td>
<td width="633">
<p>There is no built-in implementation of the <strong>Stale Message (*)</strong> pattern on the Azure Integration Services. It’s implementation requires</p>
<ul>
<li><strong>Document Messages </strong>to contain the full state of the entity.</li>
<li>The <strong>Entity Id </strong>and the snapshot<strong> timestamp </strong>must be part of the message. These two properties are used to validate whether the message is stale or newer.</li>
<li><strong>Eventual Consistency per entity is preferred over the entity’s full change history.</strong></li>
</ul>
<p>One important consideration is that ideally, the validation to check whether the message is stale and the persistence of the message should happen within a transaction. Otherwise, there could be a race condition where more than one message for the same entity are running in parallel; which could cause undesired side effects.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="service-window">Service Window (*)</h2>
<p>When integrating with legacy applications, sometimes we know that they are not available or reachable at certain times of the day or of the week due to maintenance, or because they are overloaded due to scheduled batch jobs. The <strong>Service Window (*)</strong> pattern allows us to configure our integration solution so that it does not try to reach the application during those periods. This helps us to avoid getting false positive alerts of unreachable endpoints or overloading even more the busy application with requests coming from the <strong>Messaging Channel</strong>. This pattern is not described in the Enterprise Integration Patterns book; however, I believe that it should be considered in some scenarios.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>Ideally, the Service Window (*) pattern should be implemented at the Application Adapter (*). However, this is not offered by Logic Apps connectors. One way to implement a <strong>Service Window (*)</strong> would be to trigger the Logic App in charge of connecting to the application using the <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-native-recurrence#add-recurrence-trigger" rel="noopener" target="_blank">Recurrence Trigger</a> and defining the days and the hours the workflow is meant to run. However, this approach has limitations. For instance, we wouldn’t be able to use the many other built-in triggers.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="service-activator">Service Activator</h2>
<p>In some scenarios, we want to expose the same service on an application not only via a synchronous request but also via messaging in an asynchronous manner. The <strong>Service Activator</strong> pattern describes that we can expose those synchronous services via a <strong>Messaging Channel</strong>.</p>
<p><strong>Implementation</strong></p>
<table>
<tbody>
<tr>
<td width="74"><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></td>
<td width="633">
<p>We can wrap Http endpoints using a Logic App that is triggered by messages coming from Service Bus. If the service is two-way, we can then return the response with a <strong>Correlation Identifier</strong> to the original requestor via another Service Bus queue or topic.</p>
</td>
</tr>
</tbody>
</table>
<h2>Wrapping up</h2>
<p>In this post, we have covered the <strong>Messaging Endpoints</strong> patterns and how to leverage the Azure Integration Services to implement them. As we have discussed previously, the technology is evolving and some of these patterns are built-in features of the Azure service offerings. However, others require a custom implementation on top of them. Understanding the <strong>Messaging Endpoint</strong> patterns allows us to consider different approaches to connect to the applications when architecting integration solutions. I hope you have found this post useful. Stay tuned to the next instalment of the series about the <strong>Message</strong> <strong>Routing</strong> Patterns.</p>
<p>Happy integration!&nbsp;</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>