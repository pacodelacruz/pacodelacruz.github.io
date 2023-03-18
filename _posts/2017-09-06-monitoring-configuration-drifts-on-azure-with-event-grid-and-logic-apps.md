---
layout: post
title: Monitoring Configuration Drifts on Azure with Event Grid and Logic Apps
date: 2017-09-06 22:30
author: Paco de la Cruz
comments: true
category: Operations
tags: [Azure, Azure Automation, Azure Resource Manager, Event Grid, Logic Apps, Serverless]
---
<h2>Introduction</h2>
Azure <a href="https://azure.microsoft.com/en-au/services/event-grid/">Event Grid</a> is a first-class and hyperscale eventing platform with intelligent filtering that <a href="https://azure.microsoft.com/en-au/blog/introducing-azure-event-grid-an-event-service-for-modern-applications/">has recently been released</a> in preview and is a real game changer to build event-driven serverless apps on Azure. There have been many other posts, including <a href="/serverless-logging-alerting-with-service-fabric-azure-event-grid">this one</a> from my colleague <a href="https://twitter.com/daniel2me">Dan Toomey</a>, which highlights all the magic, features and benefits of this new offering on Azure. Thus, I don't pretend to reiterate over these on this post. My goal is, however, to try to show how to solve a requirement that I have heard more than a couple of times.

As mentioned <a href="https://azure.microsoft.com/en-au/services/event-grid/">here</a>, there are three typical scenarios where Azure Event Grid comes quite handy:
<ol>
	<li>Serverless Applications</li>
	<li>Application Integration, and</li>
	<li>Ops Automation</li>
</ol>
In this post, I will show how to build an Azure Ops Automation workflow to monitor configuration drifts on Azure resources using Event Grid and <a href="https://azure.microsoft.com/en-au/services/logic-apps/">Logic Apps</a>.
<h2>User Story</h2>
<ul>
	<li>As an Op, I want to be notified whenever there is a configuration drift on my Azure Resources.</li>
</ul>
Many organisations and teams have implemented Continuous Integration / Continuous Delivery (CI/CD), and they want to keep all their infrastructure and solution configuration as code, e.g. in a VSTS Git Repo. This has become a quite common practice, and the source of truth for all infrastructure and configuration as code must be in source control. The <a href="https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-control-configure">Role-Based Access Control (RBAC)</a> on Azure allows us to restrict changes to Azure resources to certain roles or users. Furthermore, Azure provides a way to <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-lock-resources">lock resources</a> at different levels (subscription, resources group or resource) to prevent users from deleting or modifying critical resources, thus avoiding configuration drifts.

However, in some exceptional cases, Ops or Admins might need to update their configuration without having the time to go through the process of updating the repo first and then triggering the CI/CD pipeline. These configuration drifts, will make the Git repo to be out-of-sync, which results in a very high risk of subsequent releases overwriting changes in the environment with unintended side effects. Thus, there is a need to monitor the Azure resources for configuration drifts, so the source of truth can be always kept in-sync.
<h2>Scenario</h2>
As you probably may have thought, the user story above is quite broad, so let's reduce its scope for demonstration purposes to:
<ul>
	<li>As an Op, I want to be notified whenever there is a configuration drift on my Azure Web App app settings.</li>
</ul>
For this scenario, we want to receive a notification whenever the app settings of an Azure App Service (Web App) are updated and are no longer aligned to the "desired state".

To show how this can be achieved with Azure Event Grid (and the Resource Groups Publisher) and Logic Apps, I will build a Logic App workflow that is triggered whenever the app settings of an Azure Web App are modified, and validate if these settings are different to a desired state.
<h2>Solution Prerequisites</h2>
This solution requires the following:
<ol>
	<li>
<div>An Azure App Service (Web App) with some app settings configured, in my case, I configured the app settings as follows:</div>
<div></div>
<img src="/assets/img/2017/09/090617_1144_monitoringc1.png" alt="" /></li>
	<li>
