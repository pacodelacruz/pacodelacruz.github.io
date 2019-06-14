---
layout: post
title: Azure Durable Functions Pattern Approval Workflow with SendGrid
date: 2018-04-17 11:50
author: Paco de la Cruz
comments: true
categories: [Azure Functions, Development, Durable Functions, SendGrid, Uncategorized]
---
<h2>Introduction</h2>
<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview" target="_blank" rel="noopener noreferrer">Durable Functions</a> is a new (in preview at the time of writing) and very interesting extension of Azure Functions that allows you to build stateful and serverless code-based workflows. The Durable Functions extension abstracts all the state management, queueing, and checkpoint implementation commonly required for an orchestration engine. Thus, you just need to focus on your business logic without worrying much on the underlying complexities. Thanks to this extension, now you can:
<ol>
	<li>Implement long-running serverless code-based services beyond the current Azure Function limitation of 10 minutes (as long as you can break down your process into small nano-services which can be orchestrated);</li>
	<li>Chain Azure functions, i.e., call one function after the other and pass the output of the first one as an input to the next one (<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-1-function-chaining" target="_blank" rel="noopener noreferrer">Function chaining pattern</a>);</li>
	<li>Execute several functions asynchronously and then continue the workflow when any or all of the asynchronous tasks are completed (<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-2-fan-outfan-in" target="_blank" rel="noopener noreferrer">Fan-out and Fan-in pattern</a>);</li>
	<li>Get the status of a long-running workflow from external clients (<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-3-async-http-apis" target="_blank" rel="noopener noreferrer">Async HTTP APIs Pattern</a>);</li>
	<li>Implement the correlation identifier pattern to enable human interaction processes, such as an approval workflow (<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-5-human-interaction" target="_blank" rel="noopener noreferrer">Human Interaction Pattern</a>) and;</li>
	<li>Implement a flexible recurring process with lifetime management (<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-4-monitoring" target="_blank" rel="noopener noreferrer">Monitoring Pattern</a>).</li>
</ol>
It’s worth noting that Azure Durable Functions is not the only way to implement stateful workflows in a serverless manner on Azure. <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview" target="_blank" rel="noopener noreferrer">Azure Logic Apps</a> is another awesome platform, <a href="https://pacodelacruzag.wordpress.com/2018/02/02/microsoft-azure-ipaas-2/" target="_blank" rel="noopener noreferrer">core component of the Microsoft Azure iPaaS</a>, that allows you to build serverless and stateful workflows using a designer. In a previous post, <a href="https://pacodelacruzag.wordpress.com/2017/07/17/correlation-identifier-pattern-on-logic-apps/" target="_blank" rel="noopener noreferrer">I showed how to implement the approval workflow pattern on Logic Apps via SMS messages leveraging Twilio</a>.

In this post, I will show how to implement the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-5-human-interaction" target="_blank" rel="noopener noreferrer">Human Interaction Pattern</a> on Azure Durable Functions with SendGrid. You will see on the way that this implementation requires other Durable Functions patterns, such as, function chaining, fan-out and fan-in, and optionally the Async HTTP API Pattern.
<h2>Scenario</h2>
To illustrate this pattern on Durable Functions, I will be using a fictitious cat model agency called Furry Models Australia. Furry Models is running a campaign to attract the most glamorous, attractive, and captivating cats in Australia. They will be receiving photos of all aspiring cats and they need a streamlined approval process to accept or reject those applications. Furry Models want to implement this in an agile manner with a short time-to-market and with a very cost-effective solution. They know that serverless is the way to go!

<img class=" size-full wp-image-1161 aligncenter" src="/assets/img/2018/06/11-join-us1.png" alt="11 Join Us" width="570" height="598" />
<h2>Pre-requisites</h2>
To build this solution, we will need:
<ul>
	<li>A <a href="https://sendgrid.com/" target="_blank" rel="noopener noreferrer">SendGrid</a> account. Given that Azure Functions provides an output binding for SendGrid to send emails, we will be relying on this service. In case you want to implement this solution, you would need a SendGrid account. Once you sign up, you need to get your <a href="https://sendgrid.com/docs/Classroom/Send/How_Emails_Are_Sent/api_keys.html" target="_blank" rel="noopener noreferrer">API Key</a>, which is required for the Azure binding. You can get more information about the SendGrid binding for Azure Functions and how to use it <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-sendgrid" target="_blank" rel="noopener noreferrer">here</a>.</li>
	<li>An Azure Storage Account: The solution requires a Storage Account with 3 blob containers: <em>requests</em>, <em>approved</em>, and <em>rejected</em>. The <em>requests</em> container should have <strong>public access level</strong> so blobs can be viewed without a SAS token. For your own solution, you might want to make this more secure.</li>
</ul>
<h2>Solution Overview</h2>
The picture below shows an overview of the approval workflow solution I've build based on Durable Functions.

Pictures of the aspiring cats are to be dropped in an Azure storage blob container called <em>requests</em>. At the end of the approval workflow, pictures should be moved to the <em>approved</em> or <em>rejected</em> blob containers accordingly.

<img class=" size-full wp-image-1164 aligncenter" src="/assets/img/2018/06/20-solution-overview.png" alt="20 Solution Overview" width="784" height="727" />

The steps of the process are described as follows:
<ol>
	<li>The process is being triggered by an Azure Function with the <em>BlobTrigger</em> input binding monitoring the <em>requests</em> blob container. This function also implements the <em>DurableOrchestrationClient</em> attribute to instantiate a Durable Function orchestration</li>
	<li>The <em>DurableOrchestrationClient</em> starts a new instance of the orchestration.</li>
	<li>Then, the Durable Function orchestration calls another function with the <em>ActivityTrigger</em> input binding, which is in charge of sending the approval request email using the <em>SendGrid</em> output binding.</li>
	<li>SendGrid sends the approval request email to the (cat) user.</li>
	<li>Then, in the orchestration, a timer is created so that the approval workflow does not run forever, and in case no approval is received before the timer finishes the request is rejected.</li>
	<li>The (cat) user receives the email, and decides whether the aspiring cat deserves to join Furry Models or not, by clicking the <em>Approve</em> or <em>Reject</em> button. Each button has a link to an <em>HttpTrigger</em> Azure Function which expects the selection and the orchestration instanceId as query params</li>
	<li>The <em>HttpTrigger</em> function receives the selection and the orchestration instanceId. The function checks the status of the orchestration instance, if it’s not running, it returns an error message to the user. If it’s running, it raises an event to the corresponding orchestration instance.</li>
	<li>The corresponding orchestration instance receives the external event.</li>
	<li>The workflow continues when the external event is received or when the timer finishes; whatever happens first. If the timer finishes before a selection is received, the application is automatically rejected.</li>
	<li>The orchestration calls another <em>ActivityTrigger</em> function to move the blob to the corresponding container (<em>approved</em> or <em>rejected)</em>.</li>
	<li>The orchestration finishes.</li>
</ol>
A sample of the email implemented is shown below.

<img class="alignnone size-full wp-image-1166" src="/assets/img/2018/06/22b-sample-email.png" alt="22b Sample Email" width="600" />
<h2>The Solution</h2>
The implemented solution code can be found in this <a href="https://github.com/pacodelacruz/durablefunctions" target="_blank" rel="noopener noreferrer">GitHub repo</a>. I’ve used the Azure Functions Runtime v2. I will highlight some relevant bits of the code below, and I hope that the code is self-explanatory 😉:
<h3>TriggerApprovalByBlob.cs</h3>
This <em>BlobTrigger </em>function is triggered when a blob is created in a blob container and starts the Durable Function ochestration (Step 1 above)

<p/>
<script src="https://gist.github.com/pacodelacruz/978802dbfabcf7bf4e81bfc4b27977f3.js"></script>
<p/>

<h3>OrchestrateRequestApproval.cs</h3>
This is the Durable Function orchestration which handles the workflow and is started by the step 2 above.

<p/>
<script src="https://gist.github.com/pacodelacruz/56867a4e242b7d592229c39bbef2f61b.js"></script>
<p/>

<h3>SendApprovalRequestViaEmail.cs</h3>
<em>ActivityTrigger</em> function which sends the approval request via email with the <em>SendGrid</em> output binding (Step 3 above).

<p/>
<script src="https://gist.github.com/pacodelacruz/b558f66df8c389ae948c9135fbf747fd.js"></script>
<p/>

<h3>ProcessHttpGetApprovals.cs</h3>
<em>HttpTrigger</em> function that handles the Http Get request initiated by the user selection (click) on the email (Step 7 above).

<p/>
<script src="https://gist.github.com/pacodelacruz/0082c3a0eacfa9c7ff071d9dac558d28.js"></script>
<p/>

<h3>MoveBlob.cs</h3>
<em>ActivityTrigger</em> function that moves the blob to the corresponding container (Step 10 above).

<p/>
<script src="https://gist.github.com/pacodelacruz/f95e57d1f78e6e42898297cb9adcd5bd.js"></script>
<p/>

<h3>local.settings.json</h3>
These are the settings which configure the behaviour of the solution, including the storage account connection strings, the SendGrid API key, templates for the email, among others. You would need to implement these as app settings when deploying to Azure

<p/>
<script src="https://gist.github.com/pacodelacruz/bf6fa8d6a0609b7e45978cba9fd02851.js"></script>
<p/>

<h2>Wrapping up</h2>
In this post, I’ve shown how to implement an Approval Workflow (Human Interaction pattern) on Azure Durable Functions with SendGrid. Whether you wanted to learn more about Durable Functions, to implement a serverless approval workflow or you run a cat model agency, I hope you have found it useful :) Please feel free to ask any questions or add your comments below.

Happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on <a href="https://twitter.com/pacodelacruz">@pacodelacruz</a></span></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
<p style="margin:0;font-family:Calibri;font-size:11pt;"></p>
