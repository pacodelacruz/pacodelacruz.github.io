---
layout: post
title: Logic Apps Standard vs Durable Functions - How to Choose? (2021 Update)
date: 2021-11-09 10:00
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Logic Apps, Azure Functions, Architecture]
---

<p><img src="/assets/img/2021/11/how-to-choose.jpg" alt="how-to-choose" width="800" loading="lazy" style="width: 800px;"></p>
<h2 id="introduction">Introduction</h2>
<!--more-->
<p>Three years ago, after the release of Azure Durable Functions, I wrote a <a href="/articles/azure-durable-functions-vs-logic-apps">post</a> comparing the capabilities of Logic Apps and Durable Functions. Both service offerings provide orchestration capabilities on Azure. At that time, Azure Durable Functions had just been released as generally available. As you’d expect, many things have changed since then. Fast forward to 2021, <a href="https://techcommunity.microsoft.com/t5/azure-developer-community-blog/azure-logic-apps-announcement-ga-of-single-tenant-standard-sku/ba-p/2382460" rel="noopener" target="_blank">a new tier of Logic Apps has been released</a>, the Standard SKU.</p>
<p>During these three years, both platforms have evolved, and I keep hearing people wondering about how to choose between the two to build stateful workflows. Thus, I’ve decided to write an updated version of this comparison aiming at providing some decision guidelines for solution architects and developers.</p>
<p>In this post I’ll be comparing <a href="https://docs.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare#logic-app-standard-resource?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Logic Apps Standard</a> and <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-versions#new-features-in-2x?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Durable Functions 2.x</a>, and won’t be considering <a href="https://docs.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare#resource-types-and-environments?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Logic Apps consumption nor the Integration Service Environment</a> tiers. I’ll follow a similar structure to the one I used three years ago covering the sections below.</p>
<ul>
<li><a href="#development-experience-and-continuous-delivery">Development Experience and Continuous Delivery</a></li>
<li><a href="#orchestration-capabilities">Orchestration Capabilities</a></li>
<li><a href="#connectivity-capabilities">Connectivity Capabilities</a></li>
<li><a href="#state-management">State Management</a></li>
<li><a href="#hosting-and-execution-capabilities">Hosting and Execution Capabilities</a></li>
<li><a href="#management-and-monitoring">Management and Monitoring</a></li>
<li><a href="#pricing">Pricing</a></li>
</ul>
<h2 id="development-experience-and-continuous-delivery">Development Experience and Continuous Delivery</h2>
<p>Each platform has a very different development experience. For some developers, this might be one of the most important factors when deciding the platform to choose. However, I would argue that in most cases, we need to think beyond the development phase and consider how the solution will be managed, troubleshoot, and monitored once in production. In this section, I’ll compare different aspects of the development experience for each platform and will leave the operations side for a later section.</p>
<table style="border-collapse: collapse; border: 0px solid #757575;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px; border-top: none; border-left: none;" width="200">
<p>&nbsp;</p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Durable Functions</strong></p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Logic Apps</strong></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Approach</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Codeful (imperative code)</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Low-code (declarative via a visual designer)</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Supported languages</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>At the time of writing, <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#language-support&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Durable Functions supports</a>: C#, NodeJS, Python, F#, and PowerShell</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Workflows are implemented using a visual designer. Behind the visual representation of the workflow, there is a&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">JSON-based workflow definition language</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Local development and debugging</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Azure Functions can be developed and debugged offline with the local runtime and a storage emulator.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Logic Apps workflows can be <a href="https://docs.microsoft.com/en-us/azure/logic-apps/quickstart-create-logic-apps-visual-studio-code?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">developed and debugged locally using VS Code</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Continuous delivery</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Continuous delivery can be implemented <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-continuous-deployment?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">using Azure DevOps, GitHub,</a> or any other release management tool that supports Azure CLI or PowerShell.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Logic Apps can be deployed using <a href="https://techcommunity.microsoft.com/t5/integrations-on-azure/deploying-an-azure-logic-apps-standard-workflow-through-azure/ba-p/2533050" rel="noopener" target="_blank">Azure DevOps</a>, <a href="https://techcommunity.microsoft.com/t5/integrations-on-azure/deploying-a-logic-app-standard-resource-through-github-actions/ba-p/2518993" rel="noopener" target="_blank">GitHub</a>, or any other release management tool that supports Azure CLI or PowerShell. One of the key improvements introduced by Logic App Standard is the ability to parameterise environment variables <a href="https://docs.microsoft.com/en-us/azure/logic-apps/edit-app-settings-host-settings?tabs=azure-portal#manage-app-settings---localsettingsjson&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">via application settings</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="orchestration-capabilities">Orchestration Capabilities</h2>
<p>The orchestration capabilities of each platform are quite different. While the underlying implementation is abstracted from us in both platforms, it is relevant to understand how they work internally. How both engines work and how some workflow patterns are supported are described in the table below.</p>
<table style="border-collapse: collapse; border: 0px solid #757575;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px; border-top: none; border-left: none;" width="165">
<p>&nbsp;</p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="238">
<p><strong>Durable Functions</strong></p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="198">
<p><strong>Logic Apps</strong></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Orchestration trigger</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>A workflow or <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview#orchestrator-functions?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">orchestration function</a> instance can be instantiated by <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview#client-functions?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">client function</a>, which in turn can be triggered using any of the available <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp#supported-bindings&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">trigger bindings</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>A Logic App workflow can be initiated by the many different available connectors that support <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#triggers-overview?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">triggers</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Actions being orchestrated</strong>&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p><span>An </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview#orchestrator-functions?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>orchestration function</span></a><span> can call </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview#activity-functions?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>activity functions</span></a><span> and </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-sub-orchestrations?tabs=csharp&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>sub-orchestrations</span></a><span>. Activity functions can implement any of the supported </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp#supported-bindings&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>output bindings</span></a><span>. </span></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>Many different&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#actions-overview?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">workflow actions</a>&nbsp;can be orchestrated. Logic Apps workflows can invoke actions offered by the 450+ connectors or call <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-http-endpoint#create-nested-logic-apps?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">nested Logic App workflows</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Flow control</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>In Durable Functions, the workflow's flow is controlled using standard code constructs, such as conditions, switch-case statements, loops, try-catch blocks, etc.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>You can control the flow with&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-conditional-statement?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">conditional statements</a>,&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-switch-statement?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">switch statements<span style="color: #86bc25;">,</span></a>&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-loops?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">loops</a>,&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-run-steps-group-scopes?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">scopes</a><span>,</span>&nbsp;and controlling activity chaining using the&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling#catch-and-handle-failures-with-the-runafter-property?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">runAfter property</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Programmatic instance management</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>Durable Functions instances can be managed using the management API or the orchestration client as <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-instance-management?tabs=csharp#query-instances&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">described here</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>While there is a management API for instance management in Logic Apps consumption, at the time of writing, there is no API with similar functionality for Logic Apps Standard.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Concurrency control</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>Concurrency throttling&nbsp;<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-perf-and-scale#concurrency-throttles?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">is supported</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>Concurrency control can be configured at <a href="https://www.serverlessnotes.com/docs/logic-apps-improve-performance-using-parallelism" rel="noopener" target="_blank">workflow level</a> or <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#change-for-each-concurrency?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">loop level</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Error handling</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-error-handling?tabs=csharp&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Error handling</a> is implemented as part of the orchestrator function.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>Error handling is done using <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">retry policies and runAfter behaviour</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Chaining pattern</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>Functions can be&nbsp;<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#chaining&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">executed in a sequence</a>&nbsp;and outputs of one can be inputs of subsequent ones.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>Actions can easily be chained in a workflow. Additionally, the&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling#catch-and-handle-failures-with-the-runafter-property?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">runAfter property</a>&nbsp;allows executing actions based on the status of a previous action or scope.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Fan-out / fan-in pattern</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>Functions can be executed&nbsp;<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#fan-in-out&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">in parallel</a>&nbsp;and the workflow can continue when all or any of the branches finish.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>You can fan-out and fan-in actions in a workflow by implementing&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-branches?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">parallel branches</a>, or&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-loops?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">for each loops</a>&nbsp;running in parallel.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Async HTTP APIs pattern</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>Client applications or services can invoke Durable Functions orchestrations via HTTP APIs and get the&nbsp;<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#async-http&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">orchestration status</a> asynchronously. Additionally, you can&nbsp;<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-custom-orchestration-status?tabs=csharp&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">set a custom status value</a>&nbsp;that could be queried by external clients.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>The HTTP response action in Logic Apps supports asynchronous calls. You can configure it to return a HTTP 202 accepted status and a location header to retrieve the running state asynchronously.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Monitor pattern</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>The monitor pattern can be implemented as <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#monitoring&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">described here</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>The monitor pattern can be implemented using the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-loops#until-loop?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">until loop</a>. Additionally, all HTTP-based actions follow the standard <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-native-http#asynchronous-request-response-behavior?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">asynchronous operation pattern</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>Approval workflow (human interaction) pattern</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>The approval or human Interaction pattern can be implemented&nbsp;<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#human&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">as described here</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>Approval workflows can be implemented&nbsp;<a href="https://docs.microsoft.com/en-us/azure/logic-apps/tutorial-process-mailing-list-subscriptions-workflow?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">with the out-of-the-box connectors</a>&nbsp;or custom as&nbsp;<a href="/articles/correlation-identifier-pattern-on-logic-apps">described here</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="165">
<p><strong>A</strong><strong>ggregator pattern</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="238">
<p>The aggregator pattern can be implemented <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#aggregator&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">using durable entities</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="198">
<p>To implement the aggregator pattern on Logic Apps, an external data store is required.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="connectivity-capabilities">Connectivity Capabilities</h2>
<p>Logic Apps have been built with a key focus on integration. Thus, connectivity is one of its strengths. However, not all scenarios require connecting to many different systems and Durable Functions might have all that you need. The connectivity capabilities of each service offering are contrasted in the table below.</p>
<table style="border-collapse: collapse; border: 0px solid #757575;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px; border-top: none; border-left: none;" width="200">
<p>&nbsp;</p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Durable Functions</strong></p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Logic Apps</strong></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Connectors and bindings</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p><span>At the time of writing, Azure Functions provides </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp#supported-bindings&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>16 supported bindings</span></a><span>. Some of these bindings can trigger a function, while others can provide inputs or outputs. </span></p>
<p><span>Additionally, as Azure Functions can be triggered by Event Grid events, any&nbsp;</span><a href="https://docs.microsoft.com/en-us/azure/event-grid/overview#event-sources?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>Event Grid source</span></a><span>&nbsp;can potentially become a trigger for Azure Functions.</span></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p><span>At the time of writing, Logic Apps provide&nbsp;</span><a href="https://docs.microsoft.com/en-us/connectors/connector-reference/connector-reference-logicapps-connectors?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>more than 450 connectors</span></a><span>, and the list keeps growing. Among these, there are protocol connectors, Azure Services connectors, Microsoft SaaS connectors, and third-Party SaaS Connectors.</span></p>
<p>Some of these connectors can trigger Logic App workflows, while others support getting and pushing data.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Custom connectors</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>You can create&nbsp;<a href="https://microsoft.github.io/AzureTipsAndTricks/blog/tip247.html" rel="noopener" target="_blank">custom bindings</a>&nbsp;for Azure Functions.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Logic Apps allow you to build&nbsp;<a href="https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-built-in-connector/ba-p/1921272" rel="noopener" target="_blank">custom connectors</a> based on a connector extensibility model.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Network capabilities</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p><span>Azure Functions hosted on premium, dedicated plans, or an App Service Environment have </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-networking-options?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>multiple networking capabilities</span></a><span>, including inbound access restrictions, VNet integration, hybrid connections, and outbound restrictions. </span></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p><span>Given that Logic Apps Standard is hosted on the Azure Functions runtime, similar </span><a href="https://techcommunity.microsoft.com/t5/integrations-on-azure/logic-apps-anywhere-networking-possibilities-with-logic-app/ba-p/2105047" rel="noopener" target="_blank"><span>networking capabilities are available</span></a><span>. </span></p>
<p><span>It is worth noting that only the built-in connectors are executed within the network context. At the time of writing the built-in connectors include Azure blob, Azure Functions, DB2, Event Hubs, HTTP, IBM Host File, MQ, Service Bus, and SQL Server.</span></p>
<p><span>Additionally, Logic Apps offers the&nbsp;</span><a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>on-premises data gateway</span></a><span>, which, through an agent installed on a virtual machine in a private network, allows you to connect to a </span><a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-connection#supported-data-sources?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>list of supported protocols and applications</span></a><span>. </span></p>
</td>
</tr>
</tbody>
</table>
<h2 id="state-management">State Management</h2>
<p>There are some differences in how each platform manages its state. These are discussed in the table below.</p>
<table style="border-collapse: collapse; border: 0px solid #757575;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px; border-top: none; border-left: none;" width="200">
<p>&nbsp;</p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Durable Functions</strong></p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Logic Apps</strong></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Storage providers</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>At the time of writing, <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-storage-providers#azure-storage?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Azure storage</a> is the only supported storage provider for production workloads. However, <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-storage-providers#mssql?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>SQL Server</span></a><span> and </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-storage-providers#netherite?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>Event Hubs via Netherite</span></a><span> are available in preview. </span>&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p><a href="https://docs.microsoft.com/en-us/azure/azure-functions/storage-considerations#storage-account-requirements?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Azure storage</a> is supported as GA for storing the workflow state. Additionally, <a href="https://docs.microsoft.com/en-au/azure/logic-apps/set-up-sql-db-storage-single-tenant-standard-workflows?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">SQL Server</a> is available in preview.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Shared state across instances (data state)</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities?tabs=csharp&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Durable entities</a> can be leveraged to share states across instances.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Many polling triggers, e.g., file system, SQL, etc., have a state that is shared across instances. However, Logic Apps does not offer built-in support for shared custom states across instances. If that is a requirement, you would need to rely on external storage services.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="hosting-and-execution-capabilities">Hosting and Execution Capabilities</h2>
<p>Relevant hosting features and execution capabilities are compared in the table below.</p>
<table style="border-collapse: collapse; border: 0px solid #757575;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px; border-top: none; border-left: none;" width="200">
<p>&nbsp;</p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="196">
<p><strong>Durable Functions</strong></p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="204">
<p><strong>Logic Apps</strong></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Run duration limits</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="196">
<p>One instance can run without defined time limits unless there are retention limitations in the storage provider (see state management section).</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="204">
<p>In Logic Apps Standard, the maximum workflow run duration <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config?tabs=azure-portal#run-duration-and-retention-history-limits&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">can be configured in the host.json file</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Hosting</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="196">
<p>Azure Functions can run as managed PaaS on Azure, but also <a href="https://docs.microsoft.com/en-us/azure/app-service/overview-arc-integration?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">on-premises or third-party clouds using Azure Arc (currently in preview)</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="204">
<p>Logic Apps can run as managed PaaS on Azure, but also <a href="https://docs.microsoft.com/en-us/azure/app-service/overview-arc-integration?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">on-premises or third-party clouds using Azure Arc (currently in preview)</a></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p>&nbsp;<strong>Versioning</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="196">
<p><span>When deploying a new version with breaking changes different </span><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-versioning#mitigation-strategies?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank"><span>mitigations strategies</span></a><span> can be implemented.</span></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="204">
<p>When a new workflow version is deployed, in-flight workflows will continue working on the version they started with.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Throughput</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="196">
<p>Durable Functions <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-perf-and-scale#performance-targets?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">can be tuned</a> for higher throughput. For high throughput scenarios, <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-perf-and-scale#high-throughput-processing?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">the Netherite storage provider is recommended (currently in preview)</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="204">
<p>Logic Apps can be configured to have lower latency with <a href="https://docs.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare#stateful-and-stateless-workflows?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">stateless workflows</a>. However, expect Logic Apps to have considerably slower response times compared to Durable Functions.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="management-and-monitoring">Management and Monitoring</h2>
<p>Each platform offers different management and monitoring tools as described in the table below.</p>
<table style="border-collapse: collapse; border: 0px solid #757575;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px; border-top: none; border-left: none;" width="200">
<p>&nbsp;</p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Durable Functions</strong></p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Logic Apps Standard</strong></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Tracing and logging</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Durable Functions has <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-diagnostics?tabs=csharp#application-insights&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">native integration with Application Insights</a> that provides built-in tracing. Additionally, you can implement <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-diagnostics?tabs=csharp#app-logging&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">application-level custom tracing</a>. These traces can be queried, and charts can be rendered using KQL.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Logic Apps Standard offers native integration with <a href="https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-monitor-with-application/ba-p/1877849" rel="noopener" target="_blank">Application Insights</a>. These traces can be queried, and charts can be rendered using KQL.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Monitoring</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>A <a href="https://github.com/scale-tone/DurableFunctionsMonitor#durable-functions-monitor" rel="noopener" target="_blank">community-built monitor</a> is available to monitor Durable Functions.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Logic Apps allow you to monitor and view the run history of <a href="https://docs.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare#stateful-and-stateless-workflows?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">stateful workflows</a>.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Instance resubmission</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p><a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-instance-management?tabs=csharp#rewind-instances-preview&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">Failed instances can be rewound</a>, currently in preview support.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Logic Apps allows you to resubmit a failed message from the run history.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="pricing">Pricing</h2>
<p>Considering that Logic Apps Standard runs on top of the Azure Functions runtime, pricing is not very similar. However, Logic Apps Standard does not currently support pay-per-use. Pricing details of each platform are compared in the table below.</p>
<table style="border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: 1px solid #99acc2;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px; border-top: none; border-left: none;" width="200">
<p>&nbsp;</p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Durable Functions</strong></p>
</td>
<td style="border: 1px solid windowtext; text-align: center;" width="200">
<p><strong>Logic Apps Standard</strong></p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Pay-per-use</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Can run in a pay-per-use <a href="https://docs.microsoft.com/en-us/azure/azure-functions/consumption-plan?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">consumption-based plan</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Currently, there is no consumption plan for Logic App Standard.</p>
</td>
</tr>
<tr>
<td style="border: 1px solid windowtext; text-align: right;" width="200">
<p><strong>Instance-based plans</strong></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Durable Functions can run in a <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale#overview-of-plans?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">premium plan, app service plan, or an App Service Environment.</a></p>
</td>
<td style="border: solid windowtext 1.0pt; border-style: solid; border-width: 1px;" width="200">
<p>Logic Apps can run in <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-pricing#pricing-tiers-in-the-standard-model?WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">workflow standard plans and App Service Environments</a>.</p>
</td>
</tr>
</tbody>
</table>
<h2 id="wrapping-up">Wrapping-Up</h2>
<p>In this post, I’ve contrasted some features of both Durable Functions and Logic Apps Standard. Considering that now Logic Apps Standard runs on top of the Azure Functions runtime, both services are much more similar than they were before. However, depending on the requirements or even personal preferences, an architect or developer might prefer one or the other. Below, I try to provide some decision guidelines summarising the comparison presented above.</p>
<p>Logic Apps are usually better suited for the following scenarios:</p>
<ul>
<li>Enterprise application and B2B integration solutions that would benefit from the 450+ available connectors.</li>
<li>Rich built-in monitoring tools are required.</li>
<li>A low-code approach with a visual designer is preferred over code.</li>
<li>Asynchronous processes in which low response times is not a must.</li>
<li>It is acceptable to pay for a dedicated hosting plan.</li>
</ul>
<p>In contrast, Durable Functions are typically a better fit in the scenarios below.</p>
<ul>
<li>The <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp#supported-bindings&amp;WT.mc_id=AZ-MVP-5003116" rel="noopener" target="_blank">available bindings</a> suffice to meet the connectivity requirements.</li>
<li>The available <a href="https://github.com/scale-tone/DurableFunctionsMonitor" rel="noopener" target="_blank">community-built monitoring tool</a> meets the monitoring requirements.</li>
<li>A codeful approach is preferred or complex workflow constructs are required.</li>
<li>High throughput and low latency scenarios.</li>
<li>A consumption plan is preferred for cost optimisation.</li>
</ul>
<p>It is also worth noting that in some cases you might not need to choose one over another, but leverage both to build your solution, as both can complement each other.</p>
<p>I hope this post has provided you with useful information and that you are better prepared to choose the right platform to use in your next project.</p>
<p>Happy clouding! <span style="font-size: 11px; color: #666666;">&nbsp;</span></p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>