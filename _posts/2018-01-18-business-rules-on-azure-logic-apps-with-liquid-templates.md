---
layout: post
title: Externalising Business Rules on Azure Logic Apps using Liquid Templates
date: 2018-01-18 09:40
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Azure, Business Rules Engine, Liquid Templates, Logic Apps]
---
<h2>Introduction</h2>
In Azure Logic Apps workflows, you can implement <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-use-logic-app-features#conditions-run-steps-only-after-meeting-a-condition">conditions</a> and <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-switch-case">switch cases</a> to control the flow based on runtime inputs and outputs. This functionality is quite useful, and in many cases, can be used to implement the business rules required. However, those business rules are inherent to the workflow, and when business rules change often, they would end up being hard to maintain.

<a href="https://www.microsoft.com/en-au/cloud-platform/biztalk">BizTalk Server</a>, which is the on-premises integration platform from Microsoft, provides a <a href="https://docs.microsoft.com/en-us/biztalk/core/business-rules-engine">Business Rules Engine</a> that allows you to define, build, manage and maintain business rules in a way that the integration orchestrations can be abstracted from changes on the business rules. Unfortunately, at the time of writing, in Logic Apps <a href="https://feedback.azure.com/forums/287593-logic-apps/suggestions/16520170-business-rule-engine-bre">there is not such a thing</a>.

<a href="https://www.microsoft.com/en-au/cloud-platform/biztalk">BizTalk Server</a>, which is the on-premises integration platform from Microsoft, provides a <a href="https://docs.microsoft.com/en-us/biztalk/core/business-rules-engine">Business Rules Engine</a> that allows you to define, build, manage and maintain business rules in a way that the integration orchestrations can be abstracted from changes on the business rules. Unfortunately, at the time of writing, in Logic Apps <a href="https://feedback.azure.com/forums/287593-logic-apps/suggestions/16520170-business-rule-engine-bre">there is not such a feature</a>.

Recently, Microsoft have released the support of <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform">Liquid Templates</a> to transform JSON and XML objects. These transformations are based on <a href="http://dotliquidmarkup.org/">DotLiquid</a>, the .NET Implementation of <a href="https://help.shopify.com/themes/liquid">Liquid templates</a> themes from Shopify.

If we wanted to externalise business rules from Logic Apps workflows, so that when they change we don’t need to update the workflow, we have two options:
<ul>
	<li>we could use Azure Functions and implement business rules as .NET code. <a href="https://github.com/NRules/NRules/wiki/Getting-Started">NRules</a> is an interesting open-source library for this, and now</li>
	<li>we could use Liquid Templates.</li>
</ul>
In this post, I’ll show how to implement business rules in Logic Apps using Liquid Templates.
<h2>Scenario</h2>
&nbsp;

<a href="/correlation-identifier-pattern-on-logic-apps/">Farm to Table</a>, the fresh produce drone delivery company, want to implement some business rules to be applied when receiving orders. All the order processing is currently handled by Logic Apps. These business rules can change over time, and they want to be able to update them with the minimal effort (think of development, testing and deployment). They have two main business rules: 1) to define whether an order must be manually approved and 2) applying discounts based on promotions. Promotions rules change much more often than approval rules. Because these rules change often, they want to externalise them from the Logic App workflow.

They want to implement the following business rules:
<ul>
	<li><strong>Manual Approval of Orders</strong>: Based on the current capacity of the drone fleet, orders with a total weight <strong>greater than 18 kg</strong> must be approved. Additionally, orders with a<strong> subtotal of less than $10 AUD</strong> require a manual approval as well.</li>
	<li><strong>Discount</strong>: currently, there is a promotion for orders placed on the <strong>mobile app</strong>. Other channels don’t have a discount.
<ul>
	<li>All orders coming from the <strong>mobile app</strong> would get at least <strong>5% discount.</strong></li>
	<li>Orders with a subtotal of <strong>$50 AUD or more</strong> and with a total weight of <strong>less than 10 Kg</strong> would get higher discount of at least <strong>7%</strong>. Additionally,
<ul>
	<li>Orders meeting the criteria above to be delivered to <strong>Zone 1</strong>  (Zip codes: 3000, 3001, 3002, 3003, 3004, 3005, 3006, 3007, 3008), get a <strong>15% discount</strong></li>
	<li>Orders meeting the criteria above to be delivered to <strong>Zone 2 </strong>(Zip codes: 3205, 3206, 3207), get a <strong>10% discount.</strong></li>
</ul>
</li>
</ul>
</li>
</ul>
<h2>Solution</h2>
As you can imagine, implementing these business rules as part of the Logic App workflow would add a lot of complexity, let alone the effort required when business rules, like promotions, change.

To implement these business rules, we will create a Liquid template, which will be abstracted from the Logic App workflow. This will allow us to update the business rules as required without impacting the workflow, thus we can reduce the time required for development, testing and deployment.
<h3>Orders</h3>
Orders received by Farm to Table follow the structure shown below

<p/>
<script src="https://gist.github.com/pacodelacruz/0ac4504f4648ecf55db175eda5c8e291.js"></script>
<p/>

