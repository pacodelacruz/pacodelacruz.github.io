---
layout: post
title: Azure Durable Functions vs Logic Apps - How to choose?
date: 2018-05-10 12:09
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Architecture, Azure Functions, Azure iPaaS, Development, Durable Functions, iPaaS, Logic Apps, Microsoft iPaaS]
---
<h2><img class=" size-full wp-image-1176 aligncenter" src="/assets/img/2018/06/01-feature.png" alt="01 Feature" width="594" height="255" /></h2>
<h2>Introduction</h2>
Azure currently has two service offerings of serverless compute: <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview" target="_blank" rel="noopener noreferrer">Azure Logic Apps</a> and <a href="https://docs.microsoft.com/en-us/azure/azure-functions/" target="_blank" rel="noopener noreferrer">Azure Functions</a>. Until recently, <a href="https://stackoverflow.com/questions/36375220/azure-functions-vs-logic-apps" target="_blank" rel="noopener noreferrer">one could argue</a> that Azure Functions were code triggered by events while Logic Apps were event-triggered workflows. However, that changed after the release of <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview" target="_blank" rel="noopener noreferrer">Azure Durable Functions</a> which <a href="https://azure.microsoft.com/en-au/blog/break-through-the-serverless-barriers-with-azure-functions/?utm_content=71110581&amp;utm_medium=social&amp;utm_source=twitter" target="_blank" rel="noopener noreferrer">have reached General Availability</a> very recently. Durable Functions is an extension of Azure Functions that allows you to build stateful and serverless code-based workflows. With <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview" target="_blank" rel="noopener noreferrer">Azure Logic Apps</a> you can create stateful and serverless workflows through a visual designer.

If you are architecting a solution that requires serverless and stateful workflows on Azure, you might be wondering how to choose between Azure Durable Functions and Logic Apps. This post aims to shed some light to select the platform that better suits your needs.

<span style="color:#01a1dd;font-family:Questrial, sans-serif;font-size:26px;">Development</span>

For some people the development experience might be a very key factor when deciding a platform over the other. The development experience of both platforms is quite different as described below:
<table style="border-collapse:collapse;border-color:#757575;" border="0"><colgroup> <col style="width:200px;" /> <col style="width:416px;" /> <col style="width:416px;" /></colgroup>
<tbody>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:solid .5pt;border-right:solid .5pt;"></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Durable Functions</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Logic Apps</strong></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Paradigm</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Imperative code</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Declarative code</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Languages</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">At the time of writing <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#language-support" target="_blank" rel="noopener noreferrer">only C# is officially supported</a>. However, you can <a href="https://mikhail.io/2018/02/azure-durable-functions-in-fsharp/" target="_blank" rel="noopener noreferrer">make them work with F#</a> and JavaScript support is currently in preview.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Workflows are implemented using a visual designer on the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/quickstart-create-first-logic-app-workflow" target="_blank" rel="noopener noreferrer">Azure Portal</a> or <a href="https://docs.microsoft.com/en-us/azure/logic-apps/quickstart-create-logic-apps-with-visual-studio" target="_blank" rel="noopener noreferrer">Visual Studio</a>. Behind the visual representation of the workflow, there is the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language" target="_blank" rel="noopener noreferrer">JSON-based Workflow Definition Language</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Offline Development</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Can be developed offline with the local runtime and Storage emulator.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">You need to be online with access to the Azure to be able to develop your workflows.</td>
</tr>
</tbody>
</table>
Durable Functions allow you to use imperative code you might already be familiar with, but you still need to understand the constraints of this extension. Logic Apps might require you to learn to use a new development environment, but which is relatively straight forward and quite handy for scenarios where less coding is the preference.
<h2>Connectivity</h2>
<a href="/microsoft-azure-ipaas-2/" target="_blank" rel="noopener noreferrer">Logic Apps is an integration platform</a>, thus, it truly offers better connectivity than Azure Durable Functions. Some details to consider are described in the table as follows.
<table style="border-collapse:collapse;border-color:#757575;" border="0">
<tbody>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;"></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Durable Functions</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Logic Apps</strong></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Connectors or Bindings</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">The list of supported bindings is <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings#supported-bindings" target="_blank" rel="noopener noreferrer">here</a>. Some of these bindings support triggering a function, or are inputs or outputs. The list of bindings is growing, especially for the Functions runtime version 2.

