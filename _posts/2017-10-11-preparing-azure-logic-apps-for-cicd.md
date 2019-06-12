---
layout: post
title: Preparing Azure Logic Apps for CI/CD to Multiple Environments
date: 2017-10-11 12:05
author: Paco de la Cruz
comments: true
categories: [Azure, Azure iPaaS, CI/CD, Logic Apps]
---
<img class=" aligncenter" src="http://pacodelacruzag.files.wordpress.com/2017/10/101117_0432_preparingaz1.png" alt="" />
<h2> Introduction</h2>
Logic Apps can be created from the <a href="https://portal.azure.com">Azure Portal</a>, or using <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-deploy-from-vs">Visual Studio</a>. This works well if you want to create one Logic App at a time. However, if you want to deploy the same Logic App in multiple environments, e.g. Dev, Test, or Production, you want to do it in an automated way. <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates">Azure Resource Manager (ARM) Templates</a> allow you to define Azure Resources, including Logic Apps, for automated deployment to multiple environments in a consistent and repeatedly way. ARM Templates can be tailored for each environment using a <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates">Parameters</a> file.

The deployment of Logic Apps using ARM Templates and Parameters can be automated with different tools, such as, <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy">PowerShell</a>, <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy-cli">Azure CLI</a>, or <a href="http://www.integrationusergroup.com/continuous-integration-logic-apps-using-team-foundation-team-services/">VSTS</a>. In my projects, I normally use a <a href="https://docs.microsoft.com/en-us/vsts/build-release/actions/work-with-release-definitions">VSTS release definition</a> for this.

You probably have noticed that the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language">Logic App Workflow Definition Language</a> (the JSON code behind) has many similarities with the <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates">ARM Templates structure</a>, including the use of expressions and functions, variables, and parameters.

<strong>ARM Template expressions and functions</strong> are written within JSON string literals <strong>wrapped with square brackets [].</strong> ARM expressions and functions can appear in <strong>different sections of the ARM template</strong>, including the <strong>resources</strong> member, which might contain Logic Apps. The value of these expressions is <strong>evaluated at deployment time</strong>. More information <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates">here</a>.

<strong>Logic App expressions and functions</strong> are defined <strong>within the Logic App definition</strong> and might appear anywhere in a JSON string value. Logic Apps expressions and functions are <strong>evaluated at execution time</strong>. These are declared using the <strong>@ sign</strong>. More information <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language">here</a>.

These similarities can be confusing by themselves. I've seen that it's a quite common practice in ARM Templates with Logic Apps, to use ARM template expressions inside the Logic App definition. For example, using ARM parameters, ARM variables or ARM functions (like concat), within the definition of a Logic App. This might seem OK, as this is what you would normally do to tailor your deployment for any other Azure resources. However, in Logic Apps, this can be quite cumbersome. If you've done it, I'm almost sure that you know what I'm talking about.

In this post, I'll share some practices that I use to ease the preparation of Logic Apps for Continuous Integration / Continuous Delivery (CI/CD) to multiple environments using ARM Templates, <strong>when values inside the Logic App definition have to be customised per environment</strong>. If you don't have to change values within the Logic App definition, then you might not need to follow every step of this post.
<h2>Why it's not a good idea to use ARM template expressions inside a Logic App definition?</h2>
As I mentioned above, if when preparing you Logic Apps for CI/CD with ARM Templates, you have used ARM template expressions or functions inside a Logic App definition, you most probably have realised that it's quite troublesome. I personally don't like to do it that way for two reasons:
<ol>
	<li><strong>Editing the Logic App definition to include ARM Template expressions or functions is not intuitive</strong>. Adding ARM Template expressions and functions to be resolved at deployment time in a way that results in Logic Apps expressions and functions to be evaluated at execution time can be messy. Things can become harder when you have string functions in a Logic Apps, like <strong>@concat() </strong>that accept values that are to be obtained from ARM template expressions, like [parameters()] or [variables()]. I've heard and read of many people complaining about it.</li>
	<li><strong>Updating your Logic App after you have your ARM Template ready, requires more work</strong>. It's not unlikely that you would need to update your Logic App after you've prepared the ARM Template for it. Whether you need to fix a little bug found at testing, or you are required to change or add some functionality, the chances are that you would need to update the ARM template without the help of the Logic App Editor; and if you are unlucky, changes would touch those complex ARM template expressions inside your Logic App definition. Not very fun!</li>
</ol>
So, the question is, is it possible to create ARM Templates for Logic Apps that can be parameterised for multiple environments while avoiding using ARM template expressions <strong>inside</strong> the Logic App definition? Fortunately, it is :). Below, I describe how.
<h2>Scenario</h2>
For this post, I will work with a rather simple scenario: A Logic App that is triggered when a message in a Service Bus queue is received and posts the message to an https endpoint using basic auth. The <strong>endpoint url, the username and password will be different for each environment</strong>. Additionally, the Service Bus API Connection will have to be defined per environment.

