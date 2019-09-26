---
layout: post
title: Message Construction - Enterprise Integration Patterns on Azure
date: 2019-04-10 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Enterprise Integration Patterns, Azure iPaaS, Logic Apps, Service Bus, Event Grid, Azure Functions]
---
<base target="_blank"/>

![Construction 4](/assets/img/2019/04/Construction 4.jpg)

When we are designing a message-based integration solution, one of the first things we need to define is how the application data are going to be serialised into messages, so they can be transmitted to other applications. The **[Message Construction](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageConstructionIntro.html)** Enterprise Integration Patterns provide documented knowledge that can be leveraged when defining messages. In this post, I'll cover the Message Construction patterns and how these can be implemented using the Azure Integration Services. This is a part of the series which describes how to implement the [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/) on Azure:

- [Introduction](/2019/03/08/enterprise-integration-patterns-on-azure-intro)
- Message Construction (this)
- [Messaging Channels](/2019/05/09/enterprise-integration-patterns-on-azure-messaging-channels)
- [Messaging Endpoints](/2019/06/05/enterprise-integration-patterns-on-azure-endpoints)
- Message Routing
- Message Transformation
- Platform Management

The remaining posts will be published in the following weeks / months.

When we are designing how the data are to be transmitted, we need to consider:

- **Intent**: A message can have different intents. Based on the intent, we can classify the messages as **Document Message,** **Event Message**, **Command Message**, or **Query Message** as described in the following sections.
- **Content and properties**. We need to define what is going to be inside a message, and also the metadata properties or headers. Content is usually defined by the context; depending on the context, a message could be coupled to an application data model or decoupled as a **Canonical Data Model**. Metadata or headers are normally used for routing, filtering or tracking and are not necessarily part of the application data. Additionally, a message can be atomic (single object) or contain multiple objects (batched).
- **Format**: A message can be expressed in different formats, like JSON, XML, flat-structured (comma delimited), encoded stream, etc., and have multiple versions. 

