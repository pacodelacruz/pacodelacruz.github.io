---
layout: post
title: Business Activity Monitoring on Azure Logic Apps with Azure Log Analytics
date: 2017-12-01 19:19
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Azure, Azure iPaaS, Business Activity Monitoring, iPaaS, Logic Apps, Microsoft iPaaS, Monitoring]
---
<h2>Introduction</h2>
Azure Logic Apps provide <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms">built-in monitoring tools</a> that allow you to check the run history (including all inputs and outputs of triggers and actions), trigger history, status, performance, etc. Additionally, you can <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps">enable diagnostic logging on your Logic Apps</a> and send all these runtime details and events to <a href="https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-overview">Azure Log Analytics</a>. You can also install the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms">Logic Apps Management Solution</a> on OMS, which gives you a very rich aggregated view and charts of all your logic apps that are being monitored.

All these tools are great for developers or system administrators, who want to monitor and troubleshoot the execution of the workflows. However, sometimes we need to track and monitor more business-related information. Additionally, it's quite common that business users want to monitor, with a business perspective, what's happening at the integration layer.

In this post, I will show how to implement tracking capabilities for business-related information and how to create a <a href="https://en.wikipedia.org/wiki/Business_activity_monitoring">Business Activity Monitoring</a> (BAM) dashboard for Logic Apps.
<h2>Scenario</h2>
In a <a href="https://platform.deloitte.com.au/articles/correlation-identifier-pattern-on-logic-apps">previous post</a>, I introduced a fictitious company called "Farm to Table", which provides fresh produce drone delivery. This company has been leveraging Logic Apps to implement their business processes, which integrate with multiple systems on the cloud. As part of their requirements they need to monitor the business activity flowing through this integration solution.

"Farm to Table" want to be able to monitor the orders they are receiving per channel. At the moment, customers can place orders via SMS, a web online store, and a mobile app. They also want to be able to track orders placed using customer number and order number.
<h2>Solution</h2>
<h3>Prerequisites</h3>
To be able to track and monitor business properties and activity on Logic Apps, there are some prerequisites:
<ol>
	<li><strong>Azure Log Analytics workspace</strong>. We require an Azure Log Analytics workspace. We can use a workspace previously created or create a new one following the steps described <a href="https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-quick-create-workspace">here</a>.</li>
	<li><strong>Enable diagnostic logging to Azure Log Analytics on the Logic App</strong> which processes the messages we want to track, following the instructions detailed <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps">here</a>.</li>
	<li>Install the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms">Logic App Management solution for OMS</a> on the Azure Log Analytics.</li>
</ol>
<h3>Adding Tracking Properties to the Logic App Workflow</h3>
Once enabled diagnostic logging, by default, Logic Apps tracks all workflow instances and actions on Azure Log Analytics. This tracking can very easily be extended using Tracked Properties on the workflow actions. In the tracked properties, we can include business related data; for example, in this scenario we could track customer Id, order number, order total, and channel.

"Farm To Table" has implemented an HTTP Triggered Logic App workflow that receives orders from different channels, validates the orders, maps it to a canonical model, enriches the message, and then puts them into a Service Bus Queue. The order canonical model processed by this workflow follows the schema of the instance below:

<p/>
<script src="https://gist.github.com/pacodelacruz/1208a35e954c5513fb65989cc9e35d1f.js"></script>
<p/>

To track business properties of the orders, we will add the tracked properties to the action that sends the message to Service Bus. It's noteworthy that when we use tracked properties within a Logic App action, we can only use the trigger input and the action's inputs and outputs.

In the Logic App action that sends the message into a Service Bus queue, we will add the tracked properties to include the customer Id, order number, order Total, date, and channel. I'm also adding a flag to simplify my queries, but that's optional. The code below shows the <strong><em>trackedProperties</em></strong> section added to our workflow action.

<p/>
<script src="https://gist.github.com/pacodelacruz/6bda9478f6297279c87d97e9c9dd66ec.js"></script>
<p/>

Once we have started tracking those properties and we have information already logged on Azure Log Analytics, we can start querying and creating charts for our own Business Activity Monitoring Dashboard.
<h3>Querying Azure Log Analytics</h3>
Let's start querying what we are tracking from the Logic Apps on Log Analytics. Bear in mind that there is a delay between the execution of the Logic App and the log being available on Log Analytics. Based on my experience, it usually takes anywhere between 2 and 10 minutes. You can find detailed documentation on the Log Analytics query language <a href="https://docs.loganalytics.io/docs/Language-Reference">here</a>.

