---
layout: post
title: Custom Distributed Tracing and Observability Practices in Azure Functions – Part 1 - Introduction
date: 2021-02-19 10:00
author: Paco de la Cruz
comments: true
category: Azure Functions
tags: [Azure Functions, Application Insights, Azure, Observability]
---
<img src="/assets/img/2021/02/intro-feature.png" alt="intro-feature" width="1200" style="width: 1200px;">
<h2 id="introduction">Introduction</h2>
<p>As developers, we tend to focus our efforts on building and shipping our services and apps to production, but it’s quite common that we forget to think about what happens after go-live. Once we reach production, the solution becomes someone else’s problem. But, even if we could build bug-free services, distributed systems will fail. And if we don’t design and build our services with traceability and observability in mind, we won’t give the means to the operations team to troubleshoot problems when they arise. This is particularly important when our services are running in the background and there is no user interacting with the application. It might take a while to realise that there was an issue in production and a lot of effort to understand what happened.</p>
<!--more-->
<p><a href="https://azure.microsoft.com/en-au/services/functions/?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Azure Functions</a> is a fully managed serverless compute platform that allows us to implement backend services with increased productivity. Azure Functions provides very rich inbuilt <a href="https://docs.microsoft.com/en-us/azure/azure-functions/analyze-telemetry-data?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">telemetry with Application Insights</a><span>.</span> In a previous <a href="/correlated-structured-logging-on-azure-functions" rel="noopener" target="_blank">post</a>, I described how to implement distributed structured logging in Azure Functions in a way that you can correlate custom log events created in different functions. This time, I’m writing a new series of posts that describes how to implement custom tracing and some observability practices in Azure Functions to meet some of the common requirements that operations teams have by adding business-related metadata while also leveraging the structured logging capabilities. &nbsp;</p>
<p>The approach suggested in this series works well in integration solutions following the <a href="/enterprise-integration-patterns-on-azure-messaging-channels.md#publish-subscribe-channel" rel="noopener" target="_blank">publish-subscribe integration pattern</a>, implemented using Azure Functions. It also considers the <a href="/enterprise-integration-patterns-on-azure-routing#splitter" rel="noopener" target="_blank">splitter integration pattern</a>. Bear in mind that this approach is opinionated and based on many years of experience with this type of solutions. Similar practices could be used in other types of scenarios, such as synchronous APIs.&nbsp;</p>
<p>This post is part of a series outlined below:</p>
<ul>
<li>Introduction (this article) – describes the scenario and why we might need custom distributed tracing in our solution.</li>
<li><a href="/distributed-tracing-and-observability-with-azure-functions-2-design" rel="noopener" target="_blank">Solution design</a> – outlines the detailed design of the suggested solution.</li>
<li><a href="/distributed-tracing-and-observability-with-azure-functions-3-implementation" rel="noopener" target="_blank">Implementation</a> – covers how this is implemented using Azure Functions and Application Insights.</li>
</ul>
<h2 id="scenario">Scenario</h2>
<p>To demonstrate how to implement custom distributed tracing and observability practices with Azure Functions, I’ll use a common scenario, the publishing and consuming of user update events. Think of a HR or CRM system pushing user update events via webhooks for downstream systems to consume. To better illustrate the scenario, let’s follow a high-level sequence diagram that has the following participants:</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/02/participants-01.png" alt="participants-01" width="1746" loading="lazy" style="width: 1746px;"></p>
<p>&nbsp;</p>
<ul>
<li><strong>Webhook</strong> - an HR or CRM system pushing user update events via webhooks, which sends an array of user events and expects a HTTP response.</li>
<li><strong>User Updated Publisher Azure Function</strong> - receives the HTTP POST request from the webhook, validates the batch message, splits the message into individual event messages, publishes the messages into a Service Bus queue, and returns the corresponding HTTP response to the webhook.</li>
<li><strong>User Updated Service Bus queue</strong> - receives and stores individual user event messages for consumers.</li>
<li><strong>User Updated Subscriber Azure Function</strong> – listens to messages in the queue, performs the required validations and message processing, and delivers the message to the target system.</li>
<li><strong>Target System</strong> – any system that needs to be notified when users are updated, but requires an integration layer to do some validation, processing, transformation, and/or custom delivery.</li>
</ul>
<p>The process in this scenario could be depicted through the sequence diagram below. The numbered flows in the diagram are detailed after.</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/02/pubsub-sequence.png" alt="pubsub-sequence" width="831" style="width: 831px;"></p>
<p>&nbsp;</p>
<ol>
<li>A webhook pushes user update events in batch to the User Updated Publisher Azure Function (publisher function) via a HTTP post.</li>
<li><span> The publisher function validates the request payload.</span></li>
<li><span> If the request payload is invalid: The publisher function returns HTTP 400 – Bad Request to the webhook.</span></li>
<li><span> If the request is valid: The publisher function splits the body into individual event messages (</span><a href="/enterprise-integration-patterns-on-azure-routing#splitter" rel="noopener" target="_blank"><span>splitter pattern</span></a><span>).</span></li>
<li><span> For each message event in the batch: The publisher function sends the user event message to the Service Bus queue.</span></li>
<li><span> If all event messages were successfully delivered: The publisher function returns HTTP 202 Accepted to the webhook. </span></li>
<li><span> In the case of any exception: The publisher function returns HTTP 500 – Internal Server Error to the webhook. </span></li>
</ol>
<p style="padding-left: 18pt;">Asynchronously (temporal decoupling provided by Service Bus queues), and for each message available in the queue:</p>
<ol start="8">
<li>The User Updated Subscriber Azure Function (subscriber function) gets triggered by the user event message in the queue.</li>
<li>The subscriber function tries to deliver the message to the target system via a request.</li>
<li>The target system returns a response with the update request result.</li>
<li>If the update operation succeeded: The subscriber function completes the message, so it is deleted from the queue.</li>
<li>If the update operation was not successful, the message becomes available again for a retry after the message lock expires. When all attempts are exhausted, the message is dead lettered by the queue.</li>
</ol>
<h2 id="operations-requirements">Operations Requirements</h2>
<p>Now let’s think of what an operations team might need to support a solution like this in an enterprise and what we could offer as part of our design to meet their requirements.</p>
<p>The table below includes some of the common requirements from an operations perspective and what features we could offer to meet them. The right side of the table is a good segue into the next post of the series, which covers in more detail the solution design.</p>
<table style="border-color: #99acc2; border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: none;" cellpadding="4" border="1">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p><strong>Requirement</strong></p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p><strong>Potential Solution</strong></p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Being able to inspect incoming request payloads for troubleshooting purposes.</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Archive all request payloads as received.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Being able to correlate individual messages to the original batch and understand whether all messages in a batch were processed successfully.</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Capture a batch record count and a batch identifier in all tracing events for individual messages.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Ability to correlate all tracing events for every individual message.</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Capture a tracing correlation identifier in all tracing events for every message.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Being able to find tracing events for a particular message.</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Capture business-related metadata in all relevant tracing logs, including an interface identifier, message type, and an entity identifier.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Receive alerts when certain failures occur.</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Capture granular tracing event identifiers and tracing event statuses.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Being able to differentiate transient failures.</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Capture a delivery count when messages are being retried to identify when additional attempts are expected or when all attempts have been exhausted for a particular message.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Ability to query, filter, and troubleshoot what happens to individual messages in the integration solution.</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="349">
<p>Implement a comprehensive custom distributed tracing solution that allows correlation of tracing events, including tracing correlation identifiers, standard tracing checkpoints, granular event identifiers, status, and business-related metadata.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="wrapping-up">Wrapping Up</h2>
<p>In this post, I discussed why it is important to consider traceability and observability practices when we are designing distributed services, particularly when these are executed in the background with no user interaction. I also covered common requirements from an operations teams supporting this type of services and described the scenario we are going to use in our sample implementation. In the <a href="/distributed-tracing-and-observability-with-azure-functions-2-design" rel="noopener" target="_blank">next post</a>, I will cover the detailed design of the proposed solution using Azure Functions and other related services.</p>
<p>In this post, I discussed why it is important to consider traceability and observability practices when we are designing distributed services, particularly when these are executed in the background with no user interaction. I also covered common requirements from an operations teams supporting this type of services and described the scenario we are going to use in our sample implementation. In the <a href="/distributed-tracing-and-observability-with-azure-functions-2-design" rel="noopener" target="_blank">next post</a>, I will cover the detailed design of the proposed solution using Azure Functions and other related services.</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>