This very simple workflow created using the Logic App editor is shown below:

<img class=" aligncenter" src="http://pacodelacruzag.files.wordpress.com/2017/10/101117_0432_preparingaz2.png" alt="" />

And the code behind this workflow is as follows:

<p/>
<script src="https://gist.github.com/pacodelacruz/14bff80489e0ea562675335ca3ba143b.js"></script>
<p/>

The code is very straight forward, <strong>but the endpoint, username and password are yet static</strong>. Not ideal for CI/CD!
<h2>Preparing the Logic App for CI/CD to be deployed to multiple environments</h2>
In this section, I'll show how you can prepare your Logic App for CI/CD to be deployed to multiple environments using ARM Templates, without having to use any ARM Template expressions or functions inside a Logic App definition.
<h3>1. Add Logic Apps parameters to the workflow for every value that is to be changed for each environment.</h3>
Similarly to ARM Templates, the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language">Logic App workflow definition language</a> accepts parameters. We can use these Logic Apps parameters to prepare our Logic App definition for CI/CD. We need to add a Logic App parameter for every value that is to be tailored for each environment. Unfortunately, at the time of writing, adding Logic App parameters can only be done <strong>via the code view</strong>.

Using the code view, we need to:
<ul style="margin-left:38pt;">
	<li><strong>Add the parameters definition with a default value</strong>, you should follow the same principles of parameters for ARM templates, but in this case, they are defined within the Logic App definition. The default value is the one you would use otherwise as static value at development time.</li>
	<li><strong>Update the workflow definition to use those parameters</strong> instead of the fixed values.</li>
</ul>
I've done this using the code view of the workflow shown above. The updated workflow definition is as follows.

<p/>
<script src="https://gist.github.com/pacodelacruz/0316be0b7b29f8ca17db5b06492e81f1.js"></script>
<p/>

After this update, at this point in time, the workflow should work just as before, but now, instead of having fixed values, you are using <strong>Logic Apps parameters with default values</strong>. If you are doing it for yours, you can test it yourself :)
<h3>2. Get the Logic App ARM Template for CI/CD.</h3>
Once the Logic App is ready, we can get the ARM Template for CI/CD. One easy way to do it is to use the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-deploy-from-vs">Visual Studio Tools for Logic Apps</a>. This requires Visual Studio 2015 or 2017, the latest Azure SDK and the <a href="https://marketplace.visualstudio.com/items?itemName=MicrosoftCloudExplorer.CloudExplorerforVS15Preview">Cloud Explorer</a>. You can also use the <a href="https://github.com/jeffhollan/LogicAppTemplateCreator">Logic App Template Creator PowerShell module</a>. More information on how to create ARM Templates for Logic Apps <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-create-deploy-template">here</a>.

The Cloud Explorer will allow you to log in to your Azure Subscription and see the supported Azure resources, including Logic Apps. When you expand the Logic Apps menu, you will see all the Logic Apps available for that subscription.

<img class=" aligncenter" src="http://pacodelacruzag.files.wordpress.com/2017/10/101117_0432_preparingaz3.png" alt="" />

Once you've found the Logic App you want to export, right click on it, and click on <strong>Open with Logic App Editor.</strong> This will open the Logic App Editor on Visual Studio.

<img class=" aligncenter" src="http://pacodelacruzag.files.wordpress.com/2017/10/101117_0432_preparingaz4.png" alt="" />

In addition to allowing to edit Logic Apps on Visual Studio, the Visual Studio Logic App Tools let you to download the ARM Template that includes the Logic App. You just need to click the <strong>Download </strong>button<strong>, </strong>and<strong>
</strong>you will get an almost ready-to-deploy ARM Template. This functionality <strong>exports the Logic App API Connections</strong> as well. <strong>
</strong>

For this workflow, I got an ARM Template as follows:

<p/>
<script src="https://gist.github.com/pacodelacruz/27416dc0667d4b39edc189d2d2f31d84.js"></script>
<p/>

As you can see, this ARM Template includes
<ul>
	<li><strong>ARM Template parameters definition</strong>. This is where we define the ARM Template parameters. We can set a default value. The actual value for each environment is to be set on the ARM Parameters file.</li>
	<li><strong>Logic App parameters definition</strong>: These are declared <strong>within the definition </strong>of the Logic App. These are the ones we can define using the code view of the Logic App, as we did above.</li>
	<li><strong>Logic App parameters value set</strong>: Here is where we <strong>set the values for the parameters for the Logic App</strong>. This section is declared <strong>outside of the <em>definition</em> property</strong> of the Logic Apps.</li>
