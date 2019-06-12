---
layout: post
title: Async Http APIs with Azure Durable Functions (and Polling Client)
date: 2018-07-11 21:17
author: Paco de la Cruz
comments: true
categories: [APIs, Azure, Azure Functions, Durable Functions]
---
<img src="/assets/img/2018/07/070618_1006_asynchttpap1.png" alt="" />
<h2>Introduction</h2>
Azure Durable Functions have support for <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview" target="_blank" rel="noopener">different patterns</a>, which enable us to build serverless and stateful applications without worrying about the state management implementation details. One of these useful pattern is the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-3-async-http-apis" target="_blank" rel="noopener">Asynchronous Http APIs</a>. This pattern comes in handy when client applications need to trigger long running processes exposed as APIs and do something else after a status is reached. You might be thinking that the best way to implement an async API is by enabling the API to raise an event once it finishes so the client can react to it. For instance via a webhook (Http callback request), <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-event-publishing" target="_blank" rel="noopener">an Event Grid event</a>, or even using <a href="https://medium.com/@philippbauknecht/serverless-real-time-messaging-with-azure-functions-and-azure-signalr-service-c70e781ff3c3">SignalR</a>. That is true, however, in many cases, client apps cannot be modified, or there are security or networking restrictions which make polling the best or the only feasible alternative. In this post, I’ll describe how to implement the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-3-async-http-apis" target="_blank" rel="noopener">Asynchronous Http API pattern</a> on Durable Functions based on the polling approach.

Before we get into the details, it’s worth noting that this pattern can be used to implement <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-create-api-app#action-patterns" target="_blank" rel="noopener">a custom trigger or action for Logic Apps with the polling pattern</a>.
<h2>Scenario</h2>
To demonstrate how to implement this pattern, I’ll use the scenario of “Call for Speakers” in a conference. In this, potential speakers submit a topic through an app, and they are very keen to know as soon as possible if they have been selected to present. Of course, in a real scenario there will be timelines and speakers would be notified before the agenda is published, so they wouldn’t need to keep continuously asking for the status of their submission. But I believe you get the idea that this is being used for illustration purposes only ;)
<h2>Solution Overview</h2>
The solution is based on Azure Durable Functions’ building blocks, including orchestration clients, the orchestration function, and activity functions. I’ve used dummy activity function, as the main purpose of this post is to demonstrate the capabilities to implement an Asynchronous Http API. In case you want to understand how to implement this further, you can have a look at my previous post on how to implement an <a href="https://pacodelacruzag.wordpress.com/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid/" target="_blank" rel="noopener">approval workflows on Durable Functions</a>.

The diagram below shows the different components of the solution.

<img src="/assets/img/2018/07/070618_1006_asynchttpap2.png" alt="" />
<h2>Solution and Components</h2>
The main components of the solution are described below. The comments in the code should also help you to better understand each of them. You can find the <a href="https://github.com/pacodelacruz/DurableFunctions-AsyncHttpApi" target="_blank" rel="noopener">full solution sample code here</a>.
<h3>Submit Function</h3>
Http triggered function which implements the <code>DurableOrchestrationClient</code>. This function receives a “Call-for-Speakers” submission as a POST with a JSON payload as the request body. Then, it starts the submission processing orchestration. Finally, it returns to the client the URL to the endpoint to get the status using the <code>location</code> and <code>retry-after</code> http headers.

<p/>
<script src="https://gist.github.com/pacodelacruz/ec72c686b589a00a6b0568938e737e27.js"></script>
<p/>
<h3>ProcessSubmission Function</h3>
Orchestration function which implements the<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-1-function-chaining" target="_blank" rel="noopener"> function chaining pattern</a> for each of the stages of the submission process. It’s also in charge of updating the custom status of the instance as it progresses.

<p/>
<script src="https://gist.github.com/pacodelacruz/0a68b4f12e060ac5c89dc46e5e455d39.js"></script>
<p/>

<h3>Get Status Function</h3>
This is an Http function that allows clients to get the status of an instance of a Durable Function orchestration. You can get more details of this implementation <a href="https://docs.microsoft.com/en-gb/azure/azure-functions/durable-functions-http-api#async-operation-tracking" target="_blank" rel="noopener">here</a>.

In many of the samples, they explain you how to use the <code><a href="https://docs.microsoft.com/en-us/sandbox/functions-recipes/durable-manage-orchestrations#exposing-http-management-apis" target="_blank" rel="noopener">CreateCheckStatusResponse</a></code> method to generate the response to the client. However, make sure that that you fully understand what this returns. Given that the return payload includes management endpoints with their corresponding keys, by providing that, you would allow the clients not only to get the status of a running instance, but also to 1) get the activity execution history, 2) get the outputs of the already executed activity functions, 3) send external events to the orchestration instance, and 4) terminate that instance. If you don’t want to give those privileges to the clients, you need to expose an http function that handles that response and returns only the minimum information required. <span style="background-color:transparent;">In this wrapper, you could also enrich the response if need be. If you only need to send back the status, I would recommend you to use the </span><code><a href="https://docs.microsoft.com/en-us/sandbox/functions-recipes/durable-manage-orchestrations#inspecting-the-status-of-an-orchestration" target="_blank" rel="noopener">GetStatusAsync</a></code><span style="background-color:transparent;"> method instead.</span>

The sample function below makes use of the <code>GetStatusAsync</code> method of the orchestration client, and leverages the <code>location </code>and <code>retry-after</code> Http headers.

<p/>
<script src="https://gist.github.com/pacodelacruz/2347e6c2a99f60f9d89fe31e32ea8506.js"></script>
<p/>

<h2>Wrapping Up</h2>
In this post, I’ve shown how to implement Asynchronous Http APIs using Durable Functions with the polling consumer pattern. I’ve also explain the differences between using <code>CreateCheckStatusResponse</code> and <code>GetStatusAsync</code> methods. The former prepares a response which expose different management endpoints with a key that allow clients to do much more than just getting the status, while the latter just returns the instance status. I hope you’ve enjoyed and found useful this post.

If you are interested in more posts about patterns on Durable Functions, you can also check out:
<ul>
	<li><a href="https://pacodelacruzag.wordpress.com/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid/" target="_blank" rel="noopener">Azure Durable Functions Pattern: Approval Workflow with SendGrid</a></li>
	<li><a href="https://pacodelacruzag.wordpress.com/2018/04/25/azure-durable-functions-approval-workflow-with-slack/" target="_blank" rel="noopener">Azure Durable Functions Pattern: Approval Workflow with Slack</a></li>
</ul>
Happy clouding!
<p style="text-align:center;"><em>Cross-posted on <a href="https://blog.mexia.com.au/author/paco-de-la-cruz" target="_blank" rel="noopener">Mexia’s Blog</a>. Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener">@pacodelacruz</a>.</em></p>
