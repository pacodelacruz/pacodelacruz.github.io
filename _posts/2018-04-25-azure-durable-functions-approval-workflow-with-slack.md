---
layout: post
title: Azure Durable Functions Pattern - Approval Workflow with Slack
date: 2018-04-25 12:00
author: Paco de la Cruz
comments: true
categories: [Azure Functions, Development, Durable Functions, Slack]
---
<h2><img class=" size-full wp-image-1169 aligncenter" src="/assets/img/2018/06/00-feature.png" alt="00 Feature" width="532" height="385" /></h2>
<h2>Introduction</h2>
Recently, I published a post about implementing an <a href="https://pacodelacruzag.wordpress.com/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid/" target="_blank" rel="noopener noreferrer">Approval Workflow on Azure Durable Functions with SendGrid</a>. In essence, this post is not very different to that one. However, I wanted to demonstrate the same pattern on Azure Durable Functions, but now using <a href="https://slack.com/" target="_blank" rel="noopener noreferrer">Slack</a> as a means of approval. My aim is to show how easy it is to implement this pattern by using a Restful API instead of an Azure Functions binding. What you see here could easily be implemented with your own custom APIs as well :).
<h2>Scenario</h2>
In my <a href="https://pacodelacruzag.wordpress.com/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid/" target="_blank" rel="noopener noreferrer">previous post</a>, I show how Furry Models Australia streamlined an approval process for aspiring cats to join the exclusive model agency by implementing a serverless solution on Azure Durable Functions and SendGrid. Now, after a great success, they’ve launched a new campaign targeting rabbits. However, for this campaign they need some customisation. The (rabbit) managers of this campaign have started to collaborate internally with Slack instead of email. Their aim is to significantly improve their current approval process based on phone and pigeon post by having an automated serverless workflow which leverages Slack as their internal messaging platform.

<img class=" size-full wp-image-1170 aligncenter" src="/assets/img/2018/06/11-sorry.png" alt="11 Sorry" width="704" height="632" />
<h2>Pre-requisites</h2>
To build this solution, we need:
<ul>
	<li>Slack
<ul>
	<li><strong>Workspace</strong>: In case you don’t have one, you would need to <a href="https://get.slack.help/hc/en-us/articles/206845317-Create-a-Slack-workspace" target="_blank" rel="noopener noreferrer">create a workspace on Slack</a>, and you will need <a href="https://get.slack.help/hc/en-us/articles/222386767-Manage-apps-for-your-workspace" target="_blank" rel="noopener noreferrer">permissions to manage apps</a> in the workspace.</li>
	<li><strong>Channel</strong>: On that workspace, you need to <a href="https://get.slack.help/hc/en-us/articles/201402297-Create-a-channel" target="_blank" rel="noopener noreferrer">create a channel</a> where all approval requests will be sent to.</li>
	<li><strong>App</strong>: Once you have admin privileges on your Slack workspace, you should create a <a href="https://api.slack.com/slack-apps" target="_blank" rel="noopener noreferrer">Slack App</a>.</li>
	<li><strong>Incoming Webhook</strong>: On your Slack app, you would need to activate <a href="https://api.slack.com/incoming-webhooks" target="_blank" rel="noopener noreferrer">incoming webhooks</a> and then activate a new webhook. The incoming webhook will post messages to the channel you have just created. For that, you must authorise the app to post messages to the channel. Once you have authorised it, you should be able to get the Webhook URL. You will need this URL to configure your Durable Function to post an approval request message every time an application has been received.</li>
	<li><strong>Message Template</strong>: To be able to send interactive button messages to Slack we need to have the appropriate <a href="https://api.slack.com/docs/messages/builder" target="_blank" rel="noopener noreferrer">message template</a>.</li>
	<li><strong>Interactive Components</strong>: The webhook configured above enables you to post messages to Slack. Now you need a way to get the response from Slack, for this you can use <a href="https://api.slack.com/docs/message-buttons" target="_blank" rel="noopener noreferrer">interactive message buttons</a>. To configure the interactive message button, you must provide a request URL. This request URL will be the URL of the <em>HttpTrigger</em> Azure function that will handle the approval selection.</li>
</ul>
</li>
	<li>Azure Storage Account: The solution requires a Storage Account with 3 blob containers: <em>requests</em>, <em>approved</em>, and <em>rejected</em>. The <em>requests</em> container should have <strong>public access level</strong> so blobs can be viewed without a SAS token. For your own solution, you could make this more secure.</li>
</ul>
<h2>Solution Overview</h2>
The figure bellow, shows an overview of the solution we will build based on Durable Functions. As you can see, the workflow is very similar to the one implemented previously. Pictures of the aspiring rabbits are to be dropped in an Azure storage account blob container called <em>requests</em>. At the end of the approval workflow, pictures should be moved to the <em>approved</em> or <em>rejected</em> blob containers accordingly.

