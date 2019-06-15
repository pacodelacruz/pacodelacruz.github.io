---
layout: post
title: Implementing the Polling Consumer Pattern using Azure Logic Apps
date: 2017-05-12 18:27
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Azure Functions, Azure iPaaS, Enterprise Integration Patterns, Logic Apps, Microsoft iPaaS]
---
Introduction

When implementing integration projects, it's quite common that upstream systems don't have the capabilities to push messages to downstream systems, or that due to different constraints or non-functional requirements, the receivers are required to pull for messages from those systems. Gregor Hohpe describes in his book "Enterprise Integration Patterns" the <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/PollingConsumer.html">Polling Consumer Pattern</a>, in which a receiver is in charge of polling for messages from a source system. In this pattern, the receiver usually polls for messages with an incremental approach, i.e. polling only for changes from the source system; as opposed to getting a full extract. In most of these scenarios, the provider system does not keep any state on behalf of the receiver; thus, it is up to the receiver to keep a state which allows it to get changes since the previous successful poll.

Azure <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-what-are-logic-apps">Logic Apps</a> provides many trigger connectors which already implement the Polling Consumer Pattern out-of-the-box. For example, the Salesforce adapter, can trigger a workflow when a record is created or modified; the SharePoint adapter can initiate a workflow when a file or an item is created or modified; and the Gmail adapter can start a workflow when an email arrives. All these triggers work on a recurrent basis, e.g. every 5 minutes. For all these triggers, the Logic App adapter has to keep a trigger state or polling watermark which allows the connector to get only changes since the last poll. For example, the Gmail connector has to know if there are new emails since the last time it executed a poll. This trigger state or polling watermark should work even if we temporarily disable the Logic App. Even though there are many trigger connectors that make our life very easy, there might be scenarios in which a polling trigger connector is not available for a particular API or system. For example, what if we need to poll for changes from an Oracle database, from a custom API, or from an <a href="https://en.wikipedia.org/wiki/Atom_(standard)">Atom feed</a>? In this blog, I will show how to implement a custom Polling Consumer Pattern using Azure Logic Apps for those scenarios in which a trigger connector is not yet available.
<h2>Scenario</h2>
To illustrate the implementation of the Polling Consuming pattern, I will implement a fictitious scenario in which <em>'New Hire´ </em>events in an HR System are used to trigger other processes, e.g. identity and workstation provisioning. You might imagine that the end-to-end scenario could be implemented using the <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html">Publish-Subscribe Pattern</a>. That's true, however, on this blog I will focus only on the Polling Consumer interface on the Publisher side.

The HR System of my scenario provides an <a href="https://en.wikipedia.org/wiki/Atom_(standard)">Atom feed</a> for polling for updates. This Atom feed is exposed as a RESTful API endpoint and requires two query parameters: <em>'updated-min'</em> and <em>'updated-max'</em>. Both parameters are date-time in an ISO 8601 UTC format (e.g. yyyy-MM-ddTHH:mm:ss.fffZ). The lower bound (<em>updated-min</em>) is inclusive, whereas the upper bound (<em>updated-max</em>) is exclusive.

Even though my scenario is using a date-time polling watermark, the same principles can be used for other types of watermarks, such as Int64, base64 tokens, and hashes.

Coming back to my scenario, let's imagine that my Logic App is to be run every 10 mins, and I start it on the 1<sup>st</sup> of May at 8 AM UTC time; I would expect to send from my Logic App to the HR System http requests like the following:
<ul>
	<li><a href="https://hrsystem/atom/employee/newhire?updated-min=2017-05-01T08:00:00.000Z&amp;updated-max=2017-05-01T08:10:00.000Z">https://hrsystem/atom/employee/newhire?updated-min=2017-05-01T08:00:00.000Z&amp;updated-max=2017-05-01T08:10:00.000Z</a></li>
	<li><a href="https://hrsystem/atom/employee/newhire?updated-min=2017-05-01T08:10:00.000Z&amp;updated-max=2017-05-01T08:20:00.000Z">https://hrsystem/atom/employee/newhire?updated-min=2017-05-01T08:10:00.000Z&amp;updated-max=2017-05-01T08:20:00.000Z</a></li>
	<li><a href="https://hrsystem/atom/employee/newhire?updated-min=2017-05-01T08:20:00.000Z&amp;updated-max=2017-05-01T08:30:00.000Z">https://hrsystem/atom/employee/newhire?updated-min=2017-05-01T08:20:00.000Z&amp;updated-max=2017-05-01T08:30:00.000Z</a></li>