In the following sections, I will cover the Message Construction Enterprise Integration Patterns. Those marked with an asterisk (*) are those which are not described [in the original book](https://www.enterpriseintegrationpatterns.com/books1.html), but I suggest to consider.

Document Message (Intent) 
--------------------------

A **[Document Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DocumentMessage.html)** is used to transfer a data structure between applications, usually containing the full state at a point in time of an entity. The importance relies more on the content than on the timing of the message. Additionally, a document message does not give instructions to the receiver application of what to do with the message. It is up to the receiver application to decide how to process it. This approach reduces the level of coupling between sender and receiver applications.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>A Document Message can be implemented as a message that can be transmitted over <span><a href="https://azure.microsoft.com/en-au/services/service-bus/" rel=" noopener" target="_blank">Azure Service Bus</a></span> Queues or Topics as <span><strong><a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingChannelsIntro.html" rel=" noopener" target="_blank">Messaging Channel</a></strong></span>.</p>
</td>
</tr>
</table>


Event Message (Intent)
----------------------

An **[Event Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EventMessage.html)** is useful to transfer a notification when an event occurs in an object, usually containing the description of the event occurred and a reference to the object, but not the full state of it. The importance relies mostly on the timing. Due to the importance of the timing, the sender should issue the event message as fast as possible.

In some cases, a combination of a document and an event can be included in a message. This combination might be required when the receiver application needs to be informed of events, but at the same time needs the full state of the object to be able to process the event message.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Given the nature of a <strong>Command Message</strong>, Azure Service Bus Queues or Topics are better suited as messaging channels to transmit them.</p>
</td>
</tr>
</table>

Command Message (Intent) 
-------------------------

A **[Command Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CommandMessage.html)** is a request encapsulated as a message and usually targets one particular receiver application. A **Command Message** contains a clear instruction of what to do with a message on the receiver side. A Command Message couples the sender with the receiver application, as the source needs to be aware of the receiver in order to prepare a command compatible with the receiver application and also relevant according to the receiver state. Due to this coupling, command messages are not always recommended in message-based enterprise integration solutions; unless there is a requirement on the receiver application for its implementation.

Those who have worked with some standards like [OAGIS](https://oagi.org/DownloadsResources/tabid/143/Default.aspx) Business Object Documents (BOD) might be familiar the use of Command Messages and how this type of messages can increase coupling. 

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Given the nature of a <strong>Command Message</strong>, Azure Service Bus Queues or Topics are better suited as messaging channels to transmit them.</p>
</td>
</tr>
</table>

Query Message (Intent) (*)
--------------------------

Some people can argue that a Query Message is just a type of **Command Message**, and as such it is not described in the Enterprise Integration Patterns book; but there are [some important differences](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation) that we can highlight. The intent is certainly different. While a **Command Message** is usually an instruction for the intended receiver of what to do with the message and does not always require a response message, a **Query Message** instructs the receiver application that information is required to be returned to the caller. Similar to a **Command Message**, **Query Messages** usually result in coupling between the receiver and target applications, thus this pattern should not be abused in asynchronous messaging solutions and used only when strictly required.   

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Given the nature of a <strong>Query Message</strong>, Azure Service Bus Queues or Topics are better suited as messaging channels to transmit them.</p>
</td>
</tr>
</table>

**Examples of messages with different intents**

To better understand the differences between the message intents, we can think of different scenarios as detailed below:

| **Scenario: Employee Management** ||
| --- | --- |
| **Document Message:** | Employee (full state at a point in time)  |
| **Event Message** | EmployeeHired EmployeeTerminated |
| **Command Message** | CreateEmployeeAccountDisableEmployeeAccountDeleteEmployeeAccount |
| **Query Message** | GetEmployee |
| **Scenario: Azure Resource** ||
| **Document Message** | Azure Resource Definition (ARM Template in JSON) |
| **Event Message** | ResourceCreated Event Grid Event |
| **Command Message** | Create Resource |
| **Query Message** | Show Resource |

Message Model (*)
-----------------

When defining a message, we need to know whether the message is tightly coupled to an application context or defined in a decoupled way to represent a canonical business object.

- **Application Model**: Usually, when we extract a message from an application or when we are pushing a message to an application, we use a model and structure which is tightly coupled to that application so that can easily be extracted or pushed. This is true when we use an application API to extract or push messages and must use the model defined by that API. Similarly, when a message is created using the property names and structures of the database of the corresponding application.
- **[Canonical Data Model](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html)**: an Application Model gives you the advantage that the corresponding application can easily understand that message. However, when you are transmitting that message to a different application or organisation, a **Message Translator** has to be implemented to generate the corresponding message in a model that the receiver application can understand. In many cases, having a message model that is decoupled from the sender and receiver applications, provides advantages. So instead of using an application proprietary model we can use a **Canonical Data Model** which is usually defined using a business domain approach. More details of this pattern in the **Canonical Data Model** pattern section.

The **Message Model** is not mentioned as a pattern in the Enterprise Integration Patterns; however, I believe it is important to consider this aspect when constructing the message.

**Implementation**

<table>
<tr>
<td width="75">
<p><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></p>
<p><img src="/assets/img/2019/04/Event Grid.png" alt="Event Grid" width="81" style="width: 81px;" sizes="(max-width: 81px) 100vw, 81px"></p>
</td>
<td width="632">
<p>Depending on the model and intent defined, Azure Service Bus or Event Grid could be used as the Messaging Channel</p>
</td>
</tr>
</table>

Message Header (*)
------------------

In some scenarios, a message is not only composed by its body, but metadata or headers which are not necessarily part of the application data but are required for the message processing steps, like routing, filtering, tracking, etc. Having a **Message Header** is mentioned in the Enterprise Integration Patterns book but not described as a pattern. However, when defining how to construct a message, it is important to think in the message header properties that will be required as part of the implementation. Depending on the requirements, a **Message Header** can contain different properties; some properties that are common in enterprise integration solutions are:

- **MessageId**: which can be used for tracking, de-duplication or idempotency.
- **CorrelationId**: required for the **Return-Reply**
- **TrackingId** or **OperationId**: Useful for logging and tracking across multiple components.
- **ReturnAddress**: required for the **Return-Reply**
- **ExpiresAt**: required for the **Message Expiration**
- **AvailableFrom**: required for the **Message Activation (*)**
- **Timestamp** or **EffectiveDateTime**: used to specify the time when the message is/was effective.
- **ContentType**: indicates the type of content being sent.
- **Format**: indicates the format the message, useful for the **Format Indicator**
- **Version**: indicates the receiver the version of the message, useful for the **Format Indicator**
- **ApplicationMetadata or BusinessMetadata**: contain application or business data promoted as message headers required for the different **Message Routing** patterns or for logging and tracking.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Service Bus allows to&nbsp;specify&nbsp;<a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messages-payloads" rel=" noopener">built-in and <span>custom message properties or headers</span></a>.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Event Grid.png" alt="Event Grid" width="81" style="width: 81px;" sizes="(max-width: 81px) 100vw, 81px"></td>
<td width="632">
<p>Event Grid event messages are self-contained. Thus, message headers must be included in the event message body. The event schema of Event Grid is described <span><a href="https://docs.microsoft.com/en-us/azure/event-grid/event-schema" rel=" noopener" target="_blank">here</a></span>. As you can see, there is a property for the event data, and other metadata properties, including id, subject, eventType, eventTime, dataVersion, metadataVersion, etc.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>When sending a message directly to a Logic App <span><a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-http-endpoint#reference-content-from-an-incoming-request" rel=" noopener" target="_blank">via an http request</a></span>, we can utilise default or custom (x-) http headers, which in turn can be used to populate headers in Service Bus or Event Grid in a message-based integration, as described above.</p>
</td>
</tr>
</table>

Request-Reply
-------------

Sometimes a sender (requestor) needs to get a response from the receiver (replier), for instance when sending a **Command** or **Query Message**. This scenario is covered by the **[Request-Reply](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html)** pattern.  Thanks to the popularity of Web APIs, we are very familiar with synchronous request-reply interchanges; however, a request-reply can be also be implemented using asynchronous messaging with a call back, which is the pattern described in this section. The Request message should contain a **Return Address** and a **Correlation Identifier** that would specify which request a reply is for. Depending on the use case, a reply message could include an acknowledgement (a **Command Message** was executed successfully), a negative acknowledgement (the command was unsuccessful), or a result value to a **Query Message**.

**Implementation:**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>On Logic Apps, the <strong>Request-Reply</strong> pattern can be implemented with the Webhook action, which covers out-of-the-box the <strong>Request-Reply</strong>, the <strong>Return Address</strong> and the <strong>Correlation Identifier</strong> patterns.</p>
<p>This implementation is described <span><a href="/2017/07/17/correlation-identifier-pattern-on-logic-apps" rel=" noopener" target="_blank">here</a></span>.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Additionally, using Logic Apps and the Service Bus trigger with sessions, we can implement this pattern and the <strong>Correlation Identifier</strong> as described <span><a href="https://platform.deloitte.com.au/articles/logic-apps-correlation-and-message-dependency-management-on-logic-apps-with-service-bus" rel=" noopener" target="_blank">here</a></span>.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>When code-based workflows are preferred, Azure Durable Functions can also be utilised to implement the <strong>Request-Reply</strong>, the <strong>Return Address</strong> and the <strong>Correlation Identifier</strong> patterns as described in <span><a href="/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid" rel=" noopener" target="_blank">this post</a></span>.</p>
</td>
</tr>
</table>

Return Address
--------------

The **[Return Address](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ReturnAddress.html)** pattern was described in the **Request-Reply** section above. When messages must be associated, e.g. a reply message has a one-to-one correspondence with a request message, the reply message has to be sent to a particular address on which the requestor expects it. A **Return-Address** on the request message allows to decouple a replier from the requestor, as the address does not have to be hard-coded on the replier. This pattern even allows a requestor to specify different return addresses to different call-back processors. A **Return Address** is usually put in the header of the message, as it is not related to the application data transmitted.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>The Logic Apps Webhook action implements the <strong>Return Address</strong> out-of-the-box. The Return Address also includes a <strong>Correlation Identifier</strong> which links to the workflow instance that sends the request and is waiting for a response. This implementation is described <span><a href="/2017/07/17/correlation-identifier-pattern-on-logic-apps" rel=" noopener" target="_blank">here</a></span>.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Service Bus also implements the <span><strong><a href="https://docs.microsoft.com/en-us/rest/api/servicebus/message-headers-and-properties#message-headers" rel=" noopener" target="_blank">ReplyTo</a></strong></span> header out-of-the-box which can be used for this purpose.&nbsp;</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>When implementing this on Durable Functions, you need to specify the Return Address, e.g. the http endpoint where a Durable Function client is listening to continue with the workflow. Additionally, you would need to send a <strong>Correlation Identifier</strong> (e.g. the orchestration <em>instanceId</em> explained below) to be able to correlate the reply with the original request, as described in <span><a href="/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid" rel=" noopener" target="_blank">this post</a></span>.</p>
</td>
</tr>
</table>

Correlation Identifier
----------------------

The **[Correlation Identifier](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html)** pattern was mentioned in the **Request-Reply** section above. When implementing the **Request-Reply** pattern, a reply message must contain a unique **Correlation Identifier** that indicates which request message a particular reply is for. This would allow the original sender (requestor) to correlate a reply to continue with its processing.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>The Logic Apps Webhook action implements the <strong>Correlation Identifier </strong>together with the <strong>Request Reply</strong> and <strong>Return Address</strong> patterns out-of-the-box as described <span><a href="/2017/07/17/correlation-identifier-pattern-on-logic-apps" rel=" noopener" target="_blank">here</a></span>. If the original <strong>Correlation Identifier</strong> is not accepted by the replier, a mapping between a unique value in the message and the <strong>Correlation Identifier</strong> can be implemented as described in the <span><a href="/2017/07/17/correlation-identifier-pattern-on-logic-apps" rel=" noopener" target="_blank">post</a></span>.&nbsp;</p>
</td>
</tr>
<tr>
<td width="75">
<p><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></p>
<p><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></p>
</td>
<td width="632">
<p>When implementing the pattern using Logic Apps with the Service Bus trigger in the middle of the workflow, a Service Bus <strong><u>SessionId</u></strong> is used as <strong>Correlation Identifier</strong>. Additionally, this approach can process more than one replies for the same request as described <span><a href="https://platform.deloitte.com.au/articles/logic-apps-correlation-and-message-dependency-management-on-logic-apps-with-service-bus" rel=" noopener" target="_blank">here</a></span>.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Each orchestration instance of Azure Durable Functions has an <em>instanceId. </em>This can be used as the <strong>Correlation Identifier</strong> as described in the <span><a href="/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid" rel=" noopener" target="_blank">post</a></span>.</p>
</td>
</tr>
</table>

Message Sequence
----------------

The **[Message Sequence](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageSequence.html)** pattern is useful when the amount of data that is to be transmitted cannot be fitted in one single message, due to message size or application constraints. Another use case is when messages have dependencies on other messages, thus must be processed in certain sequence. So messages can be processed sequentially, each message must have certain headers described below:

- **Sequence Identifier**: to distinguish the group of messages.
- **Position Identifier:** to order each message in a sequence
- **Size or End Indicator:** to indicate the number of messages in the group or identify the last message of the group.

As you can see, the **Message Sequence** pattern requires to be implemented using a group of messages and cannot be an endless sequence. When sequenced messages can be transmitted independently, the [**Resequencer**](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Resequencer.html) pattern must be implemented on the receiver side.  

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Service Bus allows you to implement the <strong>Message Sequence</strong> Pattern. We can use the <strong>Session</strong> header as Sequence Identifier. For Position Identifier, we can use the <strong>SequenceNumber</strong> header. More information on the <span><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-sessions" rel=" noopener" target="_blank">official documentation</a></span>.</p>
</td>
</tr>
</table>

Message Expiration
------------------

In some scenarios, messages become invalid or meaningless and should be ignored when received or processed after a certain time. Additionally, when implementing messaging, once the sender has emitted a message, it has no way to recall the message. To be able to overcome this problem, we can implement the **[Message Expiration](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageExpiration.html)** pattern to advise the receiver to not process a message after that time.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Azure Service Bus has built in support for <strong>Message Expiration</strong>. Individual message expiration can be set using the <span><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-expiration" rel=" noopener" target="_blank">TimeToLive</a></span> property. Additionally, all messages put into a queue or topic will inherit the <span><a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-expiration#entity-level-expiration" rel=" noopener" target="_blank">defaultMessageTimeToLive</a></span> property set for that queue or topic. Expired messages can be dropped or sent to the dead-letter sub-queue. When no time to live is set up on Service Bus, messages are not meant to expire.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Event Grid.png" alt="Event Grid" width="81" style="width: 81px;" sizes="(max-width: 81px) 100vw, 81px"></td>
<td width="632">
<p>Azure Event Grid is meant for event messages. As mentioned earlier, timing is very important for event messages. That is why, Event Grid has a short maximum time to live for all events. At the time of writing the maximum is 24 hours, however you can configure it as detailed <span><a href="https://docs.microsoft.com/en-us/azure/event-grid/delivery-and-retry" rel=" noopener" target="_blank">here</a></span>.</p>
</td>
</tr>
</table>

Message Activation (*)
----------------------

The Enterprise Integration Patterns book does not mention the **Message Activation** pattern. However, in different scenarios, this can be a handy pattern. Imagine the case where a message is produced by a sender, but it should not be received or consumed before a certain time. Sometimes, a future **Event Message** is produced (scheduled) on the system of record but the system emits the event straight away, without being able to wait until it actually occurs. The **Event Message** is already produced, but shouldn't be consumed or processed yet. An example can be when an employee is marked as terminated on the system of record days in advanced, the system of record emits the termination event with an effective date, and the message should not be consumed by the receivers before the effective date. This can be achieved using an Activation Date header.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>Azure Service Bus supports message activation out-of-the-box with the <span><a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.scheduledenqueuetimeutc?view=azure-dotnet" rel=" noopener" target="_blank">ScheduledEnqueueTimeUtc</a></span> property, which is used to delay messages sending to a specific time in the future.</p>
</td>
</tr>
<tr>
<td width="75">
<p><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></p>
<p><img src="/assets/img/2019/04/Event Grid.png" alt="Event Grid" width="81" style="width: 81px;" sizes="(max-width: 81px) 100vw, 81px"></p>
</td>
<td width="632">
<p>For <strong>Event Message</strong> scenarios, where a <strong>Push-Push Channel</strong>&nbsp;is required, a Logic App can be implemented in which a delay action is used to hold the message before sending it to Event Grid. The delay could be set&nbsp;based on a timestamp or time span in the message or business rules at run time. Thanks to <a href="https://twitter.com/kevinlam_msft" rel=" noopener">@KevinLam_MSFT</a> for suggesting this handy implementation pattern.&nbsp;</p>
</td>
</tr>
</table>

Format Indicator
----------------

The **[Format Indicator](https://www.enterpriseintegrationpatterns.com/patterns/messaging/FormatIndicator.html)** pattern is useful to support future changes on how the data is represented on a message. When different systems are interacting through messages, the updates to support newer data structures will hardly happen in all systems at the same time. To support multiple versions working at the same time, a Format Indicator can be used so receivers or consumers can know how to interpret and process a message. The **Format Indicator** can be as simple as version number, or a namespace or schema version. Depending on the requirements and the platform, the **Format Indicator** could be transferred using a message header or within the message body.

**Implementation**

<table>
<tr>
<td width="75"><img src="/assets/img/2019/04/Azure Service Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>In Service Bus, the <em>ContentType </em>property could be used to indicate the format. Additionally, a <span><a href="https://docs.microsoft.com/en-us/rest/api/servicebus/message-headers-and-properties#message-properties" rel=" noopener" target="_blank">user defined message property</a></span> give much more flexibility to include versions or namespaces.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Event Grid.png" alt="Event Grid" width="81" style="width: 81px;" sizes="(max-width: 81px) 100vw, 81px"></td>
<td width="632">
<p>The Event Grid <span><a href="https://docs.microsoft.com/en-us/azure/event-grid/event-schema" rel=" noopener" target="_blank">event schema</a></span> allows us to determine the format of an event using the <em>dataVersion </em>property.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Logic Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>When sending a message directly to a Logic App <span><a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-http-endpoint#reference-content-from-an-incoming-request" rel=" noopener" target="_blank">via an http request</a></span>, we can utilise custom (x-) http headers, which in turn can be used to populate headers in Service Bus or Event Grid in a message-based integration.</p>
</td>
</tr>
<tr>
<td width="75"><img src="/assets/img/2019/04/Document.png" alt="Document" width="80" style="width: 80px;" sizes="(max-width: 80px) 100vw, 80px"></td>
<td width="632">
<p>In some scenarios, the format and version of a document is to be included within the message body. For instance, when dealing with XML, a namespace is the way to define the version.</p>
</td>
</tr>
</table>

Wrapping Up
-----------

In this post I have covered how to implement the Message Construction patterns using the Azure Integration Services. As we could see, in many cases, these patterns are already out-of-the-box features of the platform, and in other cases we need to take care of the implementation. Nevertheless, being aware of these challenges and the patterns which address them, allow us to be much better prepared when defining how to structure the messages for message-based enterprise integration solutions.

Happy integration!

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
