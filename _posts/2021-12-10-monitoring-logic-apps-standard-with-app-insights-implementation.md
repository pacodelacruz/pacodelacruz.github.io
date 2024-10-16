---
layout: post
title: Monitoring Logic Apps Standard with Application Insights – Implementation
date: 2021-12-10 10:00
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Logic Apps, Application Insights, Azure, Observability]
---

<p><img src="/assets/img/2021/12/100-astronaut.webp" alt="Monitoring Logic Apps Standard with Application Insights - Implementation" width="800" loading="lazy" style="width: 800px;"></p>
<h2>Overview</h2>
<p>In the <a href="/monitoring-logic-apps-standard-with-app-insights-intro" rel="noopener" target="_blank">previous post</a> of the series, we’ve covered the different built-in observability features that are available in Logic Apps Standard and how many of them are part of the traces sent to Application Insights. In this post, we’ll discuss how these features can be implemented in a sample scenario. The series is structured as outlined below:</p>
<!--more-->
<ol>
<li><a href="/monitoring-logic-apps-standard-with-app-insights-intro" rel="noopener" target="_blank"><strong>Introduction </strong></a>&nbsp;– describes the built-in observability features available in Logic Apps Standard.</li>
<li><strong>Sample implementation (this article)</strong> – shows how these features can be leveraged and implemented.</li>
<li><a href="/monitoring-logic-apps-standard-with-app-insights-querying" rel="noopener" target="_blank"><strong>Querying and analysing Logic Apps traces</strong></a> – shows how to query and analyse Logic Apps application traces, and how to publish and share queries and charts.</li>
</ol>
<p>The code of the solution shown in this post is available <a href="https://github.com/pacodelacruz/observability-pubsub-logicapps/blob/main/user-updated-pub/workflow.json" rel="noopener" target="_blank">on GitHub</a>.</p>
<h2>Scenario</h2>
<p>To demonstrate how to leverage these observability features in Logic Apps, I’ll use a very common integration scenario: the publishing and consumption of user updated events. Think of an HR or CRM system pushing user updated events via webhooks for downstream systems to consume.</p>
<p>To better illustrate the scenario, let’s describe the different components of the end-to-end solution.</p>
<p><img src="/assets/img/2021/12/101-participants-01.webp" alt="Sample scenario components" width="1621" loading="lazy" style="width: 1621px;"></p>
<p style="text-align: center;">Figure 1. Sample scenario components</p>
<p style="text-align: center;">&nbsp;</p>
<ul>
<li><strong>Webhook</strong> - an HR or CRM system pushing user updated events via webhooks as an array of events and expects an HTTP response.</li>
<li><strong>Logic Apps workflow publisher</strong>&nbsp;- receives the HTTP POST request from the webhook, validates the batch message, splits the message into individual event messages, publishes the messages into a Service Bus queue, and returns the corresponding HTTP response to the webhook.</li>
<li><strong>User updated Service Bus queue</strong> - used to send and receive individual user updated event messages.</li>
<li><strong>User updated Logic App workflow subscriber</strong>&nbsp;– listens to messages in the queue, performs the required validations and message processing, and delivers the message to Azure blob storage.</li>
<li><strong>Azure blob storage</strong>&nbsp;– simulates a target system that needs to be notified when users are updated, but requires an integration layer to do some validation, processing, transformation, and/or custom delivery.</li>
<li><strong>Application Insights</strong>&nbsp;– ingests all diagnostic traces from Logic App workflows.</li>
</ul>
<h2>Publisher Workflow</h2>
<p>The figure below shows the Logic App publisher workflow.</p>
<p><img src="/assets/img/2021/12/110-workflow-pub-00.webp" alt="Publisher workflow" width="766" loading="lazy" style="width: 766px;"></p>
<p style="text-align: center;">Figure 2. Publisher workflow</p>
<p style="text-align: center;">&nbsp;</p>
<p>The workflow’s steps are described as follows:</p>
<ol>
<li>The workflow is triggered with an HTTP request.</li>
<li>An <code>InterfaceId</code> is tracked using <code>trackedProperties</code>.</li>
<li>The request payload is parsed to validate the payload against the expected schema.</li>
<li>If the received payload is valid
<ol style="list-style-type: lower-alpha;">
<li>For each individual event,
<ol style="list-style-type: lower-roman;">
<li>The individual event message is parsed so that the different elements can be used further in the designer.</li>
<li>The individual event message is sent to Service Bus</li>
</ol>
</li>
<li>If all individual messages were successfully sent to Service Bus
<ol style="list-style-type: lower-roman;">
<li>Return a 202 Accepted response.</li>
</ol>
</li>
<li>Otherwise,
<ol style="list-style-type: lower-roman;">
<li>Return a 500 Internal Server error response</li>
<li>Terminate the workflow as failed with a custom error code and error message.</li>
</ol>
</li>
</ol>
</li>
<li>If the received payload is invalid
<ol style="list-style-type: lower-alpha;">
<li>Return HTTP 400 Bad Request</li>
<li>Terminate the workflow as failed with a custom error code and error message.</li>
</ol>
</li>
</ol>
<p>The code behind the workflow can be explored <a href="https://github.com/pacodelacruz/observability-pubsub-logicapps/blob/main/user-updated-pub/workflow.json" rel="noopener" target="_blank">on GitHub</a>.</p>
<p>The observability practices implemented in this workflow are detailed below and follow the practices described in the previous post.</p>
<ol>
<li>Application Insights integration is enabled for the Logic App.</li>
</ol>
<img src="/assets/img/2021/12/071-app-insights.webp" alt="Enabling Application Insights integration" width="822" loading="lazy" style="width: 822px;"><br>
<p style="text-align: center;">Figure 3. Enabling Application Insights integration</p>
<p style="text-align: center;">&nbsp;</p>
<ol start="2">
<li>A custom tracking id is being logged using a property in the JSON trigger body. This configuration and and code behind are shown below.</li>
</ol>
<p><img src="/assets/img/2021/12/031-custom-tracking-id.webp" alt="Configuring the custom tracking id property" width="1149" loading="lazy" style="width: 1149px;"></p>
<p style="text-align: center;">Figure 4. Configuring the custom tracking id property</p>
<p style="text-align: center;">&nbsp;</p>
<pre>"triggers": {<br> &nbsp;&nbsp;&nbsp;"manual":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"correlation":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"clientTrackingId":&nbsp;"@{coalesce(triggerBody()?['id'],&nbsp;guid())}"<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"inputs":&nbsp;{},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"kind":&nbsp;"Http",<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"type":&nbsp;"Request"<br> &nbsp;&nbsp;&nbsp;}<br>}</pre>
<p style="text-align: center;">Code snippet 1. Configuring <code>clientTrackingId</code> in a HTTP trigger</p>
<p style="text-align: center;">&nbsp;</p>
<ol start="3">
<li>Tracked properties in a <code>Compose</code> action are being used to log an interface identifier. The code behind this action is shown below.</li>
</ol>
<pre>"Trace_workflow_metadata": {<br> &nbsp;&nbsp;&nbsp;"inputs":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"InterfaceId":&nbsp;"USER.SVC01.P01v1"<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;"runAfter":&nbsp;{},<br> &nbsp;&nbsp;&nbsp;"trackedProperties":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"InterfaceId":&nbsp;"@{outputs('Trace_workflow_metadata')?['InterfaceId']}"<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;"type":&nbsp;"Compose"<br>}</pre>
<p style="text-align: center;">Code snippet 2. Configuring <code>trackedProrperties</code> in a <code>compose</code> action</p>
<p style="text-align: center;">&nbsp;</p>
<ol start="4">
<li><code>RunAfter</code> setting is being used for error handling. The code view of an action implementing <code>runAfter</code> is included below.</li>
</ol>
<pre>"Return_400_BadRequest_response": {<br> &nbsp;&nbsp;&nbsp;"inputs":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"body":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"ActivityId":&nbsp;"@{workflow().run.name}",<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Message":&nbsp;"Bad&nbsp;request.&nbsp;Invalid&nbsp;message.",<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"StatusCode":&nbsp;400<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"statusCode":&nbsp;400<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;"kind":&nbsp;"http",<br> &nbsp;&nbsp;&nbsp;"runAfter":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Parse_Request_Payload":&nbsp;[<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Failed"<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;"type":&nbsp;"Response"<br>}</pre>
<p style="text-align: center;">Code snippet 3. Configuring error handling using <code>runAfter</code></p>
<p style="text-align: center;">&nbsp;</p>
<ol start="5">
<li>When applicable, a <code>terminate</code> action with <code>failed</code> status is used adding a custom error code and error message. The code view of a <code>terminate</code> action used in this workflow is shown below.</li>
</ol>
<pre>"Terminate_as_Failed_(PublisherReceiptFailed_InvalidMessage)": {<br> &nbsp;&nbsp;&nbsp;"inputs":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"runError":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"code":&nbsp;"PublisherReceiptFailed_InvalidMessage",<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"message": "Invalid HTTP request payload received in publisher interface."<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"runStatus":&nbsp;"Failed"<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;"runAfter":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Return_400_BadRequest_response":&nbsp;[<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Succeeded"<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;"type":&nbsp;"Terminate"<br>}</pre>
<p style="text-align: center;">Code snippet 4. Using a <code>terminate</code> action with custom error code and error message</p>
<h2>Subscriber Workflow</h2>
<p>The figure below shows the Logic App subscriber workflow.</p>
<p><img src="/assets/img/2021/12/111-workflow-sub-00.webp" alt="Subscriber workflow" width="581" loading="lazy" style="width: 581px; margin-left: auto; margin-right: auto; display: block;"></p>
<p style="text-align: center;">Figure 5. Subscriber workflow</p>
<p style="text-align: center;">&nbsp;</p>
<p style="text-align: justify;">The workflow steps are described as follows.</p>
<ol>
<li style="text-align: justify;">The workflow is triggered by a message in a Service Bus queue.</li>
<li>An <code>InterfaceId</code> is tracked using <code>trackedProperties</code>.</li>
<li>The event message is parsed so that the different elements can be used in the designer.</li>
<li>Based on some values in the payload, the message can be delivered to the target system (blob) or a failed attempt can be simulated.</li>
<li>If a failed attempt is to be simulated
<ol style="list-style-type: lower-alpha;">
<li>Depending on the content, the workflow is being terminated with different error codes.</li>
<li>In some cases, the message is immediately dead-lettered.</li>
<li>In some other cases, the message is being completed.</li>
<li>In some other cases, the message is not settled, so that it becomes available for processing again after the lock expires.</li>
<li>In some of these cases, the workflow is terminated as failed, and the delivery count is included in the custom error message.</li>
</ol>
</li>
<li>Otherwise,
<ol style="list-style-type: lower-alpha;">
<li>The message is delivered to Azure blob (target system)</li>
<li>The message is completed from the Service Bus queue.</li>
</ol>
</li>
</ol>
<p>The code behind the workflow can be explored <a href="https://github.com/pacodelacruz/observability-pubsub-logicapps/blob/main/user-updated-sub-blob/workflow.json" rel="noopener" target="_blank">on GitHub</a>.</p>
<p>&nbsp;</p>
<p>The observability practices implemented in this workflow are detailed below and follow the practices described in the previous post.</p>
<ol>
<li>As shown previously, Application Insights integration is enabled for the Logic App.</li>
<li>A custom tracking id is being logged using a property in the JSON trigger body. The code behind the trigger is shown below.</li>
</ol>
<pre>"triggers": {<br> &nbsp;&nbsp;&nbsp;"When_a_message_is_received_in_a_queue_(peek-lock)":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"correlation":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"clientTrackingId":&nbsp;"@{if(empty(triggerBody()),&nbsp;guid(),&nbsp;triggerBody()['Properties']?['ClientTrackingId'])}"<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"inputs":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"host":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"connection":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"referenceName":&nbsp;"servicebus"<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"method":&nbsp;"get",<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"path":&nbsp;"/@{encodeURIComponent(encodeURIComponent('user-updated'))}/messages/head/peek",<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"queries":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"queueType":&nbsp;"Main"<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"recurrence":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"frequency":&nbsp;"Second",<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"interval":&nbsp;10<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"type":&nbsp;"ApiConnection"<br> &nbsp; }<br>}</pre>
<p style="text-align: center;">Code snippet 5. Configuring <code>clientTrackingId</code> for a Service Bus trigger</p>
<p style="text-align: center;">&nbsp;</p>
<ol start="3">
<li>As in the publisher workflow, <code>trackedProperties</code> in a <code>compose</code> action are used to log an interface identifier. The code behind this action is shown below.</li>
</ol>
<pre>"Trace_workflow_metadata": {<br> &nbsp;&nbsp;&nbsp;"inputs":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"InterfaceId":&nbsp;"USER.SVC01.S01v1"<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;"runAfter":&nbsp;{},<br> &nbsp;&nbsp;&nbsp;"trackedProperties":&nbsp;{<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"InterfaceId":&nbsp;"@{outputs('Trace_workflow_metadata')?['InterfaceId']}"<br> &nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;"type":&nbsp;"Compose"<br>}</pre>
<p style="text-align: center;">Code snippet 6. Configuring <code>trackedProperties</code> in a <code>compose</code> action</p>
<p style="text-align: center;">&nbsp;</p>
<ol start="4">
<li>Similarly, the <code>runAfter</code> setting is used for error handling.</li>
<li>Likewise, when applicable, a <code>terminate</code> action with <code>failed</code> status is used with a custom error code and error message.</li>
</ol>
<h2>Wrapping-Up</h2>
<p>In this post, I’ve described a sample implementation of a publish-subscribe scenario using Logic App Standard and how the different observability practices described previously can be implemented in the corresponding workflows. In the <a href="/monitoring-logic-apps-standard-with-app-insights-querying" rel="noopener" target="_blank">next and final post of the series</a>, I’ll describe how the different traces that result from these practices can be queried and analysed using <a href="https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Kusto Query Language</a>.</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>