Additionally, as Azure Functions can be triggered by Event Grid events, any <a href="https://docs.microsoft.com/en-us/azure/event-grid/overview#event-sources" target="_blank" rel="noopener noreferrer">Event Grid Publishers</a> can potentially become a trigger of Azure Functions.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Logic Apps provide <a href="/microsoft-azure-ipaas-2/" target="_blank" rel="noopener noreferrer">more than 200 connectors</a>, and the list just keeps growing. Among these, there are protocol connectors, Azure Services connectors, Microsoft SaaS connectors, and third-Party SaaS Connectors.

Some of these connectors can trigger Logic App workflows, while others support getting and pushing data as part of the workflow.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Custom Connectors</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">You can create <a href="https://github.com/Azure/azure-webjobs-sdk/wiki/Creating-custom-input-and-output-bindings" target="_blank" rel="noopener noreferrer">custom input and output bindings</a> for Azure Functions.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Logic Apps allow you to build <a href="https://docs.microsoft.com/en-us/azure/logic-apps/custom-connector-overview" target="_blank" rel="noopener noreferrer">custom connectors</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Hybrid Connectivity</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">Azure Functions hosted on a App Service Plan (not consumption plan) support <a href="https://docs.microsoft.com/en-us/azure/app-service/app-service-hybrid-connections" target="_blank" rel="noopener noreferrer">Hybrid Connections</a>. Hybrid connections allows to have a TCP tunnel to access on-premises systems and services securely.

Additionally, Azure Functions deployed on an App Service Plan can be <a href="https://docs.microsoft.com/en-us/azure/app-service/web-sites-integrate-with-vnet" target="_blank" rel="noopener noreferrer">integrated to a VNET</a> or deployed on a dedicated <a href="https://docs.microsoft.com/en-us/azure/app-service/environment/intro" target="_blank" rel="noopener noreferrer">App Service Environment</a> to access resources and services on-premises.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Logic Apps offers the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install" target="_blank" rel="noopener noreferrer">On-Premises Data Gateway</a>, which, through an agent installed on-premises, allows you to connect to a list of supported protocols and applications.

It’s worth mentioning that the <a href="https://trello.com/b/9GhzIReR/azure-logic-apps-product-roadmap" target="_blank" rel="noopener noreferrer">Product Team is currently working on Isolated Logic Apps</a>, which will **<strong><u>in the future</u>**</strong> be deployed on your own VNET, thus will have access to resources on-premises, which will unlock many scenarios.</td>
</tr>
</tbody>
</table>
&nbsp;
<h2>Workflow</h2>
Both workflow engines are quite different. Even though the underlying implementation is abstracted for us, it’s important to know how they work internally when architecting enterprise-grade solutions. How both engines work and how some workflow patterns are supported is described below.
<table style="border-collapse:collapse;border-color:#757575;" border="0">
<tbody>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;"></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Durable Functions</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Logic Apps</strong></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Trigger</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">A workflow instance can be instantiated by any Azure Function implementing the <em><a href="https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html" target="_blank" rel="noopener noreferrer">DurableOrchestrationClient</a></em>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Can be initiated by the many different available <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#triggers-overview" target="_blank" rel="noopener noreferrer">triggers</a> offered by the connectors.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Actions being orchestrated</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Can orchestrate Activity Functions (with the <em>ActivityTrigger</em> attribute). However, those Activity Functions could call other services, using any of the supported bindings.

Additionally, orchestrations can <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-sub-orchestrations">call sub-orchestrations</a>.

