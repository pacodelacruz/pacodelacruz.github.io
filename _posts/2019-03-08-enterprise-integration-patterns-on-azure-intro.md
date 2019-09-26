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
- Message Routing
- Message Transformation
- Platform Management

The remaining posts will be published in the following weeks / months. 

Introduction to Enterprise Integration Patterns
===============================================

![00 EIP Banner](/assets/img/2019/03/00 EIP Banner.png)

Gartner has recently coined the term "[Citizen Integrator](https://www.gartner.com/doc/3725217/citizen-integrators-bring-application-data)" referring to business users who have started leveraging these new low-code integration tools and platforms to build integration tasks. Developers and Citizen Integrators alike can now quickly build cloud-native integration solutions with these tools. Nevertheless, they can easily get into trouble if the requirements start to get a bit more complex and they don't do things appropriately.

Software Design Patterns document knowledge and expertise translated into templates that can be used to solve common challenges when designing a solution. Fifteen years ago, [Gregor Hohpe](https://twitter.com/ghohpe) and [Bobby Woolf](https://twitter.com/bobby_woolf) published the book [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/) which documents design patterns for messaging-based integration solutions, independently from their technology implementation. At that time, the tools were not as mature as they are today. However, having new tools does not mean that the integration challenges have changed in essence. Most of these patterns are still relevant nowadays on cloud-native solutions. Probably the main difference is that many of these patterns might be out-of-the-box features on many of the available platforms, so you don't need to worry about their low-level implementation.

That book focuses on integration patterns based on asynchronous messaging. However, the authors mention that there are other three integration styles.

- **File Transfer**, which is common in solution using Extract, Transform and Load (ETL) implementations.
- **Shared Database**, in which multiple applications use the same database.
- **Remote Procedure Invocation**. In this style, when an application requires information that is maintained in other application, then it requests it directly. Similarly, if an application wants to update data of another, then it makes the corresponding request. While this integration style is still very relevant, the technologies to implement it have evolved over the years, in the Microsoft world from COM to .NET Remoting, to SOAP Web Services to REST Web APIs and most recently to gRPC. Additionally, a middle layer has been introduced to minimise the coupling between requesting and serving applications. In the HTTP world, this is usually achieved with API Gateways. [Azure API Management](https://azure.microsoft.com/en-au/services/api-management/) is the Azure Integration Services' offering for API mediation.
- **Messaging**, in which asynchronous messages are exchanged between applications in a frequent and reliable manner. Messaging solves some of the challenges of Remote Procedure Invocation, by not requiring both systems to be up and running at the same time. This integration style is the one explored in the book and the focus of this series of posts.

It is worth mentioning that these authors have been working on new Messaging patterns focusing [on conversational and stateful](https://www.enterpriseintegrationpatterns.com/patterns/conversation/index.html) scenarios. These conversations can happen through asynchronous queue-based messaging or via direct synchronous calls.

If you want to build manageable, robust and resilient enterprise-grade enterprise messaging integration solutions, you must be aware of the Enterprise Integration Patterns to make sure you are making the best use of the available technologies to solve the functional and technical challenges.

Azure Integration Services Overview
===================================

The Azure Integration Services, is the set of services and platforms on Azure that we can leverage to develop, implement and manage enterprise-class integration solutions. The figure below, shows the different capabilities of an iPaaS, and which component of the Azure Integration Services we can use to implement them. You can find more details in [this post](/2018/02/02/microsoft-azure-ipaas-2).

 ![01 Azure Integration Services](/assets/img/2019/03/01 Azure Integration Services.png)

Challenges tackled by the Enterprise Integration Patterns
=========================================================

While creating the blueprints of integrations solutions, it is not uncommon that some challenges or requirements are initially missed or that existing proven approaches are not considered. That's why being aware and understanding the Enterprise Integration Patterns is key to successfully architect and implement enterprise-grade integration solutions.

The following sections describe some of these common challenges and what patterns try to address them. Challenges and patterns are grouped by the root patterns and are detailed in tables.

(*) There are some patterns that are not described in the book, but architects and developers are using them when building enterprise integration solutions. Those patterns are marked with an asterisk (*)

How to Construct a Message?
---------------------------

![10 Construction](/assets/img/2019/03/10 Construction.png)

These are common questions we need to answer when defining the structure and content of our messages, some of these are detailed in the table below. The patterns that tackle these challenges are the [**Message Construction**](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageConstructionIntro.html) patterns.

| **Challenges** | **Patterns** |
| --- |  --- |
| How do we define and structure my messages? What data should I include? | [Document Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DocumentMessage.html)[Event Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EventMessage.html)[Command Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CommandMessage.html)Query Message (*) |
| How do we shape the content of our message? Should the content and structure be coupled to an application context? Or decoupled from applications and defined to cover an organisation or industry domain? | Message Model (*) |
| What data should we include as metadata for messages to be routed, processed, grouped or tracked as required? | Message Header (*) |
| How can we define messages that support two-way communication between different applications? | [Request-Reply](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html) |
| How do we instruct a receiver application that a response is to be sent to a particular endpoint? | [Return Address](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ReturnAddress.html) |
| When there is a two-way communication via messaging, how can we correlate a response to the corresponding request? | [Correlation Identifier](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html) |
| How can we identify when messages are to be processed as part of a group and in a particular sequence? | [Message Sequence](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageSequence.html) |
| How can we define a valid lifespan of a message? | [Message Expiration](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageExpiration.html) |
| How can we defer a message to be processed after a certain point in time? | Message Activation (*) |
| How can we inform the receiver application the format a message is structured in? How can we support multiple versions of a message structure? | [Format indicator](https://www.enterpriseintegrationpatterns.com/patterns/messaging/FormatIndicator.html) |

How to transport the messages between applications?
---------------------------------------------------

![11 Channels](/assets/img/2019/03/11 Channels.png)

Once we have decided that we are going to integrate applications using asynchronous messaging, we need to define what channels to use to transport those messages. When defining the channels, we would have some questions to answer and might face some challenges described below. The [**Messaging Channels**](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingChannelsIntro.html) patterns help us to define and implement this aspect of our solution.

| **Challenges** | **Patterns** |
| --- |  --- |
| Shall we implement a direct interface between the sender application and the receiver application? Or shall the sender be unaware of potential receiver applications? | [Point-to-Point Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PointToPointChannel.html)[Publish-Subscribe Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html) |
| How is the receiver application meant to receive the message? Does it need to pull the message from the channel, or should the channel push the message to it? | Push-Pull Channel (*)Push-Push Channel (*) |
| How do we define a channel in a way that receivers know how to process a message? | [Datatype Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DatatypeChannel.html) |
| How do we manage invalid messages going through the channel? | [Invalid Message Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/InvalidMessageChannel.html) |
| Where do we store messages when they cannot longer be processed by the intended receiver? | [Dead-Letter Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DeadLetterChannel.html) |
| How do we make sure a message is delivered to the intended receiver? | [Guaranteed Delivery](https://www.enterpriseintegrationpatterns.com/patterns/messaging/GuaranteedMessaging.html) |
| When implementing Guaranteed Delivery, how do we make sure that retries don't have a negative impact on the intended receiver's health or performance? | Circuit Breaker (*) |
| How do we reduce the complexity of sending messages to a particular channel? | [Channel Adapter](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelAdapter.html) |
| How can we connect multiple applications so that they can communicate via messaging in a decoupled and organised manner? | [Message Bus](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBus.html) |
| When using disparate Message Channels, how can we connect the two? | [Messaging Bridge](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingBridge.html) |

How to connect Applications to Message Channels?
------------------------------------------------

![12 Endpoints](/assets/img/2019/03/12 Endpoints.png)

It is very important to understand how we are going to connect applications to the messaging channels. The following challenges are addressed by the [**Messaging Endpoints**](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingEndpointsIntro.html) patterns.

| **Challenges** | **Patterns** |
| --- |  --- |
| How do we reduce the complexity of receiving or sending messages to a particular application? | Application Adapter (*) |
| How can we reduce the complexity of connecting a messaging channel to an application? | [Messaging Gateway](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingGateway.html) |
| How do we map and serialise an applications domain object in the database into a meaningful independent message? | [Messaging Mapper](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingMapper.html) |
| How do we implement transactions so that messages are not lost in the process of sending or receiving a message to or from a channel? | [Transactional Client](https://www.enterpriseintegrationpatterns.com/patterns/messaging/TransactionalClient.html) |
| How can messages be consumed based on the readiness of the consumer? | [Polling Consumer](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PollingConsumer.html) |
| How can messages be consumed as soon as they are available? | [Event-Driven Consumer](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EventDrivenConsumer.html) |
| How can we process multiple messages at a time with multiples workers on the receiver application? | [Competing Consumers](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CompetingConsumers.html) |
| How can we control how many messages are processed at a given time based on the receiver requirements or capacity? | Throttled Consumer (*)Singleton Consumer (*) |
| How can a receiver application select which messages to consume? | [Selective Consumer](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageSelector.html) |
| How can different consumers receive messages in a coordinated and selective way from a single channel | [Message Dispatcher](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageDispatcher.html) |
| How can receiver applications avoid missing messages while they are not online or listening to the channel? | [Durable Subscriber](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DurableSubscription.html) |
| How can a receiver application deal with duplicate or stale messages? | [Idempotent Receiver](https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html)Message Deduplication (*)Stale Message (*) |
| How can sender or receiver application restrict the time when they can send or receive messages to or from the channel? | Service Window (*) |
| How can a receiver application consume messages received via both asynchronous messaging and synchronous channels? | [Service Activator](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingAdapter.html) |

How to route messages to the intended channel or receiver application?
----------------------------------------------------------------------

![13 Routing](/assets/img/2019/03/13 Routing.png)

Routing a message to the corresponding channel or the intended receiver application has some challenges that we need to solve. The table below describes some of them and the [**Message Routing**](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageRoutingIntro.html) patterns that could help to address them.

| **Challenges** | **Patterns** |
| --- |  --- |
| How can we perform different complex processing steps while maintaining some level of decoupling and flexibility? | [Pipes and Filters](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html) |
| How can we pass messages to different filters, components or receivers depending on a set of conditions? | [Message Router](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageRouter.html) |
| How do we implement dynamic routing based on the content or headers of a message? | [Content-Based Routing](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ContentBasedRouter.html) |
| How can we avoid non-relevant messages to be delivered? | [Message Filter](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Filter.html) |
| How can we validate whether a message is valid?  | Message Validation (*) |
| How can we maintain a dynamic configuration on the routing rules based on the receivers needs? | [Dynamic Router](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DynamicRouter.html) |
| How can we let the message define the list of receivers? | [Recipient List](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RecipientList.html) |
| How can we process messages which are composed of multiple elements which require its own processing? | [Splitter](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Sequencer.html) |
| How can we combine different messages into a larger message? | [Aggregator (Batching)](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Aggregator.html) |
| How can we get out-of-sequence related messages back into sequence? | [Resequencer](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Resequencer.html) |
| How to process a message composed of multiple elements, which require a coordinated but different handling? | [Composed Message Processor](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DistributionAggregate.html) |
| How to process messages which are to be sent to multiple receivers, for each of which we need to receive and process and response?  | [Scatter-Gather](https://www.enterpriseintegrationpatterns.com/patterns/messaging/BroadcastAggregate.html) |
| How do we handle a message which is to be sent for processing to different steps when the sequence is defined at runtime? | [Routing Slip](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RoutingTable.html) |
| How do we route a message through different processing steps when these are not sequential? | [Process Manager](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html) |
| How to avoid point-to-point integrations by decoupling senders and receivers while maintaining a centralised control of the message routing? | [Message Broker](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBroker.html) |

How can we process messages when different formats are required?
----------------------------------------------------------------

![14 Transformation](/assets/img/2019/03/14 Transformation.png)

Once we know how messages are constructed and how are to be routed, we might need to transform a message from a proprietary format to a canonical format or to another proprietary format. This transformation also has some challenges, some of these are described in the table below and [**Message Transformation**](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageTransformationIntro.html) patterns that can help to alleviate these challenges are suggested.

| **Challenges** | **Patterns** |
| --- |  --- |
| How to interchange information between different systems when they all use different formats and structures? | [Message Translator](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageTranslator.html) |
| How can we add metadata to a message body when it is required by the messaging channels or processors? | [Envelope Wrapper](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EnvelopeWrapper.html) |
| How do we satisfy a receiver application data requirements when not all the fields are available in the original system produced by the sender? | [Content Enricher](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DataEnricher.html) |
| How do we simplify a message when the original message is too complex or large for the intended receiver application?How do we remove sensitive data fields that are not intended for a receiver application? | [Content Filter](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ContentFilter.html) |
| How can we transfer large messages when the messaging channel has some size limits?How do we reduce the bandwidth requirements to interchange large messages between components while making the whole message available? | [Claim-Check](https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html)Compression/Decompression (*) |
| How do we standardise messages that contain same business information represented in different formats? | [Normaliser](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Normalizer.html) |
| How can you minimise coupling between applications that interchange messages when they use different message formats or structures? | [Canonical Data Model](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html) |
| How can we secure the message content when sensitive data could be captured by a man-in-the-middle attack? | Encryption/Decryption (*) |

How do we manage the integration platform and solution? 
--------------------------------------------------------

![15 Management](/assets/img/2019/03/15 Management.png)

Operating Integrations Solutions is usually not an easy task which brings some interesting challenges that should be considered up front. The table below describes some of these challenges and the [**Platform Management**](https://www.enterpriseintegrationpatterns.com/patterns/messaging/SystemManagementIntro.html) patterns that can be used to address them.

| **Challenges** | **Patterns** |
| --- |  --- |
| How can we manage integration solutions that leverage multiple platforms and deployed across multiple regions? | [Control Bus](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ControlBus.html) |
| How could we control the series of steps messages are to go through based on external factors to maximise the manageability and operability of the solution? | [Detour](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Detour.html) |
| How can you inspect messages that are being processed through a particular step or messaging channel? | [Wire Tap ](https://www.enterpriseintegrationpatterns.com/patterns/messaging/WireTap.html) |
| How can you trace and analyse messages that are being processed across multiple service and components? | [Message History](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageHistory.html) / [Message Store](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageStore.html) |
| How can you be aware of messages that could not be processed, and potentially fix them, to then resubmit them? | Manual Resubmission / Fix and Repair (*) |
| How can you track that an expected reply message has been received for a particular request? | [Smart Proxy](https://www.enterpriseintegrationpatterns.com/patterns/messaging/SmartProxy.html) |
| How can you test the health of a messaging solution end-to-end in production without unexpected side effects? | [Test Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/TestMessage.html) |
| How can we remove stale messages from a channel? | [Channel Purger](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelPurger.html) |
| How can we get notified when the platform is not performing as expected? | Alerts and Notifications (*) |

Wrapping Up
===========

In this post I have discussed many of the different challenges we can face when architecting and implementing enterprise messaging-based integration solutions and the integration patterns which offer us documented knowledge and templates that can be leveraged to face these challenges. These patterns are technology agnostic and were originally defined more than 15 years ago; however, given that the challenges we face today are very similar, if not the same, to those described at that time, these patterns are still very relevant in messaging-based distributed solutions.

I also gave a very brief introduction of the Azure Integration Services. In the following posts of this series, I'll cover these enterprise integration patterns in more detail and briefly describe how to implement them using the Azure Integration Services suite to build cloud-native integration. Stay tuned for the following posts of this series! 

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
