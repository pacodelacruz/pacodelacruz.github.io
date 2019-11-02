---
layout: post
title: Your Own Serverless Request Bin with Durable Entities and Azure Durable Functions 2.0
date: 2019-10-22 10:00
author: Paco de la Cruz
comments: true
category: Azure Functions
tags: [Azure Functions, Azure Durable Functions, Serverless, Dependency Injection]
---

<base target="_blank"/>

<p>This post is part of a series</p>
<ol>
<li><a href="/2019/09/26/serverless-request-bin-azure-functions" rel="noopener" target="_blank"><span>Your Own Serverless Request Bin with Azure Functions</span></a></li>
<li><span>Your Own Serverless Request Bin with Azure Durable Functions (this)</span><br><span></span></li>
</ol>
<p>In a <a href="/2019/09/26/serverless-request-bin-azure-functions" rel="noopener" target="_blank">previous post</a>, I shared how you can deploy a Serverless Request Bin using Azure Functions. I also shared how I built it using a memory cache as a persistence layer. With that approach, I was able to leverage the new constructor <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection" rel="noopener" target="_blank">Depending Injection capabilities</a> of Azure Functions that allowed me to use a memory cache across multiple functions. However, the main limitation of that particular method was lack of control of the state lifecycle. I could try to keep the Function App in memory with a timer-triggered function,&nbsp; however, the platform can change the hosting instance at any given time, which would result in the unpredictable loss of the in-memory state.</p>
<!--more-->
<p>In this post, I will show how I leveraged a new feature, currently in preview, of Azure <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview" rel="noopener" target="_blank">Durable Functions</a> called <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities" rel="noopener" target="_blank">Durable Entities</a>. These allow me to make my Serverless Request Bin stateful. With this approach, now I can have full control of the lifecycle of the bins.</p>
<h2>The Solution</h2>
<p><img src="/assets/img/2019/10/DurableRequestBinRender.png" alt="Durable Request Bin Render" width="817" style="width: 817px;"></p>
<p>As with the previous post, this solution’s code is available on <a href="https://github.com/pacodelacruz/serverless-request-bin-durable-functions" rel="noopener" target="_blank">GitHub</a>. However, please consider this a sample solution for personal use. Compared to the <a href="https://github.com/pacodelacruz/serverless-request-bin-azure-functions" rel="noopener" target="_blank">previous solution</a>, this time, I used Durable Entities to persist the state of the bins. I kept the rendering to HTML on the <em>Serverless </em>side with <a href="https://github.com/dotliquid/dotliquid">DotLiquid</a> templates.</p>
<p>The Function App is composed of three HTTP-triggered functions described as follows:</p>
<ul>
<li><strong>PersistIntoBin:</strong>&nbsp;Persists HTTP requests into a particular bin specified as a path parameter.</li>
<li><strong>GetBin:</strong>&nbsp;Gets the HTTP request history for a particular bin specified as a path parameter.</li>
<li><strong>EmptyBin:</strong>&nbsp;Deletes the HTTP request history for a particular bin.</li>
</ul>
<p>Additionally, I introduced a new Durable Entity called <code>RequestBin.cs</code> shown below. As you can see, with very few lines of code, I was able to have a simple and intuitive persistence layer. Durable Entities are persisted in Azure Storage behind the scenes, but all of that is abstracted from us. &nbsp;</p>
<script src="https://gist.github.com/pacodelacruz/8f3732badb0f5bb99d1c26126e7ec963.js"></script>
<h2>Using Durable Entities</h2>
<p>During my refactoring to change the persistence layer from an in-memory cache to Durable Entities, I learned some characteristics of Durable Entities:</p>
<ul>
<li><strong>Simple Programming Model</strong>. Durable Entities offer two <a href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities#programming-models" rel="noopener" target="_blank">programming models</a> you can use. The <em>standard </em>function programming model allows you to have a more dynamic set of operations. Alternatively, in the <em>class-based </em>programming model, operations are defined as class methods. I prefer the latter, as calls are type-safe and you get the benefit of IntelliSense, when available, in your development environment.</li>
<li><strong>Simple and intuitive persistence layer</strong>. The model of the persistence layer is defined with private or public properties of the class containing the Durable Entity function. Each entity instance has a unique identifier assigned via code; this identifier is implicit. Entities are created automatically whenever they are called or signalled for the first time using the unique identifier. By using the <em>class-based</em> programming model, you can invoke defined public methods with type-safe calls to get or mutate the state of an entity. There is no need to design or implement a database, beyond the Durable Entity function class design.</li>
<li><strong>Opinionated Programming Model</strong>. It is also worth mentioning that while the programming model abstracts a lot of complexity from you, it is also opinionated. You need to be aware that you will need to use the programming model based on the Azure Functions bindings to leverage the abstraction of Durable Entities.</li>
</ul>
<h2>Benefits of the Stateful Serverless Request Bin</h2>
<p>The Serverless Request Bin that I showed in my previous post, provides some benefits, including:</p>
<ul>
<li><strong>Owning the Request Bin</strong>, thus having no risk of someone else capturing your sensitive HTTP requests.</li>
<li>Having a very <strong>cost-effective solution</strong>, considering the free executions you get and the low cost associated with the corresponding storage.</li>
<li><strong>No need of creating a bin in advance</strong>, as the platform will create one if the bin identifier is not currently in use.</li>
<li><strong>Flexible bin identifiers</strong>. You can assign any value you like to the bin identifier, as long as it is not longer than 36 characters, and has no special characters other than a hyphen, underscore or full-stop.</li>
<li><strong>Dark Mode</strong> ;)</li>
</ul>
<p>In addition to the above, with the Stateful Serverless Request Bin, we now also get:</p>
<ul>
<li><strong>Full control over the bin lifecycle</strong>. Now, with Durable Entities we have full control over the lifecycle of your request bins.</li>
</ul>
<h2>How to Deploy your own</h2>
<p>You need an Azure Subscription and access to deploy resources (contributor role) in a resource group. Similar to the previous version, deploying your own instance of the Stateful Serverless Request Bin is very easy. You just need to click on the button below, and this will take you to a page similar to the one shown after.</p>
<p><a href="https://deploy.azure.com/?repository=https://github.com/pacodelacruz/serverless-request-bin-durable-functions#/form/setup" rel="noopener" target="_blank"><img src="/assets/img/2019/10/deploybutton.svg" alt="Deploy To Azure button" width="161" style="width: 161px;"></a></p>
<p>At the time of writing, the deploy button option does not allow you to choose the region for a new resource group. Therefore, if you are planning your deployment in a new resource group, it is highly recommended to create the resource group in advance, so you can choose the region for it.</p>
<p>Please read the following section to understand the purpose of each of the settings.</p>
<p><img src="/assets/img/2019/10/DurableDeployToAzure.png" alt="Durable Deploy To Azure" width="940" style="width: 940px;"></p>
<h2>Configuration Options</h2>
<p>The configuration options and settings of the Stateful Serverless Request Bin are described in the table below. Some of these options are available only at deployment time, while others are also available after deployment as Application Settings of the Function App created.</p>
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
<p><strong>Can it be updated after deployment?&nbsp; </strong></p>
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
<p>Used to name the different components of the Serverless Request Bin; including the Function App, the consumption plan, Application Insights, and the Azure Storage account.&nbsp;</p>
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
<p>Defines the App Insights region (Given that Application Insights is not available in all regions, choose the closest region to the resource group).</p>
</td>
<td width="110">
<p>No</p>
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
<p>Filename of the <a href="https://help.shopify.com/en/themes/liquid/basics" rel="noopener" target="_blank">Liquid template</a> to use while rendering the request bin history. Currently, only the <code>DarkHtmlRender.liquid</code> template is provided. You can add your own liquid templates to the solution as well.</p>
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
<p>The maximum number of requests to store in the Request Bin.</p>
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
<p>The maximum number of characters to read and store from the request body. If a request body is larger than this limit, the body will be truncated.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
<tr>
<td width="200">
<p>Headers to Ignore</p>
</td>
<td width="291">
<p>Azure Functions add some headers to HTTP requests. If you prefer a cleaner request bin, you can ignore the specified HTTP headers. Specify the headers to ignore as a pipe-delimited list.</p>
</td>
<td width="110">
<p>Yes</p>
</td>
</tr>
</tbody>
</table>
<h2>How to use it</h2>
<p>Using the Stateful Serverless Request Bin is very easy. The usage operations are the same as the previous version. Once you have successfully deployed the Stateful Serverless Request Bin, you can use in the following:</p>
<ul>
<li><strong>Creating a new Request Bin</strong>. Request Bins are created on the fly when the first request to the Request Bin identifier is received. Bin identifiers can be up to 36 characters long and only supports alphabetic characters, numbers, dashes, underscores and full-stops.</li>
<li><strong>Sending HTTP Requests for inspection. </strong>Send a HTTP request using any method / verb to <code>https://&lt;yourfunctionapp&gt;.azurewebsites.net/&lt;binId&gt;</code><span style="background-color: transparent;"> </span><br>e.g. <code><strong>POST</strong> https://&lt;yourfunctionapp&gt;.azurewebsites.net/1234567890?a=1&amp;b=2</code></li>
</ul>
<ul>
<li><strong>Inspecting the Request Bin history. </strong>Retrieve information about a Request Bin history via a <code><strong>GET</strong></code> call including the Bin identifier to <code>https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/&lt;binId&gt;</code><br>e.g. <code><strong>GET<span style="background-color: transparent;"> </span></strong>https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/1234567890</code></li>
</ul>
<ul>
<li><strong>Deleting the Request Bin history. </strong>Similarly, delete the history of a Request Bin by including the Bin identifier in a <strong style="background-color: transparent;"><code>DELETE</code></strong>request to<br><code>https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/&lt;binId&gt;</code><br>e.g. <strong style="background-color: transparent;"><code>DELETE</code></strong><code>https://&lt;yourfunctionapp&gt;.azurewebsites.net/history/1234567890</code></li>
</ul>
<h2>Wrapping Up</h2>
<p>Throughout this post and the companion <a href="https://github.com/pacodelacruz/serverless-request-bin-durable-functions" rel="noopener" target="_blank">sample code</a>, I’ve shown how you can deploy and configure your own Stateful Serverless Request Bin. I also shared how I built it, leveraging Durable Entities. Feel free to use it for your own development tasks. I hope you found it useful, either as a HTTP request inspector or as sample code. Please feel free to raise issues or contribute by submitting your pull requests on <a href="https://github.com/pacodelacruz/serverless-request-bin-durable-functions" rel="noopener" target="_blank">GitHub</a>! :)</p>
<p>Happy coding!</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>