</ul>
The first request would return <em>New Hire</em> events that occurred between 8:00 AM and just before 8:10 AM. The next one from 8:10 to just before 8:20, and so on.
<h2>Components</h2>
To implement this pattern, I will use:
<ol>
	<li>A<strong> Logic App</strong>, to implement a custom Polling Consumer Pattern workflow.</li>
	<li>An <strong>Azure Storage Table</strong> to persist the polling watermark.</li>
	<li>An <strong>Azure Function</strong> to extract the current polling watermark.</li>
	<li>An <strong>Azure Function</strong> to update the polling watermark after the poll.</li>
</ol>
<h2>Show me the code!</h2>
After describing the scenario, let's start having fun with the implementation. To implement the pattern, I will follow the steps below:
<ol>
	<li>Create an Azure Resource Group</li>
	<li>Create an Azure Storage Account, create a Table, and populate the Table with my Polling Watermark</li>
	<li>Create an Azure Function App</li>
	<li>Develop and deploy the Azure Functions which will allow me to retrieve and update the Polling Watermark</li>
	<li>Develop the Azure Logic App</li>
</ol>
Each step is described in detail as follows,
<h3>1. Create an Azure Resource Group</h3>
You might already have an Azure Resource Group which contains other resources for your solution. If that's the case you can skip this step. I will start creating a new Resource Group for all resources I will be using for this demo. I'll name it <strong>'pacopollingconsumer-rgrp'</strong>
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin1.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>

<h3>2. Create an Azure Storage Account, create a Table, and populate the Table with the Polling Watermark</h3>
Once I have my Azure Resource Group, I'm going to create an Azure Storage Account. I'll use this Storage Account to create a Table to persist my polling watermark. I want to create a framework that can be used for more than one scenario; whether it's polling changes from different entities from the same source system, or from more than one source system. So, I'll prepare my table to handle more than one entity and more than one source system. I'll name the table '<strong>pacopollingconsumerstrg'</strong> and use the default settings.
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin2.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
Once I've created the Storage Account, I'll create a table. I'll use the <a href="http://storageexplorer.com/">Azure Storage Explorer</a> for this. Once, I've downloaded it, I will open it and add my Azure Account by signing in to Azure.
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin3.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
After signing in, I select my subscription. Then, I should be able to see all my existing Storage Accounts
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin4.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
I create a new Table by right clicking on the Tables branch
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin5.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
I'll name the new Table <strong>'PollingWatermark'</strong>
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin6.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
Once the Table has been created, I'll add a new Entity
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin7.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
As mentioned above, I want to be able to use this table to handle more than one entity and more than one source system. I'll use the Table <strong>PartitionKey</strong> to store the <strong>Source System</strong>, which for this demo I'll use <strong>'HRSystem'</strong>, and the <strong>RowKey</strong> to store the <strong>Entity</strong>, which will be <strong>'EmployeeNewHire'</strong>. I will create a new column of time <strong>DateTime</strong> to store my <strong>Watermark</strong>, and I will set my initial value. Bear in mind that the Azure Storage Explore works with local time, however, the value will be stored in UTC.
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin8.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
Cool, now we have our Azure Table Storage ready :)
<h3>3. Create the Azure Function App</h3>
At the time of writing this post, there is no Logic App connector for Azure Table Storage. There is already a user voice for it <a href="https://feedback.azure.com/forums/287593-logic-apps/suggestions/9995193-azure-table-storage-connector">here</a>, and if you want it, I would suggest you to vote for it. (I already did ;) ) In the absence of a Logic App connector for Azure Storage Table, we will be using an Azure Function App for it. I'll create an Azure Function App called <strong>'pacopollingconsumer-func'</strong> on my resource group, using the recently created storage account for its logs, and will use the <strong>consumption plan</strong> option, which is priced based on execution, as explained <a href="https://azure.microsoft.com/en-au/pricing/details/functions/">here</a>.
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin9.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
Once I've created my Function App, I'll download the publish profile which I'll be using later to publish my functions.
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin10.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>

