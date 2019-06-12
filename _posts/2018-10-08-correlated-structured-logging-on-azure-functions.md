---
layout: post
title: Correlated Structured Logging on Azure Functions
date: 2018-10-08 19:04
author: Paco de la Cruz
comments: true
categories: [Azure, Azure Functions, Azure Log Analytics, Monitoring, Structured Logging]
---
<h2>Introduction</h2>
If you are reading this post, chances are that you know that things can go wrong and thus logging can be very useful when a solution is in production to monitor or troubleshoot it. But, not all logs are equal. While you can have enough information available, unstructured logging can be hard to read, troubleshoot or analyse. On the other hand, well-defined structured logging should allow you to search, filter and analyse your logs for better monitoring and troubleshooting. When Azure Functions run in the background, logging is even more important. Additionally, as Azure functions are meant to be single-purpose nano-services, in many scenarios, your payload would be processed by multiple functions. Therefore, a well-defined, consistent structured logging can be crucial to be able to troubleshoot a serverless-based solution spanning across multiple functions.

In this post, I’ll show how to implement correlated structured logging on Azure Functions across multiple functions with Service Bus messages using <code>ILogger</code> and Application Insights.
<h2>Scenario</h2>
To build this demo, I’ll use a scenario from <a href="https://pacodelacruzag.wordpress.com/2017/07/17/correlation-identifier-pattern-on-logic-apps/" target="_blank" rel="noopener noreferrer">Farm-To-Table</a>, a company that delivers by drones fresh produce directly from the farm to your table. They are implementing the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html" target="_blank" rel="noopener noreferrer">Pub-Sub pattern</a> to process orders. Orders are published via an http endpoint (publisher interface), then put into a topic, and finally processed by subscriber interfaces. Additionally, there are some business rules applied on the subscriber interfaces, which can have an impact on the processing. Farm-To-Table want to make sure an appropriate logging is in place, so whenever there is an issue with an order, they can troubleshoot it. They know that <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring">Application Insights</a> is the way to go for serverless logging.
<h2>Before we start</h2>
Before we start building the solution, it’s important that you understand how Azure Functions Logging with Application Insights works. I would recommend you to read the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring" target="_blank" rel="noopener noreferrer">official documentation</a>, and will only highlight some key points below.
<ul>
	<li>You can configure <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring#configure-categories-and-log-levels" target="_blank" rel="noopener noreferrer">Logging Level</a> depending on the needs.</li>
	<li>Logging is <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring#configure-the-aggregator" target="_blank" rel="noopener noreferrer">asynchronous is aggregated</a>, so you might loose some log events.</li>
	<li>You need to <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring#configure-sampling" target="_blank" rel="noopener noreferrer">configure sampling</a> according to your needs so you don’t lose important log events, but also don’t impact your app performance.</li>
	<li>Structured logging on Azure Function requires <a href="https://docs.microsoft.com/en-us/sandbox/functions-recipes/logging?tabs=csharp" target="_blank" rel="noopener noreferrer">ILogger</a>.</li>
	<li>Application Insights has a <a href="https://docs.microsoft.com/en-us/azure/application-insights/app-insights-data-retention-privacy" target="_blank" rel="noopener noreferrer">retention policy and logged data is not encrypted at rest</a>.</li>
</ul>
<h2>Good Practices for Structured Logging</h2>
As mentioned above, structured logging should ease querying and analysing logs. In order to provide these benefits, there are some practices described below that you might want to consider when implemented structured logging.
<ul>
	<li><strong>Use Event Ids</strong>: Associate a unique Id to a specific log event, so you can filter or analyse the occurrence of a particular event. Event Ids can also be used to create alerts. To make it more useful, keep a dictionary of your event ids, ideally in your code, but also accessible to the Ops team, e.g. in a wiki.</li>
	<li><strong>Standardise your logging templates</strong>. Structured logging requires a structure template. Define the templates you can use for different events. Try to keep your structured logging as consistent as possible.</li>
	<li><strong>Use a Correlation Id</strong> so different events across different components can be correlated.</li>
	<li><strong>Log a Status for relevant events</strong> so you can filter or analyse events based on a status.</li>
	<li><strong>Define the list (enumeration) of allowed values for your logging properties </strong>when it makes sense for consistent logging.</li>
	<li><strong>Log the Message or Entity Type</strong>. When you are processing different message or entity types, you want to be able to filter by type.</li>
	<li><strong>Log the relevant business information</strong>. Consider logging the minimum business information to be able to correlate log events to your business process. For instance, include the business entity id.</li>
	<li><strong>Do not log sensitive data</strong>, that you don’t want operators to see. Don’t log data that requires to be encrypted at rest when your logging platform does not support it.</li>
	<li><strong>Consider implementing logging checkpoints</strong>. When messages are processed in different steps, having defined logging checkpoints might help you to better correlate and troubleshoot events.</li>
</ul>
You don’t need to implement all these practices. Find the sweet spot so you provide the best value to the Ops team.
<h2>The Solution</h2>
Based on the scenario described above, the solution is composed by:
<div><img class=" aligncenter" src="https://platform.deloitte.com.au/hubfs/MEX%20-%20Blog%20images/MEX%20-%20Author%20-%20Paco%20de%20la%20Cruz/201809%20-%20Azure%20Functions%20Logging/010%20Arch.png" alt="010 Arch" /></div>
<div>
<ul>
	<li>One Http triggered Azure Function <em>(SubmitOrder)</em> that receives and validates the Order and then sends it to a Service Bus Queue.</li>
	<li>A Service Bus Queue used for temporal decoupling between the two functions. In a real scenario I would have used a Service Bus Topic with subscriptions to properly implement the Pub-Sub pattern, but let’s use a Queue for illustration purposes.</li>
	<li>A Service Bus Message triggered Azure Function <em>(ProcessOrder)</em> that processes the Order.</li>
	<li>Application Insights to collect all Logs and Telemetry from the Azure Functions.</li>