<div>A JSON definition of the "Desired State" of the app settings stored in an Azure Storage Blob Container, in my scenario this is as below:</div>
<pre><code>{
  "Setting-01": "expected-value-01",
  "Setting-02": "expected-value-02",
  "Setting-03": "expected-value-03"
}
</code></pre>
</li>
</ol>
<h2>Solution: A Logic App Workflow with an Event Grid Trigger</h2>
My solution implemented as a Logic App workflow with an Event Grid Trigger will follow the algorithm described below:
<ol>
	<li>Trigger the workflow when the app settings of the Web App are updated, using the Resource Groups Event Grid Event Publisher.</li>
	<li>Check the status of the Event, if it was not "Succeeded", then Terminate the Workflow. If it was "Succeeded", continue the workflow.</li>
	<li>Get the Updated State of the app settings of the Web App using the <strong>Azure Resource Connector</strong> of Logic Apps.</li>
	<li>Get the Desired State from a Blob container.</li>
	<li>
<div>Compare the New State with the Desired State. If the New State is different to the Desired State, then send a notification with the details of the event.</div></li>
</ol>
Below I described the two main steps of the workflow in more details
<h3>1. Configuring the Logic Apps Event Grid Trigger</h3>
<img src="/assets/img/2017/09/090617_1144_monitoringc2.png" alt="" />

To configure the trigger, we need to specify:
<ol>
	<li>Azure <strong>Subscription</strong></li>
	<li>Select the <strong>Resource Type</strong>, in this case Microsoft.Resources.resourceGroups, as we are monitoring Azure Resource Group changes.</li>
	<li>In the <strong>Resource Name</strong>, we enter the Resource Group name.</li>
	<li>In the <strong>Prefix</strong> <strong>Filter</strong>, we specify the ResourceId, in my case we are monitoring the App Settings of an Azure App Service.</li>
	<li>In this case, we don't need to set a <strong>Suffix Filter</strong>.</li>
	<li>And finally, we give a name to the topic subscription we are creating.</li>
</ol>
Once we execute a Logic App with this trigger, we should get a payload similar to the one shown below

<p/>
<script src="https://gist.github.com/pacodelacruz/775285ce34f1a131a2865affd464e481.js">.js"></script>
<p/>
<h3>2. Configuring the Logic App Azure Resource Manager Connector</h3>
Logic Apps provide an <strong>Azure Resource Manager connector</strong>, which allows us to do CRUD operations on Azure via Azure Resource Manager. In our scenario, we are going to use the <strong>Invoke Resource Operation</strong> to List the App Settings of a Web App. This will return the current (new) state of the Azure Resource, so we can compare it to the Desired State later on the workflow. In your own scenario, you can make use of other operations, like <strong>List Resources by Resource Group</strong>, <strong>Read a Resource</strong>, or <strong>Read a Resource Group</strong> to get the state of your Azure resources. The configuration applied for our scenario is as follows.

<img src="/assets/img/2017/09/090617_1144_monitoringc3.png" alt="" />
<h3>The Logic App Workflow</h3>
The implemented solution as a Logic App workflow is shown below. I hope it is self-explanatory. I included comments on each action to make it easier to follow.

<img src="/assets/img/2017/09/090617_1144_monitoringc4.png" alt="" />

Quite straightforward, isn't?

And in case you are wondering about the code behind, below is the same workflow showing the code view of the relevant actions.

<img src="/assets/img/2017/09/090617_1144_monitoringc5.png" alt="" />

If you want to have a look at the ARM template, including the full code behind of this Logic App, you can check it out <a href="https://gist.github.com/pacodelacruz/3787feb1068ddabf3d3390c837eb7ea4">here</a>.
<h2>Wrapping Up</h2>
In this post, I've shown how to monitor configuration drifts on Azure resources using Event Grid and Logic Apps. We've seen the <strong>Resource Groups Event Publisher</strong> of Event Grid in action and how it comes in very handy for Ops Automation scenarios. Now, you can start monitoring changes on your Azure resources by just creating subscriptions with the corresponding prefix and suffix filters on Logic Apps. What other useful Ops Automation scenarios can you think of using Event Grids and Logic Apps?

Please feel free to add your comments and questions below,

Happy eventing!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering Blog</span></a></p>