At the time of writing, an orchestration function can only call activity functions that are defined in the same Function App. This could potentially hinder reusability of services.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Many different <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#actions-overview" target="_blank" rel="noopener noreferrer">workflow actions</a> can be orchestrated. Logic Apps workflows can be calling actions of the more than 200 connectors, workflow steps, other Azure Functions, other Logic Apps, etc.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Flow Control</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">The workflow's flow is controlled using the standard code constructs. E.g. conditions, switch case statements, loops, try-catch blocks, etc.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">You can control the flow with <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-conditional-statement" target="_blank" rel="noopener noreferrer">conditional statements</a>, <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-switch-statement" target="_blank" rel="noopener noreferrer">switch statements,</a> <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-loops" target="_blank" rel="noopener noreferrer">loops</a>, <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-run-steps-group-scopes" target="_blank" rel="noopener noreferrer">scopes</a> and controlling the activity chaining with the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling#catch-and-handle-failures-with-the-runafter-property" target="_blank" rel="noopener noreferrer">runAfter property</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Chaining Pattern</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Functions can be <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-1-function-chaining" target="_blank" rel="noopener noreferrer">executed in a sequence</a> and outputs of one can be inputs of subsequent ones.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Actions can easily be chained in a workflow. Additionally the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling#catch-and-handle-failures-with-the-runafter-property" target="_blank" rel="noopener noreferrer">runAfter property</a> allows to execute actions based on the status of a previous action or scope.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Fan-Out / Fan-In Pattern</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Functions can be executed <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-2-fan-outfan-in" target="_blank" rel="noopener noreferrer">in parallel</a> and the workflow can continue when all or any of the branches finish.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">You can fan-out and fan-in actions in a workflow by simply implementing <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-branches" target="_blank" rel="noopener noreferrer">parallel branches</a>, or <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-control-flow-loops" target="_blank" rel="noopener noreferrer">ForEach loops</a> running in parallel.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Async HTTP APIs and Get Status Pattern</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Client applications or services can invoke Durable Functions orchestration via HTTP APIs asynchronously and later get the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-2-fan-outfan-in" target="_blank" rel="noopener noreferrer">orchestration status to learn when the operation completes</a>. Additionally, you can <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-custom-orchestration-status" target="_blank" rel="noopener noreferrer">set a custom status value</a> that could be query by external clients.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Client applications or services could call Logic Apps Management API to <a href="https://docs.microsoft.com/en-us/rest/api/logic/workflowruns/get" target="_blank" rel="noopener noreferrer">get the instance run status</a>. However, either the client has to have access to this API or you would need to implement a wrapper of this.

Custom Status value is not currently supported out-of-the-box. If required, you would need to persist it in a separate store and expose it with a custom API.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Approval Workflow (Human Interaction) Pattern</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">The Human Interaction (Approval Workflow) Pattern can be implemented <a href="/azure-durable-functions-approval-workflow-with-sendgrid/" target="_blank" rel="noopener noreferrer">as described here</a>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Approval Workflows can be implemented <a href="https://docs.microsoft.com/en-us/azure/logic-apps/tutorial-process-mailing-list-subscriptions-workflow" target="_blank" rel="noopener noreferrer">with the out-of-the box connectors</a> or custom as <a href="/correlation-identifier-pattern-on-logic-apps/" target="_blank" rel="noopener noreferrer">described here</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Correlation Pattern</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">The Correlation Pattern can be implemented not only when there is human interaction, but for broader scenarios in the same way as described above.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">The Correlation Pattern can easily be implemented using the <a href="/correlation-identifier-pattern-on-logic-apps/" target="_blank" rel="noopener noreferrer">webhook action</a> or with <a href="/logic-apps-correlation-and-message-dependency-management-on-logic-apps-with-service-bus" target="_blank" rel="noopener noreferrer">Service Bus sessions</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Programmatic instance management</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Client applications or services can monitor and terminate instances of Durable Functions orchestrations via the API.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Client applications or services could call Logic Apps Management API to <a href="https://docs.microsoft.com/en-us/rest/api/logic/workflowruns/get" target="_blank" rel="noopener noreferrer">monitor</a> and terminate instances of Logic App Workflows. However, either the client has to have access to this API or you would need to implement a wrapper.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Shared State across instances</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">Durable Functions support what they call “<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-eternal-orchestrations" target="_blank" rel="noopener noreferrer">eternal orchestrations</a>” which is a way to implement flexible loops with a state across loops without the need to store the complete iteration run history. However, this implementation has some important limitations, and the product team suggests to use only it for <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-4-monitoring" target="_blank" rel="noopener noreferrer">monitoring scenarios</a> that require flexible recurrence and lifetime management and when the lost of messages is acceptable.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Logic Apps does not support eternal orchestrations. However, different strategies can be used to implement endless loops with a state across instances. E.g. making use of a <a href="https://blogs.msdn.microsoft.com/david_burgs_blog/2018/02/27/reset-trigger-state-for-logic-apps/" target="_blank" rel="noopener noreferrer">trigger state</a> or storing the state in an external store to pass it from one instance to the next one in a singleton workflow.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Concurrency Control</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Concurrency throttling <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-perf-and-scale#concurrency-throttles" target="_blank" rel="noopener noreferrer">is supported</a>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Concurrency control can be configured at workflow level or loop level.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Lifespan</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">One instance can run without defined time limits.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">One instance of Logic Apps can run for up to <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config" target="_blank" rel="noopener noreferrer">90 days</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Error Handling</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">Implemented with the constructs of the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-error-handling" target="_blank" rel="noopener noreferrer">language used in the orchestration</a>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;"><a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling" target="_blank" rel="noopener noreferrer">Retry policies and catch strategies</a> can be implemented.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Orchestration Engine</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Orchestration functions and activity functions may be running on different VMs. However, Durable Functions ensures <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-checkpointing-and-replay" target="_blank" rel="noopener noreferrer">reliable execution of orchestrations</a>. To support this, check-pointing is implemented at each await statement. Additionally, the orchestration replays every time after resuming from an await call until it reaches the last activity check-pointed to rebuild the in-memory state of the instance. For high throughput scenarios, you could enable <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-perf-and-scale#orchestrator-function-replay" target="_blank" rel="noopener noreferrer">extended sessions</a>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">In Logic Apps the runtime engine breaks down the different tasks based on the workflow definition. These tasks are distributed among different workers. The engine makes sure that each task is executed at least once, and that tasks are not executed until their dependencies have finished with the expected status.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Some additional constraints and considerations</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">The orchestration function has to be implemented with <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-checkpointing-and-replay#orchestrator-code-constraints">some constraints</a> in mind, such as, the code must be deterministic and non-blocking, async calls can only be done using the <em>DurableOrchestrationContext</em> and infinite loops must be avoided.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">To control the workflow execution flow sometimes we need advanced constructs and operations, that can be complex to implement in Logic Apps.

