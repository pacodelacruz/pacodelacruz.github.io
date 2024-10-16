---
layout: post
title: Monitoring Logic Apps Standard with Application Insights – Introduction
date: 2021-12-07 10:00
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Logic Apps, Application Insights, Azure, Observability]
---

​<img src="/assets/img/2021/12/001-rocket.webp" alt="Monitoring Logic Apps Standard with Application Insights – Introduction" width="800" loading="lazy" style="width: 800px;">
<h2>Overview</h2>
<p>As I <a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">previously discussed</a>, it’s very common that developers tend to focus their efforts on building and shipping applications and services to production but forget to consider what happens after go-live. It’s easy to dismiss that the solutions being built are going to be supported by someone else. Once the services reach production, an operations team will need the means to understand the system’s state and health to support it efficiently. This is even more relevant when dealing with distributed systems that run in the background. Thus, the importance of implementing proper observability practices in our solutions to allow monitoring, troubleshooting, analysis, and alerting.</p>
<!--more-->
<p>I strongly believe in the importance of application-level observability practices. In the past, I’ve written about <a href="/business-activity-monitoring-on-azure-logic-apps" rel="noopener" target="_blank">activity monitoring on Logic Apps</a>, and <a href="/publishing-custom-queries-of-logic-apps-execution-logs" rel="noopener" target="_blank">custom queries of Logic Apps logs</a>. I’ve also covered an approach for <a href="/correlated-structured-logging-on-azure-functions" rel="noopener" target="_blank">correlated structured logging on Azure Functions</a> and wrote a <a href="/distributed-tracing-and-observability-with-azure-functions-1-intro" rel="noopener" target="_blank">series on a custom distributed tracing in Azure Functions</a>. My previous posts on Logic Apps targeted the consumption SKU, which provided integration with Log Analytics.</p>
<p>With the <a href="https://techcommunity.microsoft.com/t5/azure-developer-community-blog/azure-logic-apps-announcement-ga-of-single-tenant-standard-sku/ba-p/2382460" rel="noopener" target="_blank">release of Logic Apps Standard</a>, now Logic Apps run on top of the Azure Functions runtime. This means that traces are sent to Application Insights. Considering these new capabilities, I wanted to explore what can be leveraged on Logic Apps Standard to provide application-level observability.</p>
<p>Throughout this series, I want to share what I’ve found. I aim to provide insights so that architects and developers can incorporate observability practices when designing and building Logic Apps solutions. The series is structured as outlined below:</p>
<ol>
<li><strong>Introduction (this article)</strong> – describes the built-in observability features available in Logic Apps Standard.</li>
<li><a href="/monitoring-logic-apps-standard-with-app-insights-implementation" rel="noopener" target="_blank"><strong>Reference implementation</strong></a> – shows how these features can be leveraged and implemented.</li>
<li><a href="/monitoring-logic-apps-standard-with-app-insights-querying" rel="noopener" target="_blank"><strong>Querying and analysing Logic Apps traces</strong></a> – shows how to query and analyse Logic Apps application traces, and how to publish and share queries and charts.</li>
</ol>
<p>This series describes in detail what I recently presented at the Azure Serverless Conference.</p>
<p>&nbsp;</p>
<div class="hs-responsive-embed-wrapper hs-responsive-embed" style="width: 100%; height: auto; position: relative; overflow: hidden; padding: 0; max-width: 560px; max-height: 315px; min-width: 256px; margin: 0px auto; display: block;">
<div class="hs-responsive-embed-inner-wrapper" style="position: relative; overflow: hidden; max-width: 100%; padding-bottom: 56.25%; margin: 0;"><iframe class="hs-responsive-embed-iframe" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;" title="YouTube video player" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" xml="lang" src="https://www.youtube.com/embed/1rJZ-kA2wyQ" width="560" height="315" frameborder="0" allowfullscreen="allowfullscreen" data-service="youtube"></iframe></div>
</div>
<h2>Workflow Run, Trigger History, and Run Identifier</h2>
<p>One of the most basic monitoring capabilities of Logic Apps is its built-in <a href="https://docs.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps#review-runs-history?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">workflow run history</a> and <a href="https://docs.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps#review-trigger-history?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">trigger history</a>. At the time of writing, the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps#review-runs-history?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">available documentation</a> still refers to Logic Apps consumption SKU. However, most of these features are applicable to Logic Apps Standard as well. The figures below depict the run and trigger history of a workflow.</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/12/011-run-history.webp" alt="Logic Apps run history" width="856" loading="lazy" style="width: 856px;"></p>
<p style="text-align: center;">Figure 1. Logic Apps run history</p>
<p style="text-align: center;">&nbsp;</p>
<p style="text-align: center;"><img src="/assets/img/2021/12/012-trigger-history.webp" alt="Logic Apps trigger history" width="856" loading="lazy" style="width: 856px;"></p>
<p style="text-align: center;">Figure 2. Logic Apps trigger history</p>
<p>&nbsp;</p>
<p>It is worth noting that all Logic App workflow run instances have a unique workflow run identifier, which is accessible at run time using the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference#workflow?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><code>workflow().run.name</code></a> function. This run identifier is the one that is shown in the run history and allows to run instances to be searched by it, as shown in the picture below.</p>
<p>The instance run, trigger history, and run identifiers are sent to Application Insights traces and can be queried and analysed. I’ll show more on this in the last instalment of the series.</p>
<p><img src="/assets/img/2021/12/013-run-history-search.webp" alt="Search by run instance identifier" width="605" loading="lazy" style="width: 605px;"></p>
<p style="text-align: center;">Figure 3. Search by run instance identifier</p>
<h2>Trigger and Action Inputs and Outputs</h2>
<p>In Logic Apps Standard, workflows can be configured to be stateless or stateful. When a workflow is configured to be&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare#stateful-and-stateless-workflows?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">stateful</a>, the run view of an instance shows all inputs and outputs for the trigger and every action. It’s worth mentioning that these inputs and outputs are not sent to Application Insights traces but kept in the storage provider configured, typically a storage account.</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/12/021-inputs-outputs.webp" alt="Inputs and outputs of Logic Apps actions" width="1096" loading="lazy" style="width: 1096px;"></p>
<p style="text-align: center;">Figure 4. Inputs and outputs of Logic Apps actions</p>
<p style="text-align: center;">&nbsp;</p>
<blockquote>
<p>Tip: when processing confidential messages and these must be obfuscated to operators, inputs and outputs of the actions processing these messages must be secured. This can be configured per action as shown in the figure below. More information on this topic can be found in the&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-securing-a-logic-app?tabs=azure-portal#secure-data-in-run-history-by-using-obfuscation&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">official documentation</a>.&nbsp;</p>
</blockquote>
<p><img src="/assets/img/2021/12/022-secure-in-out.webp" alt="Securing action inputs and outputs" width="1149" loading="lazy" style="width: 1149px;"></p>
<p style="text-align: center;">Figure 5. Securing action inputs and outputs</p>
<h2>Custom Tracking Id</h2>
<p>In addition to the workflow run unique identifier, a custom tracking identifier (<span style="color: #2b579a; background-color: #e6e6e6;"><a href="https://docs.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps-log-analytics#azure-monitor-diagnostics-events?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><code>clientTrackingId</code></a></span>)&nbsp;can be assigned to each workflow run. The custom tracking identifier is configured at design time in the settings tab of the workflow trigger as shown in the figure below.&nbsp;</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/12/031-custom-tracking-id.webp" alt="Setting a custom tracking id" width="1149" loading="lazy" style="width: 1149px;"></p>
<p style="text-align: center;">Figure 6. Setting a custom tracking id</p>
<p style="text-align: center;">&nbsp;</p>
<blockquote>
<p>Tip: when configuring the custom tracking id property, make sure that you are using a defensive approach to avoid trigger failures. Some examples are described below.&nbsp;</p>
</blockquote>
<ul>
<li>
<p>Always use the&nbsp;‘<code>?</code>’ operator when reading object properties to define the custom tracking id. Otherwise, a non-existing property would cause the trigger to fail.&nbsp;</p>
</li>
<li>
<p><span style="background-color: transparent;">For cases when the </span><code>triggerBody</code><span style="background-color: transparent;"> can be null, e.g. in an HTTP trigger, use the&nbsp;</span><a href="https://docs.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference#coalesce?WT.mc_id=AZ-MVP-5003116" style="background-color: transparent;"><code>coalesce</code></a><span style="background-color: transparent;">&nbsp;function and define a default value. Otherwise, a null trigger body would cause the trigger to fail, e.g.</span></p>
</li>
</ul>
<pre>"correlation": {<br>  &nbsp; "clientTrackingId": "@{coalesce(triggerBody()?['id'], guid()))"<br>}</pre>
<p style="text-align: center;">Code snippet 1. Setting a clientTrackingId property with coalesce</p>
<ul>
<li>For those cases when the <code>triggerBody</code> can be empty, for instance when using the Service Bus trigger, implement the corresponding validation and set a default value, e.g.</li>
</ul>
<pre>"correlation": {<br>  &nbsp; "clientTrackingId": "@{if(empty(triggerBody()), guid(), triggerBody()['Properties']?['ClientTrackingId'])}"<br>}</pre>
<p style="text-align: center;">Code snippet 2. Setting a clientTrackingId property and validating an empty trigger</p>
<p>Once the client tracking id is configured for the workflow, the run view shows the value for the instance as depicted in the figure below.&nbsp;Furthermore, this identifier is available on Application Insights traces, so you can search or filter using this identifier. I’ll show more on this capability later in this series.</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/12/032-custom-tracking-id-run.webp" alt="Inspecting the client tracking id in a run view" width="1096" loading="lazy" style="width: 1096px;"></p>
<p style="text-align: center;">Figure 7. Inspecting the client tracking id in a run view</p>
<h2>Tracked Properties</h2>
<p><a href="https://docs.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps-log-analytics#azure-monitor-diagnostics-events?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Tracked properties (<code>trackedProperties</code>)</a>&nbsp;allow tracking&nbsp;inputs&nbsp;and outputs of an action in the traces sent to Application Insights. Tracked properties are configured in the settings tab of an action or in the corresponding property in the JSON code behind as shown below.&nbsp;</p>
<blockquote>
<p>Tip: given that tracked properties can only track a single action's inputs and outputs, a simple way to track multiple properties for a workflow in one action is to use a&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-perform-data-operations#compose-action?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">compose&nbsp;</a>action to compose a JSON object with all the properties that must be tracked and track the outputs of such action. This approach is shown in the figure and the corresponding JSON code below.</p>
</blockquote>
<p><img src="/assets/img/2021/12/041-tracked-properties-inputs.webp" alt="Using a Compose action to track properties" width="996" loading="lazy" style="width: 996px;"></p>
<p style="text-align: center;">Figure 8. Using a Compose action to track properties</p>
<p style="text-align: center;">&nbsp;</p>
<p style="text-align: center;"><img src="/assets/img/2021/12/042-tracked-properties-settings.webp" alt="Configuring tracked properties" width="984" loading="lazy" style="width: 984px;"></p>
<p style="text-align: center;">Figure 9. Configuring tracked properties</p>
<p style="text-align: center;">&nbsp;</p>
<pre>"Trace_workflow_metadata": {<br> &nbsp; "inputs": {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "InterfaceId": "USER.SVC01.P01.v1"<br> &nbsp; },<br> &nbsp; "runAfter": {},<br> &nbsp; "trackedProperties": {<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "InterfaceId": "@{outputs('Trace_workflow_metadata')?[InterfaceId]}"<br> &nbsp; },<br> &nbsp; "type": "Compose"<br>}</pre>
<p style="text-align: center;">Code snippet 3. Configuring trackedProperties</p>
<h2>Exception Handling</h2>
<p>To implement exception handling in Logic Apps, we can make use of the <code>runAfter</code>&nbsp;property for an&nbsp;action to run after a previous action or scope has a statuses of&nbsp;"<code>Failed</code>",&nbsp;"<code>Skipped</code>", or&nbsp;"<code>TimedOut</code>". This is described in more detail&nbsp;in the&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling#catch-and-handle-failures-by-changing-run-after-behavior?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">official documentation</a>. Additionally, <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-run-steps-group-scopes?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">scopes</a> can be used to structure the workflow in a way to have one action that handles exceptions at a scope level.&nbsp;The figures below show exception handling in a workflow.</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/12/051-run-after.webp" alt="Exception handling using runAfter configuration" width="815" loading="lazy" style="width: 815px;"></p>
<p style="text-align: center;">Figure 10. Exception handling using <code>runAfter</code> configuration</p>
<p style="text-align: center;">&nbsp;</p>
<p style="text-align: center;"><img src="/assets/img/2021/12/050-run-after.webp" alt="Exception handling using runAfter configuration" width="1473" loading="lazy" style="width: 1473px;"></p>
<p style="text-align: center;">Figure 11. Configuring <code>runAfter</code> settings for an action</p>
<h2>Terminating a Workflow</h2>
<p>The&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#terminate-action?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Terminate</a>&nbsp;action allows finishing the execution of a workflow instance based on business or non-functional requirements. This action allows terminating the workflow instance with different status including:&nbsp;"<code>Succeeded</code>",&nbsp;"<code>Failed</code>", or&nbsp;"<code>Cancelled</code>". When the workflow is terminated as&nbsp;"<code>Failed</code>", an error code and an error message can be added.&nbsp;The workflow termination status, error code, and error message are sent to Application Insights traces.</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/12/061-terminate.webp" alt="Using a Terminate action" width="965" loading="lazy" style="width: 965px;"></p>
<p style="text-align: center;">Figure 12. Using a Terminate action</p>
<h2>Application Insights Integration</h2>
<p>Logic Apps Standard provides integration with&nbsp;<a href="https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-monitor-with-application/ba-p/2003332" rel="noopener" target="_blank">Application Insights</a>. All inbuilt workflow tracing logs and metadata like workflow run identifier, custom tracking id, and tracked properties are ingested into Application Insights. All these logs can be queries using&nbsp;<a href="https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-query-overview?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Kusto query language (KQL)</a>. More information can be found in this&nbsp;<a href="https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-monitor-with-application/ba-p/2003332" rel="noopener" target="_blank">blog post</a>.&nbsp;</p>
<p>&nbsp;</p>
<p><img src="/assets/img/2021/12/071-app-insights.webp" alt="071-app-insights" width="822" loading="lazy" style="width: 822px;"></p>
<p style="text-align: center;">Figure 13. Enabling Application Insights integration</p>
<h2>Log Analytics Integration</h2>
<p>The Logic Apps consumption SKU provides direct integration with Log Analytics. As described in the previous section, Logic Apps Standard is integrated with Application Insights. However, Application Insights can be configured to be <a href="https://docs.microsoft.com/en-us/azure/azure-monitor/app/create-workspace-resource?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">workspace-based</a>. In other words, all logs and metrics of multiple Applications Insights instances can be ingested and aggregated into one Log Analytics workspace. This allows running queries across multiple Application Insights instances aggregates in the same workspace.</p>
<h2>Wrapping Up</h2>
<p>In this post, we’ve covered the different built-in observability features that we get in Logic Apps Standard. In the <a href="/monitoring-logic-apps-standard-with-app-insights-implementation" rel="noopener" target="_blank">next post</a> of the series, we will see how these features can be leveraged and implemented using a sample solution.</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>