<img class=" size-full wp-image-1171 aligncenter" src="/assets/img/2018/06/20-solution-overview1.png" alt="20 Solution Overview" width="784" height="727" />

The steps of the process are described as follows:
<ol>
	<li>The process is being triggered by an Azure Function with the <em>BlobTrigger</em> input binding monitoring the <em>requests</em> blob container. This function also implements the <em>DurableOrchestrationClient</em> attribute to instantiate a Durable Function orchestration</li>
	<li>The <em>DurableOrchestrationClient </em>starts the orchestration.</li>
	<li>Then, the Durable Function orchestration calls another function with the <em>ActivityTrigger</em> input binding, which is in charge of sending the approval request to Slack as a <a href="https://api.slack.com/docs/message-buttons" target="_blank" rel="noopener noreferrer">Slack interactive message</a>.</li>
	<li>The interactive message is posted on Slack. This interactive message includes a <em>callbackId</em> field in which we send the orchestration instance id.</li>
	<li>Then, in the orchestration, a timer is created so that the approval workflow does not run forever, and in case no approval is received before a timeout, the request is rejected.</li>
	<li>The (rabbit) user receives the interactive message on Slack, and decides whether the aspiring rabbit deserves to join Furry Models, by clicking either the Approve or Reject button. The slack interactive message button will send the response to the configured URL on the Interactive Component of the Slack App (this is the URL of the <em>HttpTrigger</em> function which handles the Slack approval response). The response contains the <em>callbackId</em> field which will allow the correlation in the next step.</li>
	<li>The <em>HttpTrigger</em> function receives the response which contains the selection and the <em>callbackId</em>. This function gets the orchestration instance id from the <em>callbackId</em> and checks the status of that instance; if it’s not running, it returns an error message to the user. If it’s running, it raises an event to the corresponding orchestration instance.</li>
	<li>The corresponding orchestration instance receives the external event.</li>
	<li>The workflow continues when the external event is received or when the timer finishes; whatever happens first. If the timer finishes before a selection is received, the application is automatically rejected.</li>
	<li>The orchestration calls another <em>ActivityTrigger</em> function to move the blob to the corresponding container (<em>approved</em> or <em>rejected</em>).</li>
	<li>The orchestration finishes.</li>
</ol>
A sample of the Slack interactive message is shown below.

<img class=" size-full wp-image-1172 aligncenter" src="/assets/img/2018/06/31-sample-message.png" alt="31 Sample Message" width="540" height="423" />

Then, when the user clicks on any of the buttons, it will call the <em>HttpTrigger</em> function described in the step 7 above. Depending on the selection and the status of the orchestration, it will receive the corresponding response:

<img class="alignnone size-full wp-image-1173" src="/assets/img/2018/06/32-sample-response.png" alt="32 Sample Response" width="505" height="88" />
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

<h3>SendApprovalRequestViaSlack.cs</h3>
<em>ActivityTrigger</em> function which sends the approval request via Slack as an Interactive Message (Step 3 above).

<p/>
<script src="https://gist.github.com/pacodelacruz/f11dc8673bbbc9888afc396d5c554b6b.js"></script>
<p/>

<h3>ProcessSlackApprovals.cs</h3>
<em>HttpTrigger</em> function that handles the response of the interactive messages from Slack (Step 7 above).

<p/>
<script src="https://gist.github.com/pacodelacruz/eba5c71be9ef73a22679f351b46f9839.js"></script>
<p/>

<h3>MoveBlob.cs</h3>
<em>ActivityTrigger</em> function that moves the blob to the corresponding container (Step 10 above).

<p/>
<script src="https://gist.github.com/pacodelacruz/f95e57d1f78e6e42898297cb9adcd5bd.js"></script>
<p/>

<h3>local.settings.json</h3>
These are the settings which configure the behaviour of the solution, including the storage account connection strings, the Slack incoming webhook URL, templates for the interactive message, among others.

You would need to implement these as app settings when deploying to Azure
<p/>
<script src="https://gist.github.com/pacodelacruz/fa5eb1415fbf0adb3be10a7eb369f9e1.js"></script>
<p/>
<h2>Wrapping up</h2>
In this post, I’ve shown how to implement an Approval Workflow (Human Interaction pattern) on Azure Durable Functions with Slack. On the way, we've also seen how to create Slack Apps with interactive messages. What you read here can easily be implemented using your own custom APIs. What we've covered should allow you to build serverless approval workflows on Azure with different means of approval. I hope you’ve found the posts of this series useful.

Happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