The Worfklow definition language offers some <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language#functions">functions</a> that we can leverage, but sometimes, we need to make use of Azure Functions to perform advanced operations required as part of the workflow.

Additionally, you need to consider some <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config#limits">limits of Logic Apps</a>.</td>
</tr>
</tbody>
</table>
<h2></h2>
<h2><span style="background-color:transparent;">Deployment</span></h2>
<span style="background-color:transparent;">The deployment of these two platforms also has its differences, as detailed below. </span>
<table style="border-collapse:collapse;border-color:#757575;" border="0">
<tbody>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;"></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Durable Functions</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Logic Apps</strong></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;"><strong>CI/CD</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">Durable Functions builds and deployments can be automated using <a href="https://blogs.msdn.microsoft.com/visualstudioalmrangers/2017/10/04/azure-function-ci-cd-devops-pipeline/" target="_blank" rel="noopener noreferrer">VSTS build and release pipelines</a>. Additionally, other build and release management tools can be used.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Logic Apps are deployed using ARM Templates as <a href="/preparing-azure-logic-apps-for-cicd/" target="_blank" rel="noopener noreferrer">described here</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;"> <strong>Versioning</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">Versioning strategy is very important in Durable Functions. If you introduce breaking changes in a new version of your workflow, in-flight instances will break and fail.

You can find <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-versioning" target="_blank" rel="noopener noreferrer">more information and mitigation strategies here</a>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Logic Apps keep version history of all workflows saved or deployed. Running instances will continue running based on the active version when they started.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;"><strong>Runtime</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">Azure Functions can not only run on Azure, but be <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-runtime-overview" target="_blank" rel="noopener noreferrer">deployed on-premises</a>, on <a href="https://azure.microsoft.com/en-us/blog/general-availability-of-app-service-and-functions-on-azure-stack/" target="_blank" rel="noopener noreferrer">Azure Stack</a>, and can run on containers as well.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Logic Apps can only run on Azure.</td>
</tr>
</tbody>
</table>
<span style="background-color:transparent;"> </span>
<h2>Management and Monitoring</h2>
<span style="background-color:transparent;">How you manage and monitor each your solutions on platform is quite different. Some of the features are described in the table as follows.</span>
<table style="border-collapse:collapse;border-color:#757575;" border="0">
<tbody>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;"></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Durable Functions</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Logic Apps</strong></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;"><strong>Tracing and Logging</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">The orchestration activity is tracked by default in<span style="background-color:transparent;"> </span><a style="background-color:transparent;" href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-diagnostics#application-insights" target="_blank" rel="noopener noreferrer">Application Insights</a><span style="background-color:transparent;">. Furthermore, you can implement</span><span style="background-color:transparent;"> </span><a style="background-color:transparent;" href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-diagnostics#logging" target="_blank" rel="noopener noreferrer">logging to App Insights</a><span style="background-color:transparent;">.  </span></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">The <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-diagnosing-failures#review-run-history" target="_blank" rel="noopener noreferrer">run history and trigger history</a> are logged by default. Additionally, you can enable <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps#turn-on-diagnostics-logging-for-your-logic-app" target="_blank" rel="noopener noreferrer">diagnostic logging</a> to send additional details to Log Analytics. You can also make use of <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms" target="_blank" rel="noopener noreferrer">trackedProperties</a> to enrich your logging.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;"><strong>Monitoring</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">To monitor workflow instances, you need to use Application Insights Query Language to <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-diagnostics#application-insights" target="_blank" rel="noopener noreferrer">build your custom queries and dashboards</a>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">The <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms" target="_blank" rel="noopener noreferrer">Logic Apps blade</a> and <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-monitor-your-logic-apps-oms#view-your-logic-app-runs-in-your-log-analytics-workspace" target="_blank" rel="noopener noreferrer">Log Analytics workspace</a> solution for Logic Apps provide very rich and friendly visual tools for monitoring.