The query below returns the custom data I'm tracking on the Logic App workflow. When building your queries, it's worth noting that
<ul>
	<li>Workflow actions are being logged using the <strong>AzureDiagnostics</strong> Log Type and with the <strong>WorkflowRuntime</strong> category.</li>
	<li>Logic Apps prepend the prefix "trackedProperties_" to each property and append a suffix to declare its type.</li>
</ul>
<p/>
<script src="https://gist.github.com/pacodelacruz/1a63afad952e7566c743dd14ee6deb64.js"></script>
<p/>

This query should return a result set as the one below:

<img src="/assets/img/2017/12/120117_0738_businessact1.png" alt="" />

Additionally, we can add filters to our queries, for instance, to get all orders by the CustomerId, we could use a query as follows:

<p/>
<script src="https://gist.github.com/pacodelacruz/421c7dd7f79c265f4e6235e266d9f502.js"></script>
<p/>
<h3>Creating a Monitoring Dashboard.</h3>
Before, Microsoft suggested to create OMS custom views and queries for this purpose. The steps to create a custom OMS dashboard for Logic Apps are described <a href="https://blogs.msdn.microsoft.com/logicapps/2017/11/03/monitor-logic-app-action-inputsoutputs-in-log-analytics-oms-using-tracked-properties/">here</a>. However, after the upgrade of OMS to the new Log Analytics Query Language (previously known as Kusto query language), the recommended approach is now to use the new Azure Log Analytics portal, create queries and charts, and pin them to a shared Azure Dashboard. If have created both, custom OMS dahsboards and custom Log Analytics Azure Dashboards, you must agree that shared Azure Dashboards and Log Analytics charts are much more user friendly.

The steps to create and shared an Azure Dashboard to include Log Analytics data and charts are described <a href="https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-tutorial-dashboards">here</a>. We will follow these steps to create our own Business Activity Monitoring Dashboard as a shared Azure Dashboard. I won't repeat what's in Microsoft's documentation, I'll just show how I'm creating the charts to be added in the Dashboard.

<strong>Order Count by Channel
</strong>

"Farm to Table" want to have a chart with the order count summarised by channel for the last 7 days in their BAM Dashboard. The query below returns those details.

<p/>
<script src="https://gist.github.com/pacodelacruz/e1d441497a6c707c6c9916c5654a60b9.js"></script>
<p/>

Once we get the results, we need to select Chart and then the Doughnut option. After that, we are ready to pin our chart to the Azure shared dashboard.

<img class="alignnone size-full wp-image-1106" src="/assets/img/2017/12/15-chart-count-by-channel.gif" alt="15 Chart Count By Channel" width="1528" height="876" />

<strong>Order Total by Date and Channel
</strong>

The company also want to have a chart with the order total summarised by date and channel for the last 7 days in their Dashboard. The query below returns those details.

<p/>
<script src="https://gist.github.com/pacodelacruz/85570eccb8ef95c28880225cc3b627e4.js"></script>
<p/>

Once we get the results, we need to select Chart and then Stacked Columns. After that, we are ready to pin our chart to the Azure shared dashboard.

<img class="alignnone size-full wp-image-1107" src="/assets/img/2017/12/16-chart-total-by-date.gif" alt="16 Chart Total By Date" width="1528" height="876" />
<h3>Business Activity Monitoring Dashboard</h3>
Once we have pinned our charts, we would be able to see them in our Azure shared dashboard. These charts are very handy and allow us to dynamically visualise the data as shown below.

<img class="alignnone size-full wp-image-1108" src="/assets/img/2017/12/20-bam-dashboard.gif" alt="20 BAM Dashboard" width="1696" height="656" />
<h2>Wrapping-up</h2>
In this post, we've seen how to easily track and monitor business information flowing through our Logic Apps, using Logic App native integration with OMS and Azure Log Analytics. Additionally, we've seen how friendly and cool the Log Analytics charts are. This gives Logic Apps another great competitive advantage as an enterprise-grade integration Platform as a Service (iPaaS) in the market.

I hope you've learned something useful and enjoyed this post! Feel free to post your comments or questions below

Happy clouding!

Paco
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
