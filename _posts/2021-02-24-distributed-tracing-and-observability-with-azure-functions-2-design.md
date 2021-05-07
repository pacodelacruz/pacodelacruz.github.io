---
layout: post
title: Custom Distributed Tracing and Observability Practices in Azure Functions – Part 2 - Solution Design
date: 2021-02-24 10:00
author: Paco de la Cruz
comments: true
category: Azure Functions
tags: [Azure Functions, Application Insights, Azure, Observability]
---
<p><img src="/assets/img/2021/02/design-featured.png" alt="Design Featured" width="800" loading="lazy" style="width: 800px;"></p>
<h2 id="introduction">Introduction</h2>
<!--more-->
<p>In my <a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">previous post</a>, I discussed why it is important to consider traceability and observability practices when we are designing distributed services, particularly when these are executed in the background with no user interaction. I also covered common requirements from an operations team supporting this type of services and described the scenario we are going to use in our sample implementation. In this post, I will cover the detailed design of the proposed solution using Azure Functions, Application Insights, and other related services. As mentioned in <a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">that post</a>, I’ll be using the <a href="/enterprise-integration-patterns-on-azure-messaging-channels#publish-subscribe-channel" rel="noopener" target="_blank">publish-subscribe pattern</a>, but this approach can be tailored for other types of scenarios.</p>
<p>This post is part of a series outlined below:</p>
<ul>
<li><span> </span><a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank"><span>Introduction</span></a><span> – describes the scenario and why we might need custom distributed tracing in our solution. </span></li>
<li><span> Solution design (this article) – outlines the detailed design of the suggested solution.</span></li>
<li><span><a href="/distributed-tracing-and-observability-with-azure-functions-3-implementation" rel="noopener" target="_blank"> Implementation</a> – covers how this is implemented using Azure Functions and Application Insights.</span></li>
</ul>
<h2 id="tracing-spans">Tracing Spans</h2>
<p>In my <a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">previous post</a>, we discussed common requirements of operations teams and identified what features we can include in our solution to meet them. Let’s now think in more detail about how we want to meet these requirements in our solution. By following some of the concepts of the <a href="https://github.com/opentracing/specification/blob/master/specification.md" rel="noopener" target="_blank"><span>OpenTracing specification</span></a><span>, we can say that the scenario described in my <a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">previous post</a></span><span>&nbsp;could be composed of the </span><a href="https://opentracing.io/docs/overview/spans/" rel="noopener" target="_blank"><span>tracing spans</span></a><span> depicted below:</span></p>
<p><span><img src="/assets/img/2021/02/Spans.drawio.png" alt="Spans.drawio" width="302" loading="lazy" style="width: 302px; margin-left: auto; margin-right: auto; display: block;"></span></p>
<ul>
<li><strong>BatchPublisher</strong>: unit of work related to receiving a batch of events, splitting the batch, and publishing individual messages to a queue.</li>
<li><strong>Publisher (1-n)</strong>: unit of work that receives individual messages, does the corresponding processing (e.g. message validation, transformation), and publishes the message to a queue. The <em>Publisher</em> spans are <em>ChildOf </em>the <em>BatchPublisher</em></li>
<li><strong>Subscriber (1-n)</strong>: unit of work that subscribes to individual messages in the queue, does the corresponding processing (e.g. message validation, transformation), and delivers the message to the target system. A <em>Subscriber </em>span <em>FollowsFrom</em> a <em>Publisher </em></li>
</ul>
<h2 id="structured-logging-key-value-pairs">Structured Logging Key-Value Pairs</h2>
<p style="text-align: justify;">After defining the relevant tracing spans, let’s determine what we want to log as part of our tracing events. We are going to use structured logging with key-value pairs to be able to query, filter, analyse and comprehend our tracing data. The proposed key-value pairs are described in the table below, each with a defined scope. Those key-value pairs with a <em>cross-span</em> scope follow the concept of <a href="https://opentracing.io/docs/overview/tags-logs-baggage/#baggage-items" rel="noopener" target="_blank">baggage items</a>; meaning that the same value is preserved across processes or tracing spans for the traced entity. A <em>span</em> scope follows the concept of <a href="https://opentracing.io/docs/overview/tags-logs-baggage/#tags" rel="noopener" target="_blank">span tags</a>, which means that the same value is kept throughout the span for the traced entity. And those with scope <em>log </em>follow the <a href="https://opentracing.io/docs/overview/tags-logs-baggage/#logs" rel="noopener" target="_blank">log</a> concept, meaning that the value is only relevant to the tracing event.</p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: none;" cellpadding="4" border="1">
<tbody>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>Key</strong></p>
</td>
<td style="border: 1pt solid windowtext; width: 386px;">
<p><strong>Description</strong></p>
</td>
<td style="border: 1pt solid windowtext; width: 101px;">
<p><strong>Scope</strong></p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>BatchId</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Batch identifier to correlate individual messages to the original batch. It is highly recommended when using the <a href="/enterprise-integration-patterns-on-azure-routing#splitter" rel="noopener" target="_blank">splitter pattern</a>.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Cross-span</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>CorrelationId</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Tracing correlation identifier of an individual message.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Cross-span</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>EntityType</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Business identifier of the message type being processed. This allows to filter or query tracing events for a particular entity type. E.g. <em>UserEvent</em>, <em>PurchaseOrder</em>, <em>Invoice</em>, etc.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Cross-span</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>EntityId</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Business identifier of the entity in the message. This together with the <em>EntityType</em> key-value pair allow to filter or query tracing events for messages related to a particular entity. E.g. <em>UserId</em>, <em>PurchaseOrderNumber</em>, <em>InvoiceNumber</em>, etc.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Cross-span</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>InterfaceId</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Business identifier of the interface. This allows to filter or query tracing events for a particular interface. Useful when an organisation defines identifiers for their integration interfaces.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Span</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>RecordCount</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Optional. Only applicable to batch events. Captures the number of individual messages or records that are present in the batch.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Span</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>DeliveryCount</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Optional. Only applicable to subscriber events of individual messages. Captures the number of times the message has been attempted to be delivered. It relies on the Service Bus message <em>DeliveryCount </em>property.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Span</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>LogLevel</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p><a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.loglevel?view=dotnet-plat-ext-5.0&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">LogLevel</a> as defined by <em>Microsoft.Extensions.Logging</em></p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Log</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>SpanCheckpoint</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Defines the tracing span and whether it is the start or finish of it, e.g. <em>PublisherStart</em> or <em>PublisherFinish</em>. Having standard checkpoints allows correlating tracing events in a standard way.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Log</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>EventId</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Captures a specific tracing event that helps to query, analyse, and troubleshoot the solution with granularity.</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Log</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 147px;">
<p><strong>Status</strong></p>
</td>
<td style="width: 386px; border: 1pt solid windowtext;">
<p>Stores the status of the tracing event, e.g., succeeded or failed</p>
</td>
<td style="width: 101px; border: 1pt solid windowtext;">
<p>Log</p>
</td>
</tr>
</tbody>
</table>
<h2 id="publisher-interface-detailed-design">Publisher Interface Detailed Design</h2>
<p>Let’s now consider how we are going to be implementing this in more detail as part of our integration interfaces. The <em>BatchPublisher</em> and <em>Publisher</em> spans will be implemented as one publisher interface/component. So now let’s cover how we want to implement the tracing and observability practices in this interface. We will use the sequence diagram below, which has the following participants: &nbsp;</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/02/participants-pub.png" alt="participants-pub" width="1052" loading="lazy" style="width: 1052px;"></p>
<p>&nbsp;</p>
<ul>
<li><strong>Webhook</strong> - an HR or CRM system pushing user update events via webhooks, which sends an array of user events and expects a HTTP response.</li>
<li><strong>User Updated Publisher Azure Function</strong> - receives the HTTP POST request from the webhook, validates the batch message, splits the message into individual event messages, publishes the messages into a Service Bus queue, returns the corresponding HTTP response to the webhook, sends the relevant tracing logs to the Diagnostics Logs, and archives the initial request to Azure Storage.</li>
<li><strong>User Updated Service Bus queue</strong> - receives and stores individual user event messages for consumers.</li>
<li><strong>Diagnostic Logs in Application Insights -</strong> captures built-in telemetry and custom tracing produced by the Azure Functions.</li>
<li><strong>Storage Archive Azure Storage blob</strong> - stores request payloads.</li>
</ul>
<p style="padding-left: 18pt;">The sequence diagram below is an expansion of the first part of the diagram shown in the <a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">previous post</a>. In this, we are going to define in more detail the tracing events, including some relevant key-value pairs defined previously. Tracing log events are depicted using the convention: <code>LogLevel: SpanCheckPointId - EventId [Status]</code>.</p>
<p><img src="/assets/img/2021/02/pub-sequence.png" alt="Pub Sequence" width="1098" loading="lazy" style="width: 1098px;"></p>
<h2 id="subscriber-interface-detailed-design">Subscriber Interface Detailed Design</h2>
<p>The <em>Subscriber</em> span will be implemented in a subscriber interface. The sequence diagram below depicts the tracing and observability practices in this interface, and includes the participants as follows:</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/02/participants-sub.png" alt="participants-sub" width="967" loading="lazy" style="width: 967px;"></p>
<p>&nbsp;</p>
<ul>
<li><strong>User Updated Service Bus queue</strong> - receives and stores individual user event messages for consumers.</li>
<li><strong>User Updated Subscriber Azure Function</strong> – listens to messages in the queue and subscribes to messages, performs the required validations against the target system - to check whether it is a valid payload, if it is not a <a href="/enterprise-integration-patterns-on-azure-endpoints#stale-message" rel="noopener" target="_blank">stale message</a>, and if all message dependencies are available - tries to deliver the message to the target system, and sends the relevant tracing events to the Diagnostics Logs.&nbsp;&nbsp;</li>
<li><strong>Target System</strong> – any system that needs to get notified when users are updated, but requires an integration layer to do some validation, processing, transformation, and/or custom delivery.</li>
<li><strong>Diagnostic Logs in Application Insights -</strong> captures built-in telemetry and custom tracing produced by the Azure Functions.</li>
</ul>
<p style="padding-left: 18pt;">As in the previous diagram, tracing log events are depicted using the convention: <code>LogLevel: SpanCheckPointId - EventId [Status]</code>.</p>
<p style="padding-left: 18pt;">&nbsp;</p>
<p><img src="/assets/img/2021/02/sub-sequence.png" alt="Sub Sequence" width="736" loading="lazy" style="width: 736px;"></p>
<h2 id="wrapping-up">Wrapping Up</h2>
<p>In this post, we have described the design of an approach to meet common observability requirements of distributed services that run in the background using Azure Functions. In the next post of this series, we will cover how this can be implemented and how we can query and analyse the produced tracing events.</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>