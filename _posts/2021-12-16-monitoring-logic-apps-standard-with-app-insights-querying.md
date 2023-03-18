---
layout: post
title: Custom Distributed Tracing and Observability Practices in Azure Functions – Part 3 - Implementation
date: 2021-03-10 10:00
author: Paco de la Cruz
comments: true
category: Azure Functions
tags: [Azure Functions, Application Insights, Azure, Observability]
---
<p><img src="/assets/img/2021/02/build-featured.png" alt="Build Featured" width="800" loading="lazy" style="width: 800px;"></p>
<h2>Introduction</h2>
<p>In the previous post of the series, we described the design of an approach to meet common observability requirements of distributed services using Azure Functions. Now, in this post, we are going to cover how this can be implemented and how we can query and analyse the produced tracing logs.</p>
<!--more-->
<p>This post is part of a series outlined below:</p>
<ul>
<li><a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">Introduction</a> – describes the scenario and why we might need custom distributed tracing in our solution.</li>
<li><a href="/distributed-tracing-and-observability-with-azure-functions-2-design" rel="noopener" target="_blank">Solution design</a> – outlines the detailed design of the suggested solution.</li>
<li>Implementation (this) – covers how this is implemented using Azure Functions and Application Insights.</li>
</ul>
<p>I strongly suggest that you read these posts in sequence, so you understand what the solution presented here is trying to achieve.</p>
<h2>Solution Components</h2>
<p>The proposed solution is based on the Azure services described below:</p>
<table style="border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: none; width: 634px;" cellpadding="4" border="1">
<tbody>
<tr>
<td style="border: 1pt solid windowtext; width: 162px;">
<p><strong>Component</strong></p>
</td>
<td style="border: 1pt solid windowtext; width: 472px;">
<p><strong>Description</strong></p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 162px; text-align: center;">
<p><img src="/assets/img/2021/02/Azure%20Functions.png" alt="Azure Functions" width="81" loading="lazy" style="width: 81px;"></p>
<p>Azure Functions</p>
</td>
<td style="border: 1pt solid windowtext; width: 472px;">
<p>It contains the publisher and subscriber interfaces with a custom distributed tracing implementation.</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 162px; text-align: center;">
<p><img src="/assets/img/2021/02/App%20Insights.png" alt="App Insights" width="80" loading="lazy" style="width: 80px;"></p>
<p>Application Insights</p>
</td>
<td style="border: 1pt solid windowtext; width: 472px;">
<p>Receives and keeps all distributed tracing logs from the Azure Function.</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 162px; text-align: center;">
<p><img src="/assets/img/2021/02/Storage.png" alt="Storage" width="80" loading="lazy" style="width: 80px;"></p>
<p>Storage Account</p>
</td>
<td style="border: 1pt solid windowtext; width: 472px;">
<p>Used for request payload archiving. Leveraging the <a href="https://docs.microsoft.com/en-us/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">lifecycle management</a> capabilities of Azure Blob Storage to transition or delete archive blobs will allow you to optimise costs or achieve privacy compliance.</p>
</td>
</tr>
<tr>
<td style="border: 1pt solid windowtext; width: 162px; text-align: center;">
<p><img src="/assets/img/2021/02/Azure%20Service%20Bus.png" alt="Azure Service Bus" width="81" loading="lazy" style="width: 81px;"></p>
<p>Service Bus</p>
</td>
<td style="border: 1pt solid windowtext; width: 472px;">
<p>Used to implement temporal decoupling between the publisher and the subscriber interfaces.</p>
</td>
</tr>
</tbody>
</table>
<h2>Show me the Code</h2>
<p>Now, let’s get deeper into the solution code. The full solution can be found in <a href="https://github.com/pacodelacruz/observability-pubsub-functions" rel="noopener" target="_blank">this GitHub repo</a>, however I will go through some of the key components of the solution in this post. I’ve also added comments to my code, ensuring it’s easy to follow.</p>
<h3>Logging Constants</h3>
<p>I suggest using constants and enumerations in the tracing implementation to have consistent values across all the components that use them. In this class, I’ve defined all of them.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/f126342891c33b19b9f4c2c430b01532.js"></script>
</p>
<h3>LoggerExtensions</h3>
<p>To ease structured logging using the default <code>ILogger</code> provider in Azure Functions, I’ve created extension methods that provide typed signatures. The different key-value pairs defined in the previous post are being used in these methods.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/53baa5617216c43e192709c254dc3144.js"></script>
</p>
<h3>LoggerHelper</h3>
<p>To map tracing event status to the standard <code>ILogger.LogLevel</code>, I’ve created this helper with a method that returns the corresponding <code>LogLevel</code> based on the process status.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/cc2af1b15bdd894d3c4418e5831eaf99.js"></script>
</p>
<h3>Publisher Function (UserUpdatedPublisher)</h3>
<p>This is the implementation of the publisher component designed and described in the previous post as an Azure Function. This function receives a HTTP request with user events in the Cloud Events format, splits it into individual events, and sends them to a Service Bus queue.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/c0013c11d9be22c35219e572b5985729.js"></script>
</p>
<h3>Subscriber Function (UserUpdatedSubscriber)</h3>
<p>This class implements the subscriber component designed and described in the previous post as an Azure Function. This function receives a Service Bus message and simulates the delivery to a target system.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/bfdf00bd946664d0026f6a2bd0e58298.js"></script>
</p>
<p>As mentioned above, the full solution can be found in <a href="https://github.com/pacodelacruz/observability-pubsub-functions" rel="noopener" target="_blank">this GitHub repo</a>.</p>
<h2>Deploying the Solution</h2>
<p>In the <a href="https://github.com/pacodelacruz/observability-pubsub-functions/tree/main/infra" rel="noopener" target="_blank">GitHub repo</a>, I’ve included the ARM templates, which create all the infrastructure needed to have this up and running. You will also need to build the demo Azure Function solution and deploy it to your Azure Function App. Bear in mind that there might be some costs associated with these resources.</p>
<h2>Generating Traffic</h2>
<p>You can simulate the webhook calls by using the VS Code <a href="https://marketplace.visualstudio.com/items?itemName=humao.rest-client" rel="noopener" target="_blank">REST Client extension</a> and posting HTTP requests. A sample request is as follows.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/71521a2ad380adecd3d2feb5f8bb3392.js"></script>
</p>
<h2>Querying the Distributed Traces</h2>
<p>In the <span>,</span> we listed some of the common requirements that an operations team has when supporting distributed backend services. After a comprehensive design of the tracing approach and its implementation, now we can finally see the benefits in action. We are now able to query our distributed traces in a meaningful way.</p>
<p>Once the Azure Function is running and has logged tracing events, logs in Application Insights can be queried using the <a href="https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Kusto query language</a>. Let’s go through some of the queries I’ve prepared to meet the observability requirements described previously. All of these queries are available in the <a href="https://github.com/pacodelacruz/observability-pubsub-functions/tree/main/src/Integration.Observability.PubSub.FnApp/AppInsightsOperationsQueries" rel="noopener" target="_blank">GitHub repo</a>.</p>
<h3>Batch Publisher Span Traces</h3>
<p>This query provides the details of all Batch Publisher spans. It correlates the <code>Start</code> and <code>Finish</code> checkpoints and returns the relevant key-value pairs. Traces can be filtered by uncommenting the filters at the bottom of the query and adding the corresponding filter values. For instance, you could filter tracing records related to a particular <code>EntityType</code> and <code>EntityId</code>, <code>InterfaceId</code>, <code>BatchId</code>, etc.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/8fdc65a6aefe37b50ca2fd2c5bd2bc62.js"></script>
</p>
<p>A sample response is as follows:</p>
<p><img src="/assets/img/2021/02/BatchPublisherResults.png" alt="Batch Publisher Results" width="1166" loading="lazy" style="width: 1166px;"></p>
<h2>Correlated Publisher and Subscriber Span Traces</h2>
<p>This query returns the correlated traces in the lifespan of an individual message. It correlates the <code>Start</code> and <code>Finish</code> checkpoints of both the <code>Publisher</code> and <code>Subscriber</code> spans and returns the relevant key-value pairs. Traces can be filtered by uncommenting the filters at the bottom of the query and adding the corresponding filter values, as discussed above.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/36274b15f79ca167962003435e914784.js"></script>
</p>
<p>The figures below depict a sample response.</p>
<p><img src="/assets/img/2021/02/CorrelatedTracesResults01.png" alt="CorrelatedTracesResults01" width="1183" loading="lazy" style="width: 1183px;"></p>
<p><img src="/assets/img/2021/02/CorrelatedTracesResults02.png" alt="Correlated Traces Results 02" width="1554" loading="lazy" style="width: 1554px;"></p>
<h2>Failed Traces</h2>
<p>This query returns traces with a <code>failed</code> status. As in the previous ones, traces can be filtered by uncommenting the filters at the bottom and adding the corresponding filter values.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/1fa1c640769dd3fa3c5a1098a7590660.js"></script>
</p>
<p>A sample response is depicted in the figure below.</p>
<p><img src="/assets/img/2021/02/FailedTracesResults.png" alt="Failed Traces Results" width="1601" loading="lazy" style="width: 1601px;"></p>
<h2>Message Count per Entity Type over Time</h2>
<p>This query returns the message count per <code>EntityType</code> over time which can be rendered into a chart as shown below.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/2e569a23ddd0e80b6a3c8ecf1531c807.js"></script>
</p>
<p><img src="/assets/img/2021/02/MessageCountOverTime.png" alt="Message Count Over Time" width="541" loading="lazy" style="width: 541px;"></p>
<h2>Error Count by InterfaceId and TraceEventId</h2>
<p>This query returns the error count grouped by <code>InterfaceId</code> and <code>TraceEventId</code> (<code>EventName</code>). This can be rendered into a pie or doughnut chart.</p>
<p>
<script src="https://gist.github.com/pacodelacruz/3fddcdc66a28c2a8b5717f74df8c5771.js"></script>
</p>
<p><img src="/assets/img/2021/02/ErrorCountByInterfaceAndEventType.png" alt="Error Count By Interface And Event Type" width="405" loading="lazy" style="width: 405px;"></p>
<p><img src="/assets/img/2021/02/ErrorCountByInterfaceAndEventTypeChart.gif" alt="Error Count By Interface And Event Type Chart" width="531" loading="lazy" style="width: 531px;"></p>
<h2>Considerations</h2>
<p>If you are planning to implement this approach in your solution, you need to configure the corresponding <a href="https://docs.microsoft.com/en-us/azure/azure-functions/configure-monitoring?tabs=v2#configure-sampling&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">logging sampling</a> and <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json#logging?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">log level</a> on your Azure Function according to your needs.</p>
<p>Furthermore, before using this approach, bear in mind the points below:</p>
<ul>
<li>There are <a href="https://azure.microsoft.com/en-us/pricing/details/monitor/?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">costs associated</a> with Application Insights for data ingestion and data retention. Depending on the volume of tracing events, this could influence the overall running costs of your solution.</li>
<li>Tracing data can be lost or sampled. Thus, this tracing strategy must not be utilised for auditing purposes.
<ul>
<li>Due to its asynchronous nature, Application Insights <a href="https://docs.microsoft.com/en-us/azure/azure-monitor/app/telemetry-channels#does-the-application-insights-channel-guarantee-telemetry-delivery-if-not-what-are-the-scenarios-in-which-telemetry-can-be-lost?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">cannot guarantee telemetry delivery</a>.</li>
<li>Application Insights has a <a href="https://docs.microsoft.com/en-us/azure/azure-monitor/app/pricing#limits-summary?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">daily data cap and throttles the number of requests per second</a>.</li>
<li>Application Insights also has a <a href="https://docs.microsoft.com/en-us/azure/azure-monitor/app/sampling?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">sampling configuration</a> on the backend.</li>
</ul>
</li>
<li><a href="https://docs.microsoft.com/en-us/azure/azure-monitor/app/data-retention-privacy#how-long-is-the-data-kept?WT.mc_id=AZ-MVP-5003116" rel="noopener">Data retention on Application Insights</a> must be configured based on the requirements.&nbsp;</li>
</ul>
<h2>Wrapping Up</h2>
<p>In this post, we’ve covered how we can implement a comprehensive tracing approach in Azure Functions, adding business-related metadata and leveraging their structured logging capabilities. We aimed to meet some of the common observability requirements that operations teams have when supporting distributed backend services.</p>
<p>We used an approach tailored for integration solutions with Azure Functions which follow the <a href="/enterprise-integration-patterns-on-azure-messaging-channels#publish-subscribe-channel" rel="noopener" target="_blank">publish-subscribe integration pattern</a> and the <a href="/enterprise-integration-patterns-on-azure-routing#splitter" rel="noopener" target="_blank">splitter integration pattern</a>. However, you can leverage similar principles in your own solution.</p>
<p>I hope you’ve found the series useful, and happy monitoring!</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>