<h3>4. Develop and Deploy the Azure Functions</h3>
Even though you can author your Azure Function from the portal, I really like the option of building and test my code locally with all the advantages of using Visual Studio. At the time of writing, Azure Function Tooling on Visual Studio supports creating C# Function as scripts (.csx files) on VS 2015 as <a href="https://blogs.msdn.microsoft.com/webdev/2016/12/01/visual-studio-tools-for-azure-functions/">described here</a>, and creating class libraries on Visual Studio 2015 and 2017 as <a href="https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/">shown here</a>. The use of compiled libraries brings some performance benefits, and will also make easier to transition to the planned Visual Studio 2017 tools for Azure Functions as <a href="https://blogs.msdn.microsoft.com/webdev/2017/04/14/azure-functions-tools-roadmap/">discussed here</a>. So, based on the recommendations, I've decided to use a class library project. I've already developed an Azure Functions class library project to implement this pattern, and made it available on GitHub, at <a href="https://github.com/pacodelacruz/PollingConsumer">https://github.com/pacodelacruz/PollingConsumer</a>. Even though you might want to reuse what I have developed, I strongly suggest you to get familiar with developing Azure Function Apps using class library projects, as <a href="https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/">described here</a>.

Some notes in regard to my <strong>'PacodelaCruz.PollingConsumer.FunctionApp'</strong> project
<ul>
	<li>In order to run Azure Functions locally, you have to install the Azure Functions CLI available <a href="https://github.com/Azure/azure-functions-cli/tree/master/src/Azure.Functions.Cli">here</a>.</li>
	<li>You will need to update the External program and Working Directory paths in the Project Properties / Web as <a href="https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/">described here</a>.<strong>
</strong></li>
</ul>
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin11.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>

<ul>
	<li>
<div>You might need to get the already referenced NuGet packages:</div>
<ul>
	<li>
<div style="text-align:justify;">Microsoft.WindowsAzure.ConfigurationManager</div></li>
	<li>
<div style="text-align:justify;">Microsoft.AspNet.WebApi</div></li>
	<li>
<div style="text-align:justify;">Microsoft.Azure.WebJobs.Host</div></li>
</ul>
</li>
</ul>
<strong>NOTE:</strong> I have found that the <strong>Newtonsoft.Json.JsonSerializerSettings</strong> on the <strong>Newtonsoft.Json version </strong><strong>10.0.2 </strong>does not work properly with Azure Functions. So, I'm using the <strong>Newtonsoft.Json version 9.0.1. </strong>I recommend you not to update it for the time being.
<ul>
	<li>You might want to update the <strong>host.json</strong> file according to your needs, <a href="https://github.com/Azure/azure-webjobs-sdk-script/wiki/host.json">instructions here</a>.</li>
	<li>You will need to update the <strong>appsettings.json</strong> file on your project to use your Azure Storage connection strings.</li>
</ul>
Now, let's explore the different components of the project:
<ul>
	<li>The <strong><a href="https://github.com/pacodelacruz/PollingConsumer/blob/master/PacodelaCruz.PollingConsumer.FunctionApp/Shared/PollingWatermarkEntity.cs">PollingWatermarkEntity</a></strong> class is used to handle the entities on the Azure Table Storage called '<strong>PollingWatermark</strong>'. If you are not familiar with working with Azure Storage Tables on C#, I would recommend you to have a read through the <a href="https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-how-to-use-tables">corresponding documentation</a>.</li>