</ul>
The structure of the ARM Template can be seen in the picture below.

<img class=" aligncenter" src="http://pacodelacruzag.files.wordpress.com/2017/10/101117_0432_preparingaz5.png" alt="" />
<h3>3. Set the Logic App parameters values with ARM Template expressions and functions.</h3>
Once we have the ARM Template, we can set the Logic App parameters values with ARM expressions and functions, including ARM parameters or ARM variables. I've done it with my ARM Template as shown below.

Before you check the updated ARM Template, some things to note:
<ul>
	<li>I added comments to the ARM Template <strong>only</strong> to make it easier to read and understand, but I don't recommend it. Comments <a href="https://stackoverflow.com/questions/244777/can-comments-be-used-in-json">are not supposed to be supported in JSON documents</a>, however, Visual Studio and ARM Templates allow it.</li>
	<li>I used the "-armparam" and "-armvar" suffixes on the ARM Template parameters and variables correspondingly. I did it only to show a clear distinction between ARM Template parameters and variables and Logic Apps parameters and variables. But the notation is sufficient (Using <strong>square brackets []</strong> for ARM Template expressions and functions, and <strong>@ sign</strong> for those of Logic Apps).</li>
	<li>I just used ARM Template parameters and variables to set the values of Logic App parameters, but you can use any other ARM Template function or expression that you might require to set Logic App parameter values.</li>
</ul>
<p/>
<script src="https://gist.github.com/pacodelacruz/5f404d6ec3caab2c543da99dda7edf6a.js"></script>
<p/>

As you can see, now we are only using ARM Template expressions and functions <strong>outside the Logic App definition</strong>. This is much easier to read and maintain. Don't you think?
<h3>4. Prepare your ARM Parameters file for each environment.</h3>
Now that we have the ARM Template ready, we can prepare an ARM Parameters file for our deployment to each environment. Below I show an example of this.

<p/>
<script src="https://gist.github.com/pacodelacruz/faf200bb202341b0a2a30becebd9b8b3.js"></script>
<p/>
<h3>5. Work on your CI/CD Pipeline.</h3>
Once we have the ARM Template and the ARM Parameter files, we can automate the deployment using our preferred tool. If you want to use VSTS, <a href="http://www.integrationusergroup.com/continuous-integration-logic-apps-using-team-foundation-team-services/">this is a good video that shows you how</a>.
<h3>6. Deploy and enjoy.</h3>
Once you have deployed the ARM Template, you will be able to see the deployed Logic App. The Logic App parameters <strong>value set </strong>section is hidden, but if you execute it, you will see how the values have been set accordingly.
<h2>Do you want this to be easier?</h2>
You might be thinking, just as I am, that this process is not as intuitive as it should be, and is a bit time consuming. If you wish to ask the product team to improve this, you might want to vote for these user voice requests on the links below:
<ul>
	<li><a href="https://feedback.azure.com/forums/287593-logic-apps/suggestions/32849239-allow-calling-functions-and-nested-logic-apps-usin">Allow calling Functions and Nested Logic Apps using Logic Apps parameters within the Resource Id (URL)</a></li>
	<li>
<div style="background:white;"><a href="https://feedback.azure.com/forums/287593-logic-apps/suggestions/31818607-allow-adding-parameters-with-a-default-value-on-th">Allow adding parameters with a default value on the designer view</a></div></li>
	<li>
<div style="background:white;"><a href="https://feedback.azure.com/forums/287593-logic-apps/suggestions/31833445-allow-to-export-logic-app-with-parameters-to-be-se">Allow to export Logic App with parameters to be set with ARM Template parameters to ease CI/CD</a></div></li>
</ul>
<h2>Wrapping Up.</h2>
In this post, I've shown how to prepare your Logic Apps for CI/CD to multiple environments using ARM Templates in a more convenient way, i.e. without using ARM Template expressions or functions inside the Logic App definition. I believe that this approach makes the ARM Template of a Logic App much easier to read and to maintain.

This method not only <strong>avoids the need of writing complex ARM Template expressions inside a Logic App definition</strong>, but also allows you to update your Logic App in the Designer, after this has been deployed using ARM Templates, and later <strong>update the ARM Template by simply updating the Logic App definition section</strong>. That's much better, isn't it?

I hope you've found this post handy, and it has helped you to streamline the configuration of your CI/CD pipelines when using Logic Apps.

Do you have a different preferred way of preparing your Logic Apps for CI/CD? Feel free to leave your comments or questions below,

Happy clouding and automating!

P.S. And remember: "I will never use ARM Template expressions inside a Logic App definition" ;)
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
