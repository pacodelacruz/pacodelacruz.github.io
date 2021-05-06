---
layout: post
title: Enterprise Integration Patterns on Azure - Introduction
date: 2019-03-08 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Enterprise Integration Patterns, Azure iPaaS, Logic Apps, Service Bus, Event Grid, Azure Functions]
---
<base target="_blank"/>

![message in bottle](/assets/img/2019/03/message in bottle.jpg)

In the past, architecting and building integration solutions was a task that only specialised developers and architects were able to do. The tools and skills were considered niche and not every developer was able to build a manageable integration solution. Now, with the available new low-code integration platforms, you can build integration solutions with a few clicks and without much experience. However, knowing how to use these tools does not necessarily mean understanding how to apply them effectively to provide the functionality, resilience, robustness and manageability required by mission-critical enterprise distributed applications.

Fifteen years ago, [Gregor Hohpe](https://twitter.com/ghohpe) and [Bobby Woolf](https://twitter.com/bobby_woolf) published the book [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/) which documents design patterns for messaging-based integration solutions, independently from their technology implementation.

The [Azure Integration Services](/2018/02/02/microsoft-azure-ipaas-2), are a set of platforms on Azure that allow you to build Integration Solutions, including Logic Apps, API Management, Service Bus, Event Grid and complemented with Azure Functions and Azure Monitor.

In this series of posts, we'll explore the Enterprise Integration Patterns, and how they can be implemented using the Azure Integration Services. This series is designed to help solution architects, system integrators, and application developers to think in advanced of the challenges they need to solve so that they can design and connect applications leveraging these patterns while using the Azure Integration Services suite.

This post is part of a series of posts outlined below:

- Introduction (this)
- [Message Construction](/2019/04/10/enterprise-integration-patterns-on-azure-message-construction)
- [Messaging Channels](/2019/05/09/enterprise-integration-patterns-on-azure-messaging-channels)
- [Messaging Endpoints](/2019/06/05/enterprise-integration-patterns-on-azure-endpoints)
- [Message Routing](/2020/09/09/enterprise-integration-patterns-on-azure-routing)
- [Message Transformation](/2020/10/07/enterprise-integration-patterns-on-azure-transformation)
- [Platform Management](/2020/12/10/enterprise-integration-patterns-on-azure-platform)

The remaining posts will be published in the following weeks / months. 

<h1>Introduction to Enterprise Integration Patterns</h1>
<p><img src="/assets/img/2019/04/00%20EIP%20Banner.png" alt="00 EIP Banner" width="1100"></p>
<p>Gartner has recently coined the term “<a href="https://www.gartner.com/doc/3725217/citizen-integrators-bring-application-data" rel="noopener" target="_blank">Citizen Integrator</a>” referring to business users who have started leveraging these new low-code integration tools and platforms to build integration tasks. Developers and Citizen Integrators alike can now quickly build cloud-native integration solutions with these tools. Nevertheless, they can easily get into trouble if the requirements start to get a bit more complex and they don’t do things appropriately.</p>
<p>Software Design Patterns document knowledge and expertise translated into templates that can be used to solve common challenges when designing a solution. Fifteen years ago, <a href="https://twitter.com/ghohpe">Gregor Hohpe</a> and <a href="https://twitter.com/bobby_woolf">Bobby Woolf</a> published the book <a href="https://www.enterpriseintegrationpatterns.com/">Enterprise Integration Patterns</a> which documents design patterns for messaging-based integration solutions, independently from their technology implementation. At that time, the tools were not as mature as they are today. However, having new tools does not mean that the integration challenges have changed in essence. Most of these patterns are still relevant nowadays on cloud-native solutions. Probably the main difference is that many of these patterns might be out-of-the-box features on many of the available platforms, so you don’t need to worry about their low-level implementation.</p>
<p>That book focuses on integration patterns based on asynchronous messaging. However, the authors mention that there are other three integration styles.</p>
<ul>
<li><strong>File Transfer</strong>, which is common in solution using Extract, Transform and Load (ETL) implementations.</li>
<li><strong>Shared Database</strong>, in which multiple applications use the same database.</li>
<li><strong>Remote Procedure Invocation</strong>. In this style, when an application requires information that is maintained in other application, then it requests it directly. Similarly, if an application wants to update data of another, then it makes the corresponding request. While this integration style is still very relevant, the technologies to implement it have evolved over the years, in the Microsoft world from COM to .NET Remoting, to SOAP Web Services to REST Web APIs and most recently to gRPC. Additionally, a middle layer has been introduced to minimise the coupling between requesting and serving applications. In the HTTP world, this is usually achieved with API Gateways. <a href="https://azure.microsoft.com/en-au/services/api-management/" rel="noopener" target="_blank">Azure API Management</a> is the Azure Integration Services’ offering for API mediation.</li>
<li><strong>Messaging</strong>, in which asynchronous messages are exchanged between applications in a frequent and reliable manner. Messaging solves some of the challenges of Remote Procedure Invocation, by not requiring both systems to be up and running at the same time. This integration style is the one explored in the book and the focus of this series of posts.</li>
</ul>
<p>It is worth mentioning that these authors have been working on new Messaging patterns focusing <a href="https://www.enterpriseintegrationpatterns.com/patterns/conversation/index.html" rel="noopener" target="_blank">on conversational and stateful</a> scenarios. These conversations can happen through asynchronous queue-based messaging or via direct synchronous calls.</p>
<p>If you want to build manageable, robust and resilient enterprise-grade enterprise messaging integration solutions, you must be aware of the Enterprise Integration Patterns to make sure you are making the best use of the available technologies to solve the functional and technical challenges.</p>
<h1>Azure Integration Services Overview</h1>
<p>The Azure Integration Services, is the set of services and platforms on Azure that we can leverage to develop, implement and manage enterprise-class integration solutions. The figure below, shows the different capabilities of an iPaaS, and which component of the Azure Integration Services we can use to implement them. You can find more details in <a href="/microsoft-azure-ipaas" rel="noopener" target="_blank">this post</a>.</p>
<p>&nbsp;<img src="/assets/img/2019/04/01%20Azure%20Integration%20Services.png" alt="01 Azure Integration Services" width="1104" style="width: 1104px;"></p>
<p>&nbsp;</p>
<h1>Challenges tackled by the Enterprise Integration Patterns</h1>
<p>While creating the blueprints of integrations solutions, it is not uncommon that some challenges or requirements are initially missed or that existing proven approaches are not considered. That’s why being aware and understanding the Enterprise Integration Patterns is key to successfully architect and implement enterprise-grade integration solutions.</p>
<p>The following sections describe some of these common challenges and what patterns try to address them. Challenges and patterns are grouped by the root patterns and are detailed in tables.</p>
<p>(*) There are some patterns that are not described in the book, but architects and developers are using them when building enterprise integration solutions. Those patterns are marked with an asterisk (*)</p>
<h2>How to Construct a Message?</h2>
<p><img src="/assets/img/2019/04/10%20Construction.png" alt="10 Construction" width="360" style="width: 360px;"></p>
<p>These are common questions we need to answer when defining the structure and content of our messages, some of these are detailed in the table below. The patterns that tackle these challenges are the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageConstructionIntro.html" rel="noopener" target="_blank"><strong>Message Construction </strong></a>patterns.</p>
<table>
<tbody>
<tr>
<td width="312">
<p><strong>Challenges</strong></p>
</td>
<td width="312">
<p><strong>Patterns</strong></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we define and structure my messages? What data should I include?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#document-message" rel="noopener" target="_blank">Document Message</a></p>
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#event-message" rel="noopener" target="_blank">Event Message</a></p>
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#command-message" rel="noopener" target="_blank">Command Message</a></p>
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#query-message" rel="noopener" target="_blank">Query Message (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we shape the content of our message? Should the content and structure be coupled to an application context? Or decoupled from applications and defined to cover an organisation or industry domain?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#message-model" rel="noopener" target="_blank">Message Model (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>What data should we include as metadata for messages to be routed, processed, grouped or tracked as required?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#message-header" rel="noopener" target="_blank">Message Header (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we define messages that support two-way communication between different applications?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#request-reply" rel="noopener" target="_blank">Request-Reply</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we instruct a receiver application that a response is to be sent to a particular endpoint?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#return-address" rel="noopener" target="_blank">Return Address</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>When there is a two-way communication via messaging, how can we correlate a response to the corresponding request?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#correlation-identifier" rel="noopener" target="_blank">Correlation Identifier</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we identify when messages are to be processed as part of a group and in a particular sequence?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#message-sequence" rel="noopener" target="_blank">Message Sequence</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we define a valid lifespan of a message?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#message-expiration" rel="noopener" target="_blank">Message Expiration</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we defer a message to be processed after a certain point in time?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#message-activation" rel="noopener" target="_blank">Message Activation (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we inform the receiver application the format a message is structured in? How can we support multiple versions of a message structure?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-message-construction#format-indicator" rel="noopener" target="_blank">Format indicator</a></p>
</td>
</tr>
</tbody>
</table>
<h2>How to transport the messages between applications?</h2>
<p><img src="/assets/img/2019/04/11%20Channels.png" alt="11 Channels" width="360" style="width: 360px;"></p>
<p>Once we have decided that we are going to integrate applications using asynchronous messaging, we need to define what channels to use to transport those messages. When defining the channels, we would have some questions to answer and might face some challenges described below. The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingChannelsIntro.html" rel="noopener" target="_blank"><strong>Messaging Channels</strong></a> patterns help us to define and implement this aspect of our solution.</p>
<table>
<tbody>
<tr>
<td width="312">
<p><strong>Challenges</strong></p>
</td>
<td width="312">
<p><strong>Patterns</strong></p>
</td>
</tr>
<tr>
<td width="312">
<p>Shall we implement a direct interface between the sender application and the receiver application? Or shall the sender be unaware of potential receiver applications?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#point-to-point-channel" rel="noopener" target="_blank">Point-to-Point Channel</a></p>
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#publish-subscribe-channel" rel="noopener" target="_blank">Publish-Subscribe Channel</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How is the receiver application meant to receive the message? Does it need to pull the message from the channel, or should the channel push the message to it?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#push-pull-channel" rel="noopener" target="_blank">Push-Pull Channel (*)</a></p>
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#push-push-channel" rel="noopener" target="_blank">Push-Push Channel (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we define a channel in a way that receivers know how to process a message?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#data-type-channel" rel="noopener" target="_blank">Datatype Channel</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we manage invalid messages going through the channel?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#invalid-message-channel" rel="noopener" target="_blank">Invalid Message Channel</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>Where do we store messages when they cannot longer be processed by the intended receiver?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#dead-letter-channel" rel="noopener" target="_blank">Dead-Letter Channel</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we make sure a message is delivered to the intended receiver?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#guaranteed-delivery" rel="noopener" target="_blank">Guaranteed Delivery</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>When implementing Guaranteed Delivery, how do we make sure that retries don’t have a negative impact on the intended receiver’s health or performance?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#circuit-breaker" rel="noopener" target="_blank">Circuit Breaker (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we reduce the complexity of sending messages to a particular channel?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#channel-adapter" rel="noopener" target="_blank">Channel Adapter</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we connect multiple applications so that they can communicate via messaging in a decoupled and organised manner?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#message-bus" rel="noopener" target="_blank">Message Bus</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>When using disparate Message Channels, how can we connect the two?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-messaging-channels#messaging-bridge" rel="noopener" target="_blank">Messaging Bridge</a></p>
</td>
</tr>
</tbody>
</table>
<h2>How to connect Applications to Message Channels?</h2>
<p><img src="/assets/img/2019/04/12%20Endpoints.png" alt="12 Endpoints" width="360" style="width: 360px;"></p>
<p>It is very important to understand how we are going to connect applications to the messaging channels. The following challenges are addressed by the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingEndpointsIntro.html" rel="noopener" target="_blank"><strong>Messaging Endpoints</strong></a> patterns.</p>
<table>
<tbody>
<tr>
<td width="312">
<p><strong>Challenges</strong></p>
</td>
<td width="312">
<p><strong>Patterns</strong></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we reduce the complexity of receiving or sending messages to a particular application?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#application-adapter" rel="noopener" target="_blank">Application Adapter (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we reduce the complexity of connecting a messaging channel to an application?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#messaging-gateway" rel="noopener" target="_blank">Messaging Gateway</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we map and serialise an applications domain object in the database into a meaningful independent message?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#messaging-mapper" rel="noopener" target="_blank">Messaging Mapper</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we implement transactions so that messages are not lost in the process of sending or receiving a message to or from a channel?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#transactional-client" rel="noopener" target="_blank">Transactional Client</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can messages be consumed based on the readiness of the consumer?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#polling-consumer" rel="noopener" target="_blank">Polling Consumer (Message Pull)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can messages be consumed as soon as they are available?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#event-driven-consumer" rel="noopener" target="_blank">Event-Driven Consumer (Message Push)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we process multiple messages at a time with multiples workers on the receiver application?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#competing-consumers" rel="noopener" target="_blank">Competing Consumers</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we control how many messages are processed at a given time based on the receiver requirements or capacity?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#throttled-consumer" rel="noopener" target="_blank">Throttled Consumer (*)</a></p>
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#singleton-consumer" rel="noopener" target="_blank">Singleton Consumer (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can a receiver application select which messages to consume?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#selective-consumer" rel="noopener" target="_blank">Selective Consumer</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can different consumers receive messages in a coordinated and selective way from a single channel</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#message-dispatcher" rel="noopener" target="_blank">Message Dispatcher</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can receiver applications avoid missing messages while they are not online or listening to the channel?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#durable-subscriber" rel="noopener" target="_blank">Durable Subscriber</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can a receiver application deal with duplicate or stale messages?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#idempotent-receiver" rel="noopener" target="_blank">Idempotent Receiver</a><br><a href="/enterprise-integration-patterns-on-azure-endpoints#stale-message" rel="noopener" target="_blank">Stale Message (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can sender or receiver application restrict the time when they can send or receive messages to or from the channel?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#service-window" rel="noopener" target="_blank">Service Window (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can a receiver application consume messages received via both asynchronous messaging and synchronous channels?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-endpoints#service-activator" rel="noopener" target="_blank">Service Activator</a></p>
</td>
</tr>
</tbody>
</table>
<h2>How to route messages to the intended channel or receiver application?</h2>
<p><img src="/assets/img/2019/04/13%20Routing.png" alt="13 Routing" width="360" style="width: 360px;"></p>
<p>Routing a message to the corresponding channel or the intended receiver application has some challenges that we need to solve. The table below describes some of them and the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageRoutingIntro.html" rel="noopener" target="_blank"><strong>Message Routing</strong></a> patterns that could help to address them.</p>
<table>
<tbody>
<tr>
<td width="312">
<p><strong>Challenges</strong></p>
</td>
<td width="312">
<p><strong>Patterns</strong></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we perform different complex processing steps while maintaining some level of decoupling and flexibility?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#pipes-and-filters" rel="noopener" target="_blank">Pipes and Filters</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we pass messages to different filters, components or receivers depending on a set of conditions?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing" rel="noopener">Message Router</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we implement dynamic routing based on the content or headers of a message?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#content-based-router" rel="noopener" target="_blank">Content-Based Routing</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we avoid non-relevant messages to be delivered?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#message-filter" rel="noopener" target="_blank">Message Filter</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we validate whether a message is valid?&nbsp;</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#message-validator" rel="noopener" target="_blank">Message Validation (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we maintain a dynamic configuration on the routing rules based on the receivers needs?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#dynamic-router" rel="noopener" target="_blank">Dynamic Router</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we let the message define the list of receivers?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#recipient-list" rel="noopener" target="_blank">Recipient List</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we distribute load across multiple regions or minimise the dispatch of messages to unavailable endpoints?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#load-balancer" rel="noopener" target="_blank">Load Balancer (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we process messages which are composed of multiple elements which require its own processing?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#splitter" rel="noopener" target="_blank">Splitter</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we combine different messages into a larger message?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#aggregator" rel="noopener" target="_blank">Aggregator (Batching)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>When messages are in the right order in the Messaging Channel, how can we make sure that they are processed in the correct order?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#first-in-first-out" rel="noopener" target="_blank">First-in, First-out (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we get out-of-sequence related messages back into sequence?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#resequencer" rel="noopener" target="_blank">Resequencer</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How to process a message composed of multiple elements, which require a coordinated but different handling?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#composed-message-processor" rel="noopener" target="_blank">Composed Message Processor</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How to process messages which are to be sent to multiple receivers, for each of which we need to receive and process and response?&nbsp;</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#scatter-gather" rel="noopener" target="_blank">Scatter-Gather</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we handle a message which is to be sent for processing to different steps when the sequence is defined at runtime?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#routing-slip" rel="noopener" target="_blank">Routing Slip</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we route a message through different processing steps when these are not sequential?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#process-manager" rel="noopener" target="_blank">Process Manager</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How to avoid point-to-point integrations by decoupling senders and receivers while maintaining a centralised control of the message routing?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-routing#message-broker" rel="noopener" target="_blank">Message Broker</a></p>
</td>
</tr>
</tbody>
</table>
<h2>How can we process messages when different formats are required?</h2>
<p><img src="/assets/img/2019/04/14%20Transformation.png" alt="14 Transformation" width="360" style="width: 360px;"></p>
<p>Once we know how messages are constructed and how are to be routed, we might need to transform a message from a proprietary format to a canonical format or to another proprietary format. This transformation also has some challenges, some of these are described in the table below and <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageTransformationIntro.html" rel="noopener" target="_blank"><strong>Message Transformation</strong></a> patterns that can help to alleviate these challenges are suggested.</p>
<table>
<tbody>
<tr>
<td width="312">
<p><strong>Challenges</strong></p>
</td>
<td width="312">
<p><strong>Patterns</strong></p>
</td>
</tr>
<tr>
<td width="312">
<p>How to interchange information between different systems when they all use different formats and structures?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#message-translator" rel="noopener" target="_blank">Message Translator</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we add metadata to a message body when it is required by the messaging channels or processors?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#envelope-wrapper" rel="noopener" target="_blank">Envelope Wrapper</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we satisfy a receiver application data requirements when not all the fields are available in the original system produced by the sender?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#content-enricher" rel="noopener" target="_blank">Content Enricher</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we simplify a message when the original message is too complex or large for the intended receiver application? <br>How do we remove sensitive data fields that are not intended for a receiver application?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#content-filter" rel="noopener" target="_blank">Content Filter</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we transfer large messages when the messaging channel has some size limits?</p>
<p>How do we reduce the bandwidth requirements to interchange large messages between components while making the whole message available?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#claim-check" rel="noopener" target="_blank">Claim-Check</a><br><a href="/enterprise-integration-patterns-on-azure-transformation#compression-decompression" rel="noopener" target="_blank">Compression/Decompression (*)</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How do we standardise messages that contain same business information represented in different formats?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#normaliser" rel="noopener" target="_blank">Normaliser</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can you minimise coupling between applications that interchange messages when they use different message formats or structures?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#canonical-data-model" rel="noopener" target="_blank">Canonical Data Model</a></p>
</td>
</tr>
<tr>
<td width="312">
<p>How can we secure the message content when sensitive data could be captured by a man-in-the-middle attack?</p>
</td>
<td width="312">
<p><a href="/enterprise-integration-patterns-on-azure-transformation#encryption-decryption" rel="noopener" target="_blank">Encryption/Decryption (*)</a></p>
</td>
</tr>
</tbody>
</table>
<h2>How do we manage the integration platform and solution?&nbsp;</h2>
<p><img src="/assets/img/2019/04/15%20Management.png" alt="15 Management" width="360" style="width: 360px;"></p>
<p>Operating Integrations Solutions is usually not an easy task which brings some interesting challenges that should be considered up front. The table below describes some of these challenges and the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/SystemManagementIntro.html" rel="noopener" target="_blank"><strong>Platform Management</strong></a> patterns that can be used to address them.</p>
<table style="height: 996px;">
<tbody>
<tr style="height: 46px;">
<td style="height: 46px; width: 261.025px;">
<p><strong>Challenges</strong></p>
</td>
<td style="height: 46px; width: 261.025px;">
<p><strong>Patterns</strong></p>
</td>
</tr>
<tr style="height: 113px;">
<td style="height: 113px; width: 261.025px;">
<p>How can we manage integration solutions that leverage multiple platforms and deployed across multiple regions?</p>
</td>
<td style="height: 113px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#control-bus" rel="noopener" target="_blank">Control Bus</a></p>
</td>
</tr>
<tr style="height: 135px;">
<td style="height: 135px; width: 261.025px;">
<p>How could we control the series of steps messages are to go through based on external factors to maximise the manageability and operability of the solution?</p>
</td>
<td style="height: 135px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#detour" rel="noopener" target="_blank">Detour</a></p>
</td>
</tr>
<tr style="height: 113px;">
<td style="height: 113px; width: 261.025px;">
<p>How can you inspect messages that are being processed through a particular step or messaging channel?</p>
</td>
<td style="height: 113px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#wire%20Tap" rel="noopener" target="_blank">Wire Tap&nbsp;</a></p>
</td>
</tr>
<tr style="height: 113px;">
<td style="height: 113px; width: 261.025px;">
<p>How can you trace and analyse messages that are being processed across multiple service and components?</p>
</td>
<td style="height: 113px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#message-history" rel="noopener" target="_blank">Message History</a> / <a href="/enterprise-integration-patterns-on-azure-platform#message-store" rel="noopener" target="_blank">Message Store</a></p>
</td>
</tr>
<tr style="height: 113px;">
<td style="height: 113px; width: 261.025px;">
<p>How can you be aware of messages that could not be processed, and potentially fix them, to then resubmit them?</p>
</td>
<td style="height: 113px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#message-resubmission" rel="noopener" target="_blank">Message Resubmission (*)</a> / <a href="/enterprise-integration-patterns-on-azure-platform#repair-and-resubmit" rel="noopener" target="_blank">Repair and Resubmit (*)</a></p>
</td>
</tr>
<tr style="height: 91px;">
<td style="height: 91px; width: 261.025px;">
<p>How can you track that an expected reply message has been received for a particular request?</p>
</td>
<td style="height: 91px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#smart-proxy" rel="noopener" target="_blank">Smart Proxy</a></p>
</td>
</tr>
<tr style="height: 113px;">
<td style="height: 113px; width: 261.025px;">
<p>How can you test the health of a messaging solution end-to-end in production without unexpected side effects?</p>
</td>
<td style="height: 113px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#test-message" rel="noopener" target="_blank">Test Message</a></p>
</td>
</tr>
<tr style="height: 68px;">
<td style="height: 68px; width: 261.025px;">
<p>How can we remove stale messages from a channel?</p>
</td>
<td style="height: 68px; width: 261.025px;">
<p><a href="/enterprise-integration-patterns-on-azure-platform#channel-purger" rel="noopener" target="_blank">Channel Purger</a></p>
</td>
</tr>
<tr style="height: 91px;">
<td style="height: 91px; width: 261.025px;">
<p>How can we get notified when the platform is not performing as expected?</p>
</td>
<td style="height: 91px; width: 261.025px;"><a href="/enterprise-integration-patterns-on-azure-platform#monitoring-events-and-alerts" rel="noopener" target="_blank">Monitoring Events and Alerts (*)</a></td>
</tr>
</tbody>
</table>
<h1>Wrapping Up</h1>
<p>In this post I have discussed many of the different challenges we can face when architecting and implementing enterprise messaging-based integration solutions and the integration patterns which offer us documented knowledge and templates that can be leveraged to face these challenges. These patterns are technology agnostic and were originally defined more than 15 years ago; however, given that the challenges we face today are very similar, if not the same, to those described at that time, these patterns are still very relevant in messaging-based distributed solutions.</p>
<p>I also gave a very brief introduction of the Azure Integration Services. In the following posts of this series, I’ll cover these enterprise integration patterns in more detail and briefly describe how to implement them using the Azure Integration Services suite to build cloud-native integration. Stay tuned for the following posts of this series!&nbsp;</p>


<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