</ul>
<p/>
<script src="https://gist.github.com/pacodelacruz/73b41a54465aa60f27b1669a28695ffb.js"></script>
<p/>
<ul>
	<li>The <a href="https://github.com/pacodelacruz/PollingConsumer/blob/master/PacodelaCruz.PollingConsumer.FunctionApp/Shared/PollingWatermark.cs"><strong>PollingWatermark</strong></a> class helps us to wrap the <strong>PollingWatermarkEntity</strong> and make it more user-friendly. By using the constructor, we are naming the PartitionKey as SourceSystem, and the RowKey as Entity. Additionally, we are returning another property called NextWatermark that is going to be used as the upper bound when querying the source system and when updating the Polling Watermark after we have successfully polled the source system.</li>
</ul>
<p/>
<script src="https://gist.github.com/pacodelacruz/a80e92d4ce70d820d943479191252be0.js"></script>
<p/>

Now, let's have a look at the Functions code:
<ul>
	<li>
<div><strong>GetPollingWatermark function</strong>. This Http triggered function returns a JSON object containing a DateTime Polling Watermark which is stored on an Azure Storage Table based on the provided <strong>'sourceSystem'</strong> and <strong>'entity'</strong> query parameters in the <strong>GET</strong> request. The function bindings are defined in the corresponding <a href="https://gist.github.com/pacodelacruz/24a782ba68e112440b701118c74a9d79">function.json</a> file.</div></li>
</ul>
<p/>
<script src="https://gist.github.com/pacodelacruz/830b23cd0f707c85d7a05e095467832d.js"></script>
<p/>
<ul>
	<li><strong>UpdatePollingWatermark function</strong>. This Http triggered function updates a DateTime Polling Watermark stored on an Azure Storage Table based on the payload in the JSON format sent as the body on the <strong>PATCH</strong> request. The JSON payload is the one returned by the GetPollingWatermark function. It uses the 'NextWatermark' property as the new value. The function bindings are defined in the corresponding
<a href="https://gist.github.com/pacodelacruz/1532dd7cab68be962d0f516efbfadcf0">function.json</a> file.</li>
</ul>
<p/>
<script src="https://gist.github.com/pacodelacruz/6e08d88ae7f17e52b9ffe16e35aecd5e.js"></script>
<p/>

Once you have updated the <strong>appsettings.json</strong> file with your own connection strings, you can test your functions locally. You can use <a href="https://www.getpostman.com/">PostMan</a> for this. On Visual Studio, hit F5, and wait until the Azure Functions CLI starts. You should see the URL to run the functions as shown below.
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin12.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
Once your project is running, you can then call the functions from PostMan by adding the corresponding query parameters. By calling the <strong>GetPollingWatermark</strong> function hosted locally, you should get the PollingWatermark, as previously set, as a JSON object.

<span style="background-color:#fafafa;">[<strong>GET</strong>] http://localhost:7071/api/GetPollingWatermark?<strong>sourceSystem=HRSystem&amp;entity=EmployeeNewHire</strong>
</span>
<p style="text-align:center;"><img class="" src="https://www.mexia.com.au/wp-content/uploads/2017/05/04.10-Test-GetPollingWatermark.png" alt="" width="753" height="519" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
To call the <strong>UpdatePollingWatermark</strong> function you need to use the <strong>PATCH</strong> method, specify that the body is of type <strong>application/json</strong>, and add in the Request Body, as obtained in the previous call. After calling it, you should see that the PollingWatermark has been updated based on the sent value. So far so good :)

<span style="background-color:#fafafa;">[<strong>PATCH</strong>] http://localhost:7071/api/<strong>UpdatePollingWatermark</strong>
</span>
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin14.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
After having successfully tested both functions, we are ready to publish our Function App project. To do so, we have to <strong>right click</strong> on the project, and then click <strong>Publish</strong>. This will allow us to import the Publish Profile that we downloaded previously in one of the first steps described above.
<p style="text-align:center;"><img src="/assets/img/2017/05/050217_1240_implementin15.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
After having successfully published the Azure Function, now we need to configure the connection strings on the Azure Function <strong>app settings</strong>. You will need to add a new app setting called <strong>PollingWatermarkStorage</strong> and set the value to the connection string of the storage account containing the PollingWatermark table.
<p style="text-align:center;"><img src="/assets/img/2017/05/050417_1244_implementin1.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
Now you should be able to test the functions hosted on Azure. You need to go to your Function App, navigate to the corresponding function, and get the function URL. As I set the authentication level to function, a function key will be contained in the URL.
<p style="text-align:center;"><img src="/assets/img/2017/05/050417_1244_implementin2.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
Bear in mind that we need to add the corresponding query params or request body. Your URLs should be something like:

