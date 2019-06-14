---
layout: post
title: Azure Functions or WebJobs? Where to run my background processes on Azure?
date: 2016-09-14 22:14
author: Paco de la Cruz
comments: true
categories: [Azure, Azure Functions, Azure WebJobs]
---

![functionsvswebjobs-icon](/assets/img/2016/09/functionsvswebjobs-icon1.jpg)

Introduction
------------

[Azure WebJobs](https://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/) have been a quite popular way of running background processes on Azure. They have been around [since early 2014](http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx). When they were released, they were a true PaaS alternative to Cloud Services [Worker Roles](https://azure.microsoft.com/en-us/documentation/articles/cloud-services-choose-me/) bringing many benefits like the [WebJobs SDK](https://azure.microsoft.com/en-us/documentation/articles/websites-dotnet-webjobs-sdk/), easy configuration of scalability and availability, a dashboard, and more recently all the advantages of Azure Resource Manager and a very flexible continuous delivery model. My colleague [Namit](https://blog.kloud.com.au/author/namittskloud/) previously [compared WebJobs to Worker Roles.](https://blog.kloud.com.au/2016/02/29/background-business-azure-worker-role-vs-web-job/)

Meanwhile, [Azure Functions](https://azure.microsoft.com/en-us/documentation/articles/functions-overview/) were [announced earlier this year](https://azure.microsoft.com/en-us/blog/introducing-azure-functions/) (march 2016). Azure Functions, or "Functions Apps" as they appear on the Azure Portal, are Microsoft's [Function as a Service (FaaS)](https://en.wikipedia.org/wiki/Function_as_a_Service) offering. With them, you can create [microservices](https://en.wikipedia.org/wiki/Microservices) or small pieces of code which can run synchronously or asynchronously as part of composite and distributed cloud solutions. Even though they are still in the making (at the time of this writing they are in [Public Preview version 0.5](https://blogs.msdn.microsoft.com/appserviceteam/2016/09/01/azure-functions-0-5-release-august-portal-update/)), Azure Functions are now an appealing alternative for running background processes. Azure Functions are being built on top of the WebJobs SDK but with the option of being deployed with a [Serverless](http://martinfowler.com/articles/serverless.html) model.

So, the question is: which option suits better my requirements to run background processes? In this post, I will try to contrast each of them and shade some light so you can better decide between the two.

Comparing Available Triggers
----------------------------

Let's see what trigger options we have for each:

**WebJobs Triggers**

WebJobs can be initiated by:

- messages in [Azure Service Bus](https://azure.microsoft.com/en-us/documentation/articles/websites-dotnet-webjobs-sdk-service-bus/) queues or topics (when created using the SDK and configured to run continuously),
- messages in an [Azure storage queue](https://azure.microsoft.com/en-us/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/) (when created using the SDK and configured to run continuously),
- [blobs](https://azure.microsoft.com/en-us/documentation/articles/websites-dotnet-webjobs-sdk-storage-blobs-how-to/) added to a container in an Azure Storage account (when created using the SDK and configured to run continuously),
- a [schedule](https://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/) configured with a CRON expression (if configured to run on-demand),
- [HTTP call](https://github.com/projectkudu/kudu/wiki/WebJobs-API) by calling the [Kudu WebJobs API](https://github.com/projectkudu/kudu/wiki/WebJobs-API) (when configured to run on-demand),

Additionally, with the [SDK extensions](https://github.com/Azure/azure-webjobs-sdk-extensions), the triggers below were added:

- file additions or changes in a particular directory (of the Web App File System),
- queue messages containing a record id of Azure Mobile App table endpoints,
- queue messages containing a document id of documents on DocumentDB collections, and
- third-party WebHooks (requires the [Kudu credentials](https://github.com/projectkudu/kudu/wiki/Deployment-credentials)).

Furthermore, the [SDK 2.0 (currently in beta)](https://github.com/Azure/azure-webjobs-sdk/wiki/Release-Notes) is adding support to:

- [events in an Event Hub](https://github.com/Azure/azure-webjobs-sdk/wiki/EventHub-support) stream,
- and even though not fully documented yet, FTP, SFTP, and other Cloud File Storage SaaS (such as OneDrive or DropBox) are supported as [Trigger bindings](https://github.com/Azure/azure-webjobs-sdk-templates/blob/eb2bd44f78bd68bfb1b234426260eda1c7536f2c/Bindings/bindings.json).

**Azure Functions Triggers**

Being Function Apps founded on WebJobs SDK, most of the triggers listed above for WebJobs are supported by Azure Functions. The options available at the time of writing this post are:

- messages in [Azure Service Bus](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-service-bus/) queues or topics,
- messages in Azure Storage queues,
- blobs added to containers in [Azure Storage](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-storage/),
- [schedule or timer](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-timer/): with CRON expressions,
- third-party [WebHooks](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-http-webhook/): (currently supported GitHub and Slack and custom WebHooks),
- [HTTP calls](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-http-webhook/) (functions called as APIs),
- events in an [Event Hub](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-event-hubs/) stream,
- queue messages containing a record Id of an [Azure Mobile App](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-mobile-apps/) table endpoints, and
- queue messages containing a document id of documents on [DocumentDB](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-documentdb/) collections.

And, currently provided as experimental options

- files added in Cloud File Storage SaaS platforms, such as Box, DropBox, OneDrive, FTP and SFTP (SaaSFileTrigger Template).

I believe the main difference between both in terms of triggers is the HTTP trigger option as detailed below:

**Authentication for HTTP Triggers**

Being WebJobs hosted on the Kudu SCM site, to trigger them via an HTTP call we need to use the [Kudu credentials](https://github.com/projectkudu/kudu/wiki/Deployment-credentials), which is not ideal. Azure Function Apps, on the other hand, provide more authentication options, including Azure Active Directory and third-party identity providers like Facebook, Google, Twitter, and Microsoft accounts.

**HTTP Triggers Metadata**

Functions support exposing their API metadata based on the [OpenAPI specification](https://openapis.org/specification), which eases the integration with consumers. This option is not available for WebJobs.

Comparing Outbound Bindings
---------------------------

After comparing the trigger bindings for both options, let's have a look at the output bindings for each.

**WebJobs Outputs**

The WebJobs SDK provides the following out-of-the-box output bindings:

- messages to Azure Service Bus queues,
- messages to Azure Storage queues,
- blobs to Azure Storage containers,
- events to [Event Hub](https://github.com/Azure/azure-webjobs-sdk/wiki/EventHub-support)
- push notifications to [Notification Hub](https://github.com/Azure/azure-webjobs-sdk-extensions),
- create items [in Azure Mobile Apps](https://github.com/Azure/azure-webjobs-sdk-extensions) tables,
- write documents to [DocumentDB](https://github.com/Azure/azure-webjobs-sdk-extensions),
- send emails via [SendGrid](https://sendgrid.com/),
- send SMS messages via [Twilio](https://twilio.com/)

**Azure Functions Outputs**

Function Apps can output messages to different means. Options available at the time of writing are detailed below:

- messages to Azure Service Bus queues,
- messages to [Azure Storage](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-storage/) queues,
- blobs to [Azure Storage](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-storage/) containers,
- HTTP response when called via the [Http trigger](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-http-webhook/),
- events in an [Event Hub](https://azure.microsoft.com/en-us/services/event-hubs/) stream,
- push notifications to [Notification Hub](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-notification-hubs/),
- write records to [Azure Mobile App](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-mobile-apps/) table endpoints, and
- write JSON documents on [DocumentDB](https://azure.microsoft.com/en-us/documentation/articles/functions-bindings-documentdb/),
- send emails via [SendGrid](https://sendgrid.com/),
- send SMS messages via [Twilio](https://twilio.com/)

In regard to supported outputs, the only difference between the two is that Azure Functions can return a response to the caller when triggered via HTTP. Otherwise, they provide pretty much the same capabilities.

Supported Languages
-------------------

Both, WebJobs and Function Apps support a wide variety of languages, including: bash (.sh), batch (.bat / .cmd), C#, F#, Node.Js, PHP, PowerShell, and Python.

So no difference here, probably just that WebJobs require some of them to be compiled as an executable (.exe) program.

Tooling
-------

WebJobs can be easily created using Visual Studio and the WebJobs SDK. For those WebJobs which are a compiled console application, you can run and test them locally, which comes in always very handy.

At the time of this writing, there is no way you can program, compile and test your Functions with Visual Studio. So you might need to code all your functions using the online [functions editor](https://azure.microsoft.com/en-us/documentation/articles/functions-reference/) which provides different templates. However, being Functions a very promising offering, I believe Microsoft will provide better tooling by the time they reach General Availability. In the meantime, here an [alpha version](https://www.npmjs.com/package/azurefunctions) tool and a [ScriptCs Functions emulator](https://www.nuget.org/packages/ScriptCs.AzureFunctions) by my colleague [Justin Yoo](https://blog.kloud.com.au/author/justinchronicles/).

Managing *"VM"* Instances, Scaling, and Pricing
-----------------------------------------------

This is probably the most significant difference between WebJobs and Azure Functions.

**WebJobs** require you to create and manage an Azure App Service (Web App) and the underlying App Service Plan (a.k.a. server farm). If you want your WebJob to run continuously, you need at least one instance on a Basic App Service Plan to support "Always On". For WebJobs you always need to pay for at least one VM Instance (as PaaS) regardless of this being used or idle. For WebJobs, the [App Service Plan Pricing](https://azure.microsoft.com/en-us/pricing/details/app-service/) applies. However, you can always deploy more than one App Service on one App Service Plan. If you have larger loads or load peaks and you need auto-scaling, then you would require at least a Standard App Service Plan.

Conversely, with **Azure Functions** and the [Dynamic Service Plan](https://azure.microsoft.com/en-us/documentation/articles/functions-scale/), the creation and management of a VM Instances and configuring scaling is all abstracted now. We can write functions without caring about server instances and get the benefits of a [Serverless](http://martinfowler.com/articles/serverless.html) architecture. Functions scale out automatic and dynamically as load increases, and scale down if decreases. Scaling up or down is performed based on the traffic, which depends on the configured triggers.

With functions, you get billed only for the [resources you actually use](https://azure.microsoft.com/en-us/pricing/details/functions/). The cost is calculated by the number of executions, memory size, and execution time measure as Gigabyte Seconds. If you have background processes which don't require a dedicated instance and you only want to pay for the compute resources in use, then a dynamic plan would make a lot of sense.

It's worth noting that if you already have an App Service Plan, which you are already managing and paying for, and has resources available, you can deploy your Functions on it and avoid extra costs.

One point to consider with the Dynamic Service Plan (Serverless model) is that as you don't control which instances are hosting your Azure Functions, there might be a [cold-startup overhead](https://msdn.microsoft.com/en-us/library/cc656914(v=vs.110).aspx). This wouldn't be the case for Functions running on your own App Service Plan (server farm) or WebJobs running as continuous on an "*Always On*" Web App where you have "dedicated" instances and can benefit from having your components loaded in memory.

Summary
-------

As we have seen, being Azure Functions built on top of the WebJobs SDK, they provide a lot of the previously available and already mature functionality but with additional advantages.

In terms of **triggers**, Functions now provide HTTP triggers without requiring the use of Publish Profile credentials and bringing the ability to have authentication integrated with Azure AD or third party identity providers. Additionally, functions give the option to expose an OpenAPI specification.

In terms of **binding outputs** and **supported languages**, both provide pretty much the same.

In regard to **tooling**, at the time of writing, WebJobs allow you to develop and test offline with Visual Studio. It is expected that by the time Azure Functions reach General Availability, Microsoft will provide much better tools for them.

I would argue that the most significant difference between Azure Functions and WebJobs is the ability to deploy Functions on the new [Dynamic Service Plan](https://azure.microsoft.com/en-us/documentation/articles/functions-scale/). With this service plan, you can have the advantages of not worrying about the underlying instances or scaling, it's all managed for you. This also means that you only pay for the compute resources you actually use. However, when needed or when you are already paying for an App Service Plan, you have the option of squeezing in your Functions in the same instances and avoid additional costs.

Coming back to the original question, which technology suits better your requirements? I would say that If you prefer a "serverless" approach in which you don't need or want to worry about the underlying instances and scaling, then Functions is the way to go (considering you are OK with the temporary lack of mature tools). But if you still favour managing your instances, WebJobs might be a better fit for you.

I will update this post once Functions reach GA and tools are there. Probably (just probably), Azure Functions will provide the best of both worlds and the question will only be whether to choose a Dynamic Service Plan or not. We will see!

Feel free to share your experiences or add comments or queries below.

<p style="text-align:center;"><em>Cross-posted on <a href="https://blog.kloud.com.au/author/pacodelacruzag/">Kloud's blog</a>.<br/>
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>