Furthermore, you can <a href="/business-activity-monitoring-on-azure-logic-apps/" target="_blank" rel="noopener noreferrer">build your own monitoring dashboards</a> and <a href="/publishing-custom-queries-of-logic-apps-execution-logs/" target="_blank" rel="noopener noreferrer">queries</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;"><strong>Resubmitting</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;">There is no out-of-the-box functionality to resubmit failed messages.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Failed instances can easily be resubmitted from the Logic Apps blades or the Log Analytics workspace.</td>
</tr>
</tbody>
</table>
<span style="background-color:transparent;color:#01a1dd;font-family:Questrial, sans-serif;font-size:26px;">Pricing</span>

Another important consideration when choosing the right platform is pricing. Even though both options offer a serverless option where you only pay for what you use, there are some differences to consider as described below.
<table style="border-collapse:collapse;border-color:#757575;" border="0">
<tbody>
<tr>
<td style="padding-left:14px;padding-right:14px;border-top:none;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;"></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Durable Functions</strong></td>
<td style="padding-left:14px;padding-right:14px;border-top:.5pt solid;border-left:none;border-bottom:.5pt solid;border-right:.5pt solid;text-align:center;"><strong>Logic Apps</strong></td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"> <strong>Serverless</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">In the consumption plan, you pay per-second of resource consumption and the number of executions. More details <a href="https://azure.microsoft.com/en-gb/pricing/details/functions/" target="_blank" rel="noopener noreferrer">described here</a>.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;"><span style="background-color:transparent;">For workflows you pay per-action and trigger (skipped, failed or succeeded). There is also a marginal cost for storage.</span>

In case you need B2B integration, XML Schemas and Maps or Liquid Templates, you would need to pay for an Integration Account.

More details <a href="https://azure.microsoft.com/en-us/pricing/details/logic-apps/" target="_blank" rel="noopener noreferrer">here</a>.</td>
</tr>
<tr>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;text-align:right;"><strong>Instance Based</strong></td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">Durable Functions can also be deployed on App Service Plans or App Service Environments where you pay per instance.</td>
<td style="padding-left:14px;padding-right:14px;border:.5pt solid;vertical-align:top;">At the moment there is no option to run Logic Apps on your dedicated instances. However, this <a href="https://trello.com/b/9GhzIReR/azure-logic-apps-product-roadmap" target="_blank" rel="noopener noreferrer">will change in the future</a>.</td>
</tr>
</tbody>
</table>
<span style="color:#01a1dd;font-family:Questrial, sans-serif;font-size:26px;background-color:transparent;">Wrapping-Up</span>

This post contrasts in detail the capabilities and features of both serverless workflow platforms available on Azure. The platform better suited really depends on the functional and non-functional requirements and also on your preferences. As a wrap-up, we could say that:

Logic Apps are better suited when
<ul>
	<li>Building integration solutions and leveraging the very extensive list of connectors would reduce the time-to-market and ease connectivity,</li>
	<li>Visual tools to manage and troubleshoot workflows are required,</li>
	<li>It’s ok to run only on Azure, and</li>
	<li>A visual designer and less coding are preferred.</li>
</ul>
And Durable Functions are a better fit if
<ul>
	<li>The list of available bindings is sufficient to meet the requirements,</li>
	<li>The logging and troubleshooting capabilities are sufficient, and you can build your custom monitoring tools,</li>
	<li>You require them to run not only on Azure, but on Azure Stack or Containers, and</li>
	<li>You prefer to have all the power and flexibility of a robust programming language.</li>
</ul>
It’s also worth mentioning that in most cases, the operation costs of Logic Apps tend be higher than those of Durable Functions, but that would depend case by case. And for enterprise-grade solutions, you should not decide on a platform based on price only, but you have to consider all the requirements and the value provided by the platform.

Having said all this, you can always mix and match Logic Apps and Azure Functions in the same solution so you can get the best of both worlds. <span style="background-color:transparent;">Hopefully this post has given you enough information to better choose the platform for your next cloud solution.</span>

You can also watch a summary video of this comparison here: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/__CDv956fgE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering Blog</span></a><br/>
<em>Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a></em></p>