[<strong>GET</strong>] https://pacopollingconsumer-func.azurewebsites.net/api/GetPollingWatermark?code=[functionKey]&amp;SourceSystem=HRSystem&amp;Entity=EmployeeNewHire

[<strong>PATCH] </strong>https://pacopollingconsumer-func.azurewebsites.net/api/UpdatePollingWatermark?code=[functionKey]
<h3>5. Develop the Logic App implementing the Polling Consumer workflow.</h3>
Now that we have implemented the required Azure Function, we are ready to build our Logic App. Below you can see the implemented workflow. I added the following steps:
<ul>
	<li><strong>Recurrent</strong> trigger</li>
	<li><strong>Function</strong> action, to call the <strong>GetPollingWatermark</strong> function with the GET method and passing the query parameters.</li>
	<li><strong>Parse JSON</strong> to parse the response body.</li>
	<li><strong>Http Request </strong>action to call the HR System endpoint passing the <strong>updated-min</strong> and <strong>updated-max</strong> parameters using the <strong>PollingWatermark</strong> and <strong>NextPollingWatermark</strong> properties of the parsed JSON.</li>
	<li><strong>Call a nested Logic App</strong>. Because the Atom Feed is a batch of entries, on the nested Logic App, I will be implementing debatching using <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-loops-and-scopes">SplitOn</a>. You can also use ForEach and send each entry to an Azure Service Bus queue or topic.</li>
	<li><strong>Function </strong>action to update the polling watermark for the next poll by calling the <strong>UpdatePollingWatermark</strong> function with the PATCH method and passing as request body the response obtained from the previous function call.</li>
</ul>
<p style="text-align:center;"><img src="/assets/img/2017/05/050417_1244_implementin3.png" alt="" /><span style="font-family:Times New Roman;font-size:12pt;">
</span></p>
In case it's of help, you can have a look at the code view of the Logic App. Just bear in mind that I removed some sensitive information.

<p/>
<script src="https://gist.github.com/pacodelacruz/4f78501a9981d5194a4256569b1b9ec7.js"></script>
<p/>
<h2>Other potential scenarios</h2>
Now that you have seen how I have implemented the custom Polling Consumer Patterns on Azure Logic Apps supported by Azure Functions and Azure Storage Tables, you should be able to implement the same pattern with slight variations for your own scenarios. You might need to use a numeric or a hash polling watermark, or instead of passing it to an API, you might need to pass it to a Function, that on its turn, queries a database for which a connector with a trigger is not yet available.
<h2>Conclusion</h2>
The Polling Consumer pattern is quite common in integration projects. Logic Apps provides different trigger connectors which implement the Polling Consumer pattern out-of-the-box. However, there are scenarios in which we need to connect to a system for which there is no trigger connector available yet. In these scenarios, we need to implement a custom Polling Consumer pattern. Through this post, we have seen how to implement a custom Polling Consumer Pattern using Azure Logic Apps together with Azure Functions and Azure Storage. To do so, we created an Azure Resource Group, we created an Azure Function, we developed Azure Functions using a C# class library project, we tested the functions locally, we deployed them to Azure, and configured and tested them on Azure as well. Finally, we developed a Logic App to implement the Polling Consumer pattern.

As discussed above, you can implement the same pattern with slight variations to accommodate your own scenarios with different requirements; and I really hope this post will make your implementation easier, faster, and more fun. Do you have any other ideas or suggestions on how to implement the custom Polling Consumer Pattern? Feel free to share them below <span style="font-family:Wingdings;">J</span>

I hope this has been of help, and happy clouding!

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
