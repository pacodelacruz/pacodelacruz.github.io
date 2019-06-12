---
layout: post
title: Publishing Custom Queries of Logic Apps Execution Logs
date: 2017-12-07 21:34
author: Paco de la Cruz
comments: true
categories: [Azure, Azure iPaaS, Azure Log Analytics, Logic Apps, Monitoring]
---
In a previous post, I showed how to implement <a href="https://pacodelacruzag.wordpress.com/2017/12/01/business-activity-monitoring-on-azure-logic-apps/">Business Activity Monitoring for Logic Apps</a>. However, sometimes developers, ops, or business users want to query execution logs to get information about the processing of business messages. Whether for troubleshooting or auditing, there are some questions these personas might have, like:
<ul>
	<li>When was a business document processed?</li>
	<li>What was the content of a received document?</li>
	<li>How was that message processed?</li>
</ul>
As we saw in that <a href="https://pacodelacruzag.wordpress.com/2017/12/01/business-activity-monitoring-on-azure-logic-apps/">post</a>, we can send diagnostic log information and custom tracked properties to Azure Log Analytics. We also saw how easy is to query those logs to get information about Logic Apps execution and messages processed. Now the question is, how can we publish those custom queries, so different users can make use of them? <span style="background-color:transparent;">In this post, I’ll show one easy way to do that.</span>
<h2>1. Tracking the relevant custom properties and sending data to Log Analytics.</h2>
The first thing to do is to track the relevant custom properties we need for our queries as <strong>tracked properties</strong> in our Logic App workflow. Then you need to configure the Logic App workflow to send diagnostics information to Azure Log Analytics. You can follow the instructions on my <a href="https://pacodelacruzag.wordpress.com/2017/12/01/business-activity-monitoring-on-azure-logic-apps/">previous post</a> to perform those steps.
<h2>2. Creating the queries to get the information our users need</h2>
Once the information is being logged on Log Analytics, we need to create the queries to give the users the information they need. For that, first we need to open the Azure Log Analytics Portal. To open the portal we need to
<ul>
	<li>Go to the Log Analytics Resource on the Azure Portal</li>
	<li>Go to Log Search</li>
	<li>Click on Analytics</li>
</ul>
And now you are ready to create your own queries.

<img class="alignnone size-full wp-image-1117" src="https://pacodelacruzag.files.wordpress.com/2017/12/20-open-log-analytics.gif" alt="20 Open Log Analytics" width="2152" height="1152" />

Based on the tracked properties of the Logic App workflow shown in my <a href="https://pacodelacruzag.wordpress.com/2017/12/01/business-activity-monitoring-on-azure-logic-apps/">previous post</a>, I wrote this query to get all orders processed in the time range selected. This query returns, order number, total, date, channel, customer Id, the name of the Logic App workflow which processed this message, and the workflow run id. These last 2 columns would be quite handy for troubleshooting.

<p/>
<script src="https://gist.github.com/pacodelacruz/e8d0e9ae83ff39760e33bbe085b16900.js"></script>
<p/>

<img class="alignnone size-full wp-image-1119" src="https://pacodelacruzag.files.wordpress.com/2017/12/21a-query-and-results.png" alt="21a Query and Results" width="2211" height="993" />
<h2>3. Saving the custom Azure Log Analytics query</h2>
Once we have the query ready, we can save it and export it, so later we can publish it. To do that, we follow the steps below
<ul>
	<li>Click the Save button</li>
	<li>Give a name and category to the query. The category is quite useful for searching among all saved queries.</li>
	<li>Then we click the Export button</li>
	<li>And select the Share a Link to Query option, so the link to the query is saved in the clipboard.</li>
</ul>
<img class="alignnone size-full wp-image-1118" src="https://pacodelacruzag.files.wordpress.com/2017/12/21-save-and-export-query.gif" alt="21 Save and Export Query" width="2184" height="1152" />
<h2>4. Publishing the custom Azure Log Analytics query to the users</h2>
After we have gotten the link to the query, we can publish it in the same Dashboard we created for our BAM charts described in my <a href="https://pacodelacruzag.wordpress.com/2017/12/01/business-activity-monitoring-on-azure-logic-apps/">previous post</a>. We need to:
<ul>
	<li>Edit the shared Azure Dasboard</li>
	<li>Add the Markdown tile.</li>
	<li>Add the <a href="https://en.wikipedia.org/wiki/Markdown">markdown text</a> which contains the link to the query created above.</li>
</ul>
Now the users will have all the charts and custom queries they need in one single place!

<img class="alignnone size-full wp-image-1120" src="https://pacodelacruzag.files.wordpress.com/2017/12/22-add-query-to-dashboard.gif" alt="22 Add Query to Dashboard" width="2776" height="1448" />
<h2>Making easier to users to get to the workflow run logs of a particular business message on Logic Apps.</h2>
Logic Apps provide a <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms">very detailed view</a> of the execution of the workflows. However, I’ve been asked so many times to make easier to users to get the run details of a particular business message. Here is a tip on how to do it.

First, we need to create a query to get the runId of the workflow instance that processed the message. I created this query to get those details for the orders submitted by a particular user.
<ul>
	<li>Once we have that query, we publish it to the same markdown tile in our dashboard.</li>
	<li>We also add the link to the workflow Azure resource to the same dashboard tile.</li>
</ul>
Now users can query the orders submitted by a user, get the workflow run id, and get the workflow run details in very few clicks.

<img class="alignnone size-full wp-image-1121" src="https://pacodelacruzag.files.wordpress.com/2017/12/23-query-logic-app-instance-by-business-id.gif" alt="23 Query Logic App Instance by Business Id" width="2776" height="1500" />
<h2>Wrapping-up</h2>
In this post, we’ve seen how to create and publish custom queries of Logic Apps execution logs. We’ve also seen how to make easier to users to get the workflow run details of the processing of a particular business message. Now you should be ready to start creating and publishing your own custom queries and creating amazing monitoring and tracking dashboards for your Logic Apps solutions.

I hope you’ve got some useful tips from this post and you’ve enjoyed it. Feel free to leave your questions or comments below,
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
