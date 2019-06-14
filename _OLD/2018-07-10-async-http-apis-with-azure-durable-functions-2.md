---
layout: post
title: Async Http APIs with Azure Durable Functions (and Polling Client)
date: 2018-07-10 17:10
author: Paco de la Cruz
comments: true
categories: [APIs, Azure, Azure Functions, Durable Functions, Uncategorized]
---
<h2><img class="alignnone size-full wp-image-1187" src="/assets/img/2018/07/070618_1006_asynchttpap1.png" alt="070618_1006_AsyncHttpAP1.png" width="811" height="374" /></h2>
<h2>Introduction</h2>
Azure Durable Functions have support for <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview" target="_blank" rel="noopener noreferrer">different patterns</a>, which enable us to build serverless and stateful applications without worrying about the state management implementation details. One of these useful patterns is the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-3-async-http-apis" target="_blank" rel="noopener noreferrer">Asynchronous Http APIs</a>. This pattern comes in handy when client applications need to trigger long running processes exposed as APIs and do something else after a status is reached. You might be thinking that the best way to implement an async API is by enabling the API to raise an event once it finishes so the client can react to it. For instance via a webhook (Http callback request), <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-event-publishing" target="_blank" rel="noopener noreferrer">an Event Grid event</a>, or even using <a href="https://medium.com/@philippbauknecht/serverless-real-time-messaging-with-azure-functions-and-azure-signalr-service-c70e781ff3c3">SignalR</a>. That is true, however, in many cases, client apps cannot be modified, or there are security or networking restrictions which make polling the best or the only feasible alternative. In this post, I’ll describe how to implement the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-3-async-http-apis" target="_blank" rel="noopener noreferrer">Asynchronous Http API pattern</a> on Durable Functions based on the polling client approach.

Before we get into the details, it’s worth noting that this pattern can be used to implement <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-create-api-app#action-patterns" target="_blank" rel="noopener noreferrer">a custom trigger or action for Logic Apps with the polling pattern</a>.
<h2>Scenario</h2>
To demonstrate how to implement this pattern, I’ll use the scenario of “Call for Speakers” in a conference. In this, potential speakers submit a topic through an app, and they are very keen to know as soon as possible if they have been selected to present. Of course, in a real scenario there will be timelines and speakers would be notified before the agenda is published, so they wouldn’t need to keep continuously asking for the status of their submission. But I believe you get the idea that this is being used for illustration purposes only ;)
<h2>Solution Overview</h2>
The solution is based on Azure Durable Functions’ building blocks, including orchestration clients, the orchestration function, and activity functions. I’ve used dummy activity functions, as the main purpose of this post is to demonstrate the capabilities to implement an Asynchronous Http API. In case you want to understand how to implement this further, you can have a look at my previous posts on how to implement <a href="https://pacodelacruzag.wordpress.com/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid/" target="_blank" rel="noopener noreferrer">approval workflows on Durable Functions</a>.

The diagram below shows the different components of the solution. As you can see, there is a client app which submits the application by calling the <code>Submit</code> Azure function, and then can get the status of the application by calling a GET <code>Status</code> Azure function. The orchestration function controls the process and calls the activity functions.

<img class="alignnone size-full wp-image-1188" src="/assets/img/2018/07/070618_1006_asynchttpap2.png" alt="070618_1006_AsyncHttpAP2.png" width="792" height="484" />
<h2>Solution Components</h2>
The main components of the solution are described below. The comments in the code should also help you to better understand each of them. You can find the <a href="https://github.com/pacodelacruz/DurableFunctions-AsyncHttpApi" target="_blank" rel="noopener noreferrer">full solution sample code here</a>.
<h3>Submit Function</h3>
Http triggered function which implements the <code>DurableOrchestrationClient</code>. This function receives a “Call-for-Speakers” submission as a POST with a JSON payload as the request body. Then, it starts the submission processing orchestration. Finally, it returns to the client the URL to the endpoint to get the status using the <code>location</code> and <code>retry-after</code> http headers.

<p/><script src=" $1 "></script><p/>
<h3>Process Submission Function</h3>
Orchestration function which implements the<a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview#pattern-1-function-chaining" target="_blank" rel="noopener noreferrer"> function chaining pattern</a> for each of the stages of the submission approval process. It’s also in charge of updating the custom status of the instance as it progresses.

<p/><script src=" $1 "></script><p/>
<h3>Get Status Function</h3>
This is an Http function that allows clients to get the status of an instance of a Durable Function orchestration. You can get more details of this implementation <a href="https://docs.microsoft.com/en-gb/azure/azure-functions/durable-functions-http-api#async-operation-tracking" target="_blank" rel="noopener noreferrer">here</a>.

In many of the samples, they explain you how to use the <code><a href="https://docs.microsoft.com/en-us/sandbox/functions-recipes/durable-manage-orchestrations#exposing-http-management-apis" target="_blank" rel="noopener noreferrer">CreateCheckStatusResponse</a></code> method to generate the response to the client. However, make sure that that you fully understand what this returns. Given that the returned payload includes management endpoints with their corresponding keys, by providing that, you would allow the clients not only to get the status of a running instance, but also to 1) get the activity execution history, 2) get the outputs of the already executed activity functions, 3) send external events to the orchestration instance, and 4) terminate that instance. If you don’t want to give those privileges to the clients, you need to expose an http function that returns only the minimum information required. <span style="background-color:transparent;">In this wrapper function, you could also enrich the response if need be. If you only need to send the status back, I would recommend you to use the </span><code><a href="https://docs.microsoft.com/en-us/sandbox/functions-recipes/durable-manage-orchestrations#inspecting-the-status-of-an-orchestration" target="_blank" rel="noopener noreferrer">GetStatusAsync</a></code><span style="background-color:transparent;"> method instead. </span>

The sample function below makes use of the <code>GetStatusAsync</code> method of the orchestration client, and leverages the <code>location</code> and <code>retry-after</code> Http headers.

<p/><script src=" $1 "></script><p/>
<h3>Wrapping Up</h3>
In this post, I’ve shown how to implement Asynchronous Http APIs using Durable Functions with the polling consumer pattern. I’ve also explain the differences between using <code>CreateCheckStatusResponse</code> and <code>GetStatusAsync</code> methods. The former prepares a response which expose different management endpoints with a key that allow clients to do much more than just getting the status, while the latter just returns the instance status. I hope you’ve enjoyed and found useful this post.

If you are interested in more posts about patterns on Durable Functions, you can also check out:
<ul>
	<li><a href="https://pacodelacruzag.wordpress.com/2018/04/17/azure-durable-functions-approval-workflow-with-sendgrid/" target="_blank" rel="noopener noreferrer">Azure Durable Functions Pattern: Approval Workflow with SendGrid</a></li>
	<li><a href="https://pacodelacruzag.wordpress.com/2018/04/25/azure-durable-functions-approval-workflow-with-slack/" target="_blank" rel="noopener noreferrer">Azure Durable Functions Pattern: Approval Workflow with Slack</a></li>
</ul>
Happy clouding!
<p style="text-align:center;"><em>Cross-posted on <a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz" target="_blank" rel="noopener noreferrer">Deloitte Platform Engineering Blog</a>.
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>