<h3>Business Rules as a Liquid Template (version 1)</h3>
We will implement the business rules as a Liquid template. If you want to get familiar with Liquid templates, I would recommend to go to the <a href="https://help.shopify.com/themes/liquid">official documentation</a>. It’s worth noting that because Logic Apps implementation of Liquid is based on <a href="http://dotliquidmarkup.org/">DotLiquid</a>, not all the filters are supported or work exactly the same. For instance, <a href="https://help.shopify.com/themes/liquid/filters">filters</a> are called with the first letter in uppercase, as opposed to lower case in the Ruby implementation. Additionally, you need to be familiar with the creation of Integration Accounts and adding Liquid templates to it as <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform">described here</a>.

Below, there is the first iteration version of the business rules implementation in Liquid. As you can see, I’m creating two arrays of zip codes for the zone conditional. Everything else should be self-explanatory

<p/>
<script src="https://gist.github.com/pacodelacruz/5c1cb1a12bf5b7270d9afb84609cdc9b.js"></script>
<p/>

This Liquid template, should output a response like the one shown below:

<p/>
<script src="https://gist.github.com/pacodelacruz/e2fba844aa939e9312531e352e7cfb5c.js"></script>
<p/>

Having this response will allow us to use the <strong>requiresApproval</strong> boolean property, and the calculated <strong>discount</strong> later in the workflow, with all the business rules externalised from the workflow.
<h3>Business Rules as a Liquid Template (version 2)</h3>
As part of the requirements, Farm to Table, wants to be able to enable and disable discounts very easily. They also want to be able to easily change the channels to which discounts apply, zones, and thresholds for discount levels and approvals. To do that, we can refactor the Liquid template, so we can create a vocabulary with easy-to-maintain variables.

A sample of the refactored business rules is shown below

<p/>
<script src="https://gist.github.com/pacodelacruz/4ea16535ad8ab1ca1d96d419659a3a04.js"></script>
<p/>

This template will create exactly the same output, but now it should be easier to maintain.
<h3>The Workflow</h3>
The Order processing Logic App workflow is instantiated by an HTTP call. Then we need to apply business rules to know the discount and whether the order must be manually approved. To do so, we will use the <strong>Transform JSON to JSON</strong> action to invoke the Liquid template with the business rules. We pass the order as the <strong>content</strong> parameter and the name of the Liquid template as the <strong>map</strong>. The Liquid template must be already in the Integration Account and the Integration account must be assigned to the Logic App workflow. Then the workflow will proceed with further processing, including the manual approval when required. A snapshot of the first part of the workflow is shown as follows.

<img class="alignnone size-full wp-image-1126" src="/assets/img/2018/01/10-workflow-original.png" alt="10 Workflow Original"/>
<h3>Implementing a generic Business Rules Engine</h3>
Now, we know how to call externalised business rules using Liquid templates within Logic Apps workflows.

However, you could even implement a serverless Business Rules Engine using these tools, to be used not only from Logic Apps but other apps. This could be handy when business rules change often, and you prefer not to implement them as part of the application code.

To do so, we could deploy all our business rules as Liquid templates in an Integration Account, and then create an Http triggered Logic App workflow, which receives as body the JSON document for which business rules are to be applied. Additionally, this Logic App will need to receive the name of the Liquid template to be invoked.

In my Logic App, I am accepting the name of the Liquid template as an Http header. I named the custom header as <strong>X-BusinessRulesName.</strong> I used this header value to dynamically invoke the corresponding Liquid template in a Transform JSON to JSON action. The <strong>Transform JSON to JSON action</strong> is called <strong>Apply Business Rules.</strong> This Logic App will return the output of the Business Rule as the response body.

The Logic App workflow is shown below, including the code behind the Apply Business Rules (Transform JSON to JSON) action.

<img class="alignnone size-full wp-image-1127" src="/assets/img/2018/01/21-workflow-for-bre.png" alt="21 Workflow for BRE" />

This Http Logic App, together with business rules as Liquid templates, can be used as a Business Rules Engine. Quite handy, don't you think? :)
<h2>Considerations</h2>
There are two considerations to be taken into account when using Liquid Templates
<ul>
	<li>Liquid Templates, particularly the DotLiquid implementation might not have all the <a href="https://help.shopify.com/themes/liquid/tags/control-flow-tags">Control Flow</a> tags and <a href="https://help.shopify.com/themes/liquid/filters">Filters</a> required for your business rules. So you need to know what the limitations are.</li>
	<li>Liquid Templates are to be stored in an Integration Account. As described <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-pricing#pricing">here</a>, you can use an included (free) Integration Account with no additional cost to host your Liquid Templates. However, this one does not provide an SLA . For production workloads, a Basic or Standard Integration Account is suggested, and there is a <a href="https://azure.microsoft.com/en-us/pricing/details/logic-apps/">cost associated with it</a>.</li>
</ul>
<h2>Wrapping Up</h2>
In this post, I’ve shown how to implement externalised business rules on Logic Apps using Liquid templates. Additionally, we’ve seen how we can implement a generic Business Rules Engine using this same approach and dynamically invoking a particular business rule Liquid template. I hope you’ve found this post handy. Feel free to add your comments or ask any questions below.

Happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://engineering.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Engineering Blog</span></a></p>
