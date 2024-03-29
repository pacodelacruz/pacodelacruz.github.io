---
layout: post
title: Your Own Serverless Request Bin with Azure Functions
date: 2019-09-26 10:00
author: Paco de la Cruz
comments: true
category: Azure Functions
tags: [Azure Functions, Serverless, Dependency Injection]
---

<base target="_blank"/>

<p>This post is part of a series</p>
<ol>
<li>Your Own Serverless Request Bin with Azure Functions (this)</li>
<li><a href="/serverless-request-bin-azure-durable-functions" rel="noopener" target="_blank"><span>Your Own Serverless Request Bin with Azure Durable Functions</span></a></li>
</ol>
<p>If you have developed or consumed HTTP APIs or webhooks, chances are that you have had the need of troubleshooting and inspecting HTTP requests. In the past, there was a very popular and handy free site called <em>Request Bin</em> (<u>requestb.in)</u> that allowed you to capture your HTTP requests and inspect their content, including the body, headers, and query params, etc. Unfortunately, due to ongoing abuse, the publicly hosted version of Request Bin was discontinued. After that, other sites started offering hosted alternatives to Request Bin. You also have the possibility of self-hosting and running your own Request Bin <a href="https://github.com/Runscope/requestbin" target="_blank" rel="noopener">in a container</a>.</p>
<p>One concern I have always had when using this type of free public site to inspect my HTTP requests is that I do not know how they handle my information. More often than not, I need to inspect requests that contain credentials, JWTs with access to sensitive resources and payloads with sensitive information. I cannot be sure that the provider is not logging those requests. Furthermore, it is not possible to guarantee that other external users will not get or guess my request bin identifier, which would give them full access to my HTTP request history.</p>
<p>Most of my development these days happens on Azure. Thus, the easiest option to solve this problem would be to host my own <a href="https://github.com/Runscope/requestbin" target="_blank" rel="noopener">Request Bin</a> on <a href="https://azure.microsoft.com/en-au/services/container-instances/" target="_blank" rel="noopener">Azure Container Instances</a>. However, this is not very cost-effective, as I would need to pay for the container instance even if it is idle for long periods, or dealing with the overhead of deleting it every time I finish inspecting HTTP request and redeploying when needed again. Being a developer, I started wondering how hard it could be to build my own Request Bin and host it on Azure Functions so that I would not need to pay when it was idle.</p>
<p>I wanted to try out first implementing it using Azure Functions with a memory cache that could be used across multiple requests (with the corresponding limitations). In this post (Part 1), I explain how I built a Serverless Request Bin using Azure Functions and a memory cache and how you can easily deploy your own so you can use it for your development tasks to inspect HTTP Requests in a secure and cost-effective manner.</p>
<p>In a future post, I'll explore how to do it using the new features of <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities" rel="noopener" target="_blank">Durable Entities </a>of Azure Durable Functions.&nbsp;&nbsp;</p>
<h2>The Solution</h2>
<p><strong><img src="/assets/img/2019/09/RequestBinRender.png" alt="RequestBinRender" width="833" style="width: 833px;"></strong></p>
<p>The solution’s code is available on <a href="https://github.com/pacodelacruz/serverless-request-bin-azure-functions" rel="noopener" target="_blank">GitHub</a>. Please don’t expect an enterprise-grade solution. Consider this a sample solution for personal use. When I was building it, I wanted to try out the new <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection" target="_blank" rel="noopener">Dependency Injection</a> capabilities of Azure Functions, and the ability to return not only an <code>ObjectResult</code> but HTML content from functions. I also used a <a href="https://github.com/dotliquid/dotliquid" rel="noopener" target="_blank">DotLiquid</a> template to transform objects to HTML.</p>
<p>The Azure Function App is composed of four functions. Functions are just wrappers that call services. Services are instantiated via Constructor Dependency Injection and configured during the <code>FunctionStartup</code>. The functions are described as follows:</p>
<ul>
<li><strong>PersistIntoBin</strong>. Persists HTTP requests into a particular bin specified as a path parameter.</li>
<li><strong>GetBin</strong>. Gets the HTTP request history for a particular bin specified as a path parameter.</li>
<li><strong>EmptyBin</strong>. Deletes the HTTP request history for a particular bin.</li>
<li><strong>KeepFunctionAppWarm</strong>. Keeps the Function App warm not only to avoid cold starts but also to keep the in-memory cache. This timer-triggered function does not guarantee against instance recycling or host replacement by the cloud provider.</li>
</ul>
<h2>Benefits of the Serverless Request Bin</h2>
<p>If you deploy your own instance of the Serverless Request Bin, you would get some benefits, including:</p>
<ul>
<li><strong>Owning the Request Bin</strong>, thus having no risk of someone else capturing your sensitive HTTP Requests.</li>
<li>Having a very <strong>cost-effective solution</strong>, considering the free executions you get and the low cost associated with the corresponding storage.</li>
<li><strong>No need of creating a Bin in advance</strong>, the platform will create one if the Bin identifier is not currently in used.</li>
<li><strong>Flexible bin identifiers</strong>. You can assign any value you like to the bin identifier, as long as it is not longer than 36 characters, and has no special characters other than hyphen, underscore or dot.</li>
<li><strong>Dark Mode</strong> ;)</li>
</ul>
<h2>How to Deploy your own</h2>
<p>Deploying your own instance on your Azure Subscription is very easy. You just need to click on the button below, and this will take you to a page similar to the one shown after.</p>
<p><a href="https://deploy.azure.com/?repository=https://github.com/pacodelacruz/serverless-request-bin-azure-functions#/form/setup" rel="noopener" target="_blank"><img src="/assets/img/2019/09/deploybutton.svg" alt="deploybutton" width="161" style="width: 161px;"></a></p>
<p>If you are planning to deploy the Serverless Request Bin in a new resource group, it is highly recommended creating the resource group in advance, so you can choose the region for the resource group. At the time of writing, the deploy button option does not allow you to choose the region for a new resource group.</p>
<p>Please read the following section to understand the purpose of each of the settings.</p>
<p><img src="/assets/img/2019/09/Deploy%20to%20Azure.png" alt="Deploy to Azure" width="947" style="width: 947px;"></p>
<h2>Configuration Options</h2>
<p>The configuration options and settings of the Serverless Request Bin are described in the table below. Some of these options are available only at deployment time, while others are also available after deployment as Application Settings of the Function App created.</p>
<table>
<tbody>
<tr>
<td width="200">
<p><strong>Setting</strong></p>
</td>
<td width="291">
<p><strong>Description</strong></p>
</td>
<td width="110">
<p><strong>Can be updated after deployment? &nbsp;</strong></p>
</td>
</tr>
<tr>
<td width="200">
<p>Directory</p>
</td>
<td width="291">
<p>Azure Active Directory Tenant that you want to use to deploy the solution</p>
</td>
<td width="110">
<p>No</p>
</td>
</tr>
<tr>
<td width="200">
<p>Subscription</p>
</td>
<td width="291">
<p>Azure subscription in which you want to deploy the solution</p>
</td>
<td width="110">
<p>No</p>
</td>
</tr>
<tr>
<td width="200">
<p>App Name</p>
</td>
<td width="291">
<p>Used to name the different components of the Serverless Request Bin, including the Function App, the consumption plan, Application Insights, and the Azure Storage account. &nbsp;</p>
</td>
<td width="110">
<p>No</p>
</td>
</tr>
<tr>
<td width="200">
<p>App Insights Region</p>
</td>
<td width="291">
<p>Given that Application Insights is not available in all regions, choose the closest region to the resource group.</p>
</td>
<td width="110">
<p>No</p>
</td>
</tr>
<tr>
<td width="200">
<p>Request Bin Provider</p>
</td>
<td width="291">
<p>App Setting to configure the Request Bin Provider to store the HTTP request history. Currently, only <code>Memory</code> is supported. In the future, other providers might be added. The “Memory” provider keeps the request bin history in a memory cache</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
<tr>
<td width="200">
<p>Request Bin Renderer</p>
</td>
<td width="291">
<p>App Setting to configure the Request Bin Renderer to return the Request Bin history to the user. Currently, only <code>Liquid</code> is supported. The <code>Liquid</code> renderer allows you to convert the Request Bin history object to HTML.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
<tr>
<td width="200">
<p>Request Bin Renderer Template</p>
</td>
<td width="291">
<p>File name of the <a href="https://help.shopify.com/en/themes/liquid/basics" rel="noopener" target="_blank">Liquid template</a> to use while rendering the request bin history. Currently, only the <code>DarkHtmlRender.liquid</code> template is provided. You can add your own liquid templates as well.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
<tr>
<td width="200">
<p>Request Bin Max Size</p>
</td>
<td width="291">
<p>Maximum number of request to store in the Request Bin.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
<tr>
<td width="200">
<p>Request Body Max Length</p>
</td>
<td width="291">
<p>Maximum number of characters to read and store of a request body. If a request body is larger than this limit, the body would be truncated.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
<tr>
<td width="200">
<p>Request Bin Sliding Expiration</p>
</td>
<td width="291">
<p>For the <code>Memory</code> (cache) Provider, the sliding expiration time in minutes. This setting is to configure the In-memory cache, however, the Function App host can be replaced or recycled at any time by the platform.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
<tr>
<td width="200">
<p>Request Bin Absolute Expiration</p>
</td>
<td width="291">
<p>For the <code>Memory</code> (cache) Provider, the absolute expiration time in minutes. This setting is to configure the In-memory cache, however, the Function App host can be replaced or recycled at any time by the platform.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
</tbody>
</table>
<h2>How to use it</h2>
<p>Using the Serverless Request Bin is very easy. Once you have successfully deployed the Serverless Request Bin, you can use it as follows:</p>
<ul>
<li><strong>Creating a new Request Bin</strong>. Request Bins are created on the fly when the first request to the Request Bin identifier is received. Bin identifiers can be up to 36 characters long and only support digits, letters and the dash, underscore and dot symbols.</li>
<li><strong>Sending HTTP Requests for inspection. </strong>Send a HTTP request using any of the methods to <code>https://&lt;yourfunctionapp&gt;.azurewebsites.net/&lt;binId&gt;</code><span style="background-color: transparent;"> </span><br>e.g. <code><strong>POST</strong> https://&lt;yourfunctionapp&gt;.azurewebsites.net/1234567890?a=1&amp;b=2</code></li>
</ul>
<ul>
<li><strong>Inspecting the Request Bin history </strong><code><strong>GET</strong> https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/&lt;binId&gt;</code><br>e.g. <code><strong>GET<span style="background-color: transparent;"> </span></strong>https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/1234567890</code></li>
</ul>
<ul>
<li><strong>Deleting the Request Bin history</strong><br><strong style="background-color: transparent;"><code>DELETE</code></strong><code>https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/&lt;binId&gt;</code><br>e.g. <strong style="background-color: transparent;"><code>DELETE</code></strong><code>https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/1234567890</code></li>
</ul>
<h2>What you can learn from this solution</h2>
<p>You can just use the solution and hopefully, it provides the value you want from it. However, you can also learn some things from the <a href="https://github.com/pacodelacruz/serverless-request-bin-azure-functions" rel="noopener" target="_blank">source code</a>, including:</p>
<ul>
<li><strong>Dependency Injection in Azure Functions</strong>: Constructor <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection" rel="noopener" target="_blank">Dependency Injection</a> is used to control which service implementations are in charge of the Request Bin management and rendering. Currently, there is only one implementation for each of the interfaces. However, the solution is prepared to support more implementations.</li>
<li><strong>Options Pattern in Azure Functions</strong>. The options pattern is described <a href="https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-2.2" rel="noopener" target="_blank">in detail here</a> and can be used in Azure Functions injecting configuration settings using the <code>IOptions&lt;T&gt;</code> interface via Dependency Injection.</li>
<li><strong>Returning HTML content from an HTTP triggered Azure Function. </strong>Most of the HTTP triggered Azure Functions samples we can find on the web return an <code>ObjectResult</code>. However, you can also return a <code>ContentResult</code>, in this case, we are returning content of type <code>text/html</code>.</li>
<li><strong>Rendering an object into HTML using DotLiquid</strong>. You can see how you can transform an object into HTML using <a href="https://help.shopify.com/en/themes/liquid/basics">Liquid Templates</a> and DotLiquid.</li>
</ul>
<h2>Limitations of this version of the Serverless Request Bin</h2>
<p>This solution should be considered a sample application and only targeted to personal use. There are some known limitations as listed below:</p>
<ul>
<li><strong>Ephemerality</strong>: On the current version of the Serverless Request Bin, the request history is persisted in a memory cache that can be recycled at any time. There is a timer trigger function that tries to keep the instance in memory, however, the hosting instance can be replaced or recycled at any time.</li>
</ul>
<h2>Future Work</h2>
<p>Being this a sample solution, there is a lot of room for improvement. The most important one being <strong>Durability.</strong>&nbsp;It is on my plans to write a second version of the Serverless Request Bin using <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities">Durable Entities</a>. Additionally, you might be thinking that it would be much better to have a lightweight Single Page Application that renders in a more elegant way the Request Bin history to the user. I agree, and contributions are more than welcome :)</p>
<h2>Wrapping Up</h2>
<p>In this post and the companion <a href="https://github.com/pacodelacruz/serverless-request-bin-azure-functions" rel="noopener" target="_blank">sample code</a>, I’ve shown how I built a Serverless Request Bin using Azure Functions and how you can deploy and configure your own. Feel free to use it for your own development tasks. I hope you find it helpful either as a solution or as a sample. Please provide feedback by raising issues or even contribute by submitting your pull requests on GitHub!&nbsp;</p>
<p>Happy coding!</p>


<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>