</ul>
<h2></h2>
<h2>The Solution Components</h2>
You can find the solution source code <a href="https://github.com/pacodelacruz/AzureFunctions.StructuredLoggingSample">here</a>. Below I will describe the different components. I hope that the code with the corresponding comments are clear and intuitive enough :)
<h3>Logging Constants</h3>
Based on the practices I mentioned above, I’ve created a <code>LoggingConstants</code> class that contains my template and enumerations for consistent structured logging.
<p/>
<script src="https://gist.github.com/pacodelacruz/358fba21870b3afd85aa137a6243c636.js"></script>
<p/>

<h3>Submit Order</h3>
The <code>SubmitOrder</code> function is triggered by an Http post and drops a message into a Service Bus message queue. Checks whether an order is valid and logs an event accordingly.
<p/>
<script src="https://gist.github.com/pacodelacruz/508590d2561478a7cc25ba21aaa0c65e.js"></script>
<p/>

<h3>Process Order</h3>
The <code>ProcessOrder</code> function is triggered by a message on a Service Bus Queue. Simulates the Order processing and logs events accordingly.
</div>

<p/>
<script src="https://gist.github.com/pacodelacruz/0959fdeab82f70e3c25f2d5ed4291fbf.js"></script>
<p/>
<h2>Querying and Analysing the Structured Logging</h2>
Once the Azure Function is running and has started logging, you can use <a href="https://docs.microsoft.com/en-us/azure/application-insights/app-insights-analytics" target="_blank" rel="noopener noreferrer">Analytics in Application Insights</a> for querying your structured logs using the <a href="https://docs.loganalytics.io/docs/Learn/Getting-Started/Getting-started-with-queries" target="_blank" rel="noopener noreferrer">Azure Log Analytics Query Language</a> (a.k.a. Kusto). Below I show some sample queries for the structure logging in the code above. I added some comments to the queries for clarification.
<h3>Get All Tracking Traces</h3>
This query returns all tracking events of our structured logging.

<p/>
<script src="https://gist.github.com/pacodelacruz/28ce16d1544c0024186099c607c33ff9.js"></script>
<p/>

A sample of the result set is as follows

<img src="https://platform.deloitte.com.au/hs-fs/hubfs/MEX%20-%20Blog%20images/MEX%20-%20Author%20-%20Paco%20de%20la%20Cruz/201809%20-%20Azure%20Functions%20Logging/020%20All%20Logging%20Events.png?width=1919&amp;name=020%20All%20Logging%20Events.png" alt="020 All Logging Events" width="1919" />
<h3>Correlated Traces</h3>
Returns correlated traces using the check points.

<p/>
<script src="https://gist.github.com/pacodelacruz/a22c1b806a2ce44e091ef91b4024e963.js"></script>
<p/>

A sample of the result set is as follows

<img src="https://platform.deloitte.com.au/hs-fs/hubfs/MEX%20-%20Blog%20images/MEX%20-%20Author%20-%20Paco%20de%20la%20Cruz/201809%20-%20Azure%20Functions%20Logging/021%20Correlated%20Events.png?width=1502&amp;name=021%20Correlated%20Events.png" alt="021 Correlated Events" width="1502" />
<h3>Correlated Traces by Entity Id</h3>
Gets the correlated traces for a particular Entity Id

<p/>
<script src="https://gist.github.com/pacodelacruz/c5a247d75f6f6fef8d6df3b0d99a47c7.js"></script>
<p/>
<h3>Occurrence of Errors</h3>
Get the occurrence of errors by Event Id

<p/>
<script src="https://gist.github.com/pacodelacruz/6cdedb1eab2eaf21e6f32d9a96ddd45f.js"></script>
<p/>

<img src="https://platform.deloitte.com.au/hs-fs/hubfs/MEX%20-%20Blog%20images/MEX%20-%20Author%20-%20Paco%20de%20la%20Cruz/201809%20-%20Azure%20Functions%20Logging/023b%20CountByEventId.png?width=1277&amp;name=023b%20CountByEventId.png" alt="023b CountByEventId" width="1277" align="center" />
<h2></h2>
<h2>Publishing Application Insights Custom Queries and Charts</h2>
You can save and publish your Application Insights custom queries for yourself or to share so other users can easily access them. You can also include them in your custom Azure Dashboard. In a <a href="https://pacodelacruzag.wordpress.com/2017/12/07/publishing-custom-queries-of-logic-apps-execution-logs/" target="_blank" rel="noopener noreferrer">previous post</a> I showed how to do it for Logic Apps, but you can so the same for your Azure Functions logging with Application Insights queries.
<h2>Wrapping Up</h2>
In this post, we saw how easy is to implement structured logging in Azure Functions using out-of-the-box capabilities with Application Insights. We also saw how to query and analyse our structure logging in Application Insights using its Query Language. I hope you have found this post handy. Please feel free to leave your feedback or questions below

Happy Logging!
<p style="text-align:center;"><em>Cross-posted on <a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz">Deloitte Platform Engineering blog</a>.
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>
