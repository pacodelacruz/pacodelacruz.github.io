---
layout: post
title: Triggering an Azure Logic App by SMS messages with Twilio
date: 2017-07-05 21:54
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Logic Apps, Twilio]
---
<h2>Introduction</h2>
SMS messaging has been a widely adopted way of communication over the last decades, not only for people but for organisations as well. Even though nowadays there are many messaging apps that are more popular and flexible than plain SMS, there are still scenarios in which businesses find SMS messaging a valuable way to communicate with their customers, employees or business partners. For instance, there are food chains that allow you to place your favourite order via SMS, some business provide order and shipment tracking via SMS, and there are many confirmation systems that utilise SMS so users can confirm their attendance to a meeting or appointment via a text message.

<a href="https://www.twilio.com/">Twilio</a> is a flexible and extensible PaaS offering that provides, among other communication services, that of SMS. Twilio allows you to send and receive SMS messages and integrate them into your business processes via APIs. Meanwhile, Logic Apps offers the <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-create-api-twilio">Twilio connector</a> which provides actions to <strong>send</strong>, <strong>list</strong> and <strong>get</strong> messages from Twilio. However, at the time of writing, this connector does not include a trigger, so that a Logic instance can be initiated when an SMS is received on a phone number of a Twilio account.

In this post, I will show how to trigger an Azure Logic App when an SMS message is received on a Twilio phone number.
<h2>Prerequisites</h2>
If you want to implement this scenario, you would need the following:
<ol>
	<li>A Twilio Account</li>
	<li>A Twilio Phone Number (you have to buy a phone number that can send and receive SMS messages, which has a monthly cost)</li>
	<li>Balance on your Twilio Account to allow for the Programmable SMS costs (receiving and sending SMS messages)</li>
</ol>
<h2>Implementing an SMS triggered Azure Logic App</h2>
The steps to implement an SMS triggered Logic App are described below:
<h3>1. Create a Logic App with an Http Request Trigger</h3>
As mentioned before, the Twilio connector does not offer a trigger to initiate a Logic App instance. However, Twilio provides a Webhook that allows you to subscribe your Http endpoint so that Twilio makes an Http Post whenever an SMS message is received. Therefore, we can create an Http triggered Logic App to receive these calls from the webhook. The quickest way to do is to start with the corresponding <strong>HTTP Triggered template. </strong>

<img src="https://www.mexia.com.au/wp-content/uploads/2017/07/070517_1134_Triggeringa1.png" alt="" />

Once the Logic Apps has been created, we should make sure that <strong>POST</strong> is selected as the method to trigger the workflow, so Twilio can call the Logic App endpoint and send the corresponding payload.
<h3>2. Getting the Http Url</h3>
After we save the Logic App containing the Http Request trigger, we get the workflow endpoint Url, including the corresponding SAS token. For your own solution, you might want to implement an intermediate layer and abstract the actual Logic App Url using <a href="https://azure.microsoft.com/en-us/services/api-management/">Azure API Management</a> or an <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-proxies">Azure Functions Proxy</a>. For this exercise, I will continue using the generated Url.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/07/070517_1134_Triggeringa2.png" alt="" />
<h3>3. Configuring the Twilio Webhook to Call the Http endpoint when a message comes in</h3>
Once we have the Logic App endpoint Url, on the Twilio Portal we configure the Twilio Phone Number to use a <strong>Webhook</strong> when <strong>a message comes in </strong>to call the Logic App endpoint with an <strong>Http Post. </strong>The Webhook will send the SMS details as form data to the Logic App.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/07/070517_1134_Triggeringa3.png" alt="" />

Now, we are all set to trigger our Logic App when an SMS is received.
<h3>4. Getting the SMS Message Details</h3>
You can inspect the message sent by the Twilio Webhook by sending an SMS to trigger your Logic App, and checking the output body of your run as shown below.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/07/070517_1134_Triggeringa4.png" alt="" />

The Twilio Webhook sends Url encoded form data ("$content-type": "application/x-www-form-urlencoded"). You should expect a request output body similar to the one shown below.

<p/>
<script src="https://gist.github.com/pacodelacruz/d551c4eded5f02723925f3c924d39d89.js"></script>
<p/>

Being this a Form Data Post request, we can use the function <strong>@triggerFormDataValue()</strong> to get each of the properties sent by Twilio, e.g. <strong>@triggerFormDataValue('Body')</strong> and <strong>@triggerFormDataValue('From')</strong>
<h3>5. Sending an SMS Response</h3>
To make the Logic App a bit more fun, and show more of the Twilio connector capabilities, I'll add one action to the workflow to reply back to the sender via SMS.

When we add the <strong>Twilio - Send Text Message</strong>, we first have to configure the API Connection. We have to give the connection a name and provide the Account Id, and Access Token. We can get these details from the Twilio Dashboard. We get the <strong>Account Id</strong> as the Account SID and the <strong>Access Token</strong> as the Auth Token.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/07/070517_1134_Triggeringa6.png" alt="" />

<img src="https://www.mexia.com.au/wp-content/uploads/2017/07/070517_1134_Triggeringa7.png" alt="" />

Once we have configured the API Connection, we are ready to continue configuring the actual action. We should be able to select from the drop down list the <strong>From </strong><strong>Phone Number </strong>in our Twilio account that we want to use to send the response message. In our scenario, we get the <strong>To Phone Number</strong> using <strong>@triggerFormDataValue('From')</strong>. For this exercise, I'm adding the original Text as part of the Reply Text as shown below.

<img class="alignnone wp-image-1021" src="https://www.mexia.com.au/wp-content/uploads/2017/07/12.-Send-SMS.png" alt="" width="601" height="259" />

And, that's pretty much it. We have implemented a Logic App that is triggered by an SMS which replies back to the sender. In case you are interested on seeing the code behind, you can have a look at it below:

<p/>
<script src="https://gist.github.com/pacodelacruz/09828b34c9c146f943aca81daec51687.js"></script>
<p/>
<h1>Conclusion</h1>
In this post, I've shown how to trigger a Logic App workflow by an SMS message using a Twilio account, how to get the message details, and how to reply back to the sender using the Twilio connector. Thanks to the Logic Apps capabilities, this implementation is quite easy and straight forward. Of course, you can extend and enrich your workflow making use of the many different connectors to get or push data from and to diverse apps and LOB systems. I hope you have found this post useful and it comes in handy when you are implementing solutions that require SMS integration. Feel free to add your comments or queries below.

Happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
