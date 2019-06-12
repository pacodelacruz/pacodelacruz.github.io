---
layout: post
title: Implementing the Correlation Identifier Pattern on Stateful Logic Apps using the Webhook Action
date: 2017-07-17 10:26
author: Paco de la Cruz
comments: true
categories: [Azure, Azure iPaaS, Development, Enterprise Integration Patterns, Logic Apps, Serverless, Twilio, Webhook]
---
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/071617_2338_Implementin1.jpg" alt="" /></p>

<h2>Introduction</h2>
In many business scenarios, there is the need to implement long-running processes which first send a message to a second process and then pause and wait for an asynchronous response before they continue. Being this an asynchronous communication, the challenge is to correlate the response to the original request. The <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html">Correlation Identifier</a> enterprise integration pattern targets this scenario.

Azure Logic Apps provides a stateful workflow engine that allow us to implement robust integration workflows quite easily. One of the workflow actions in Logic Apps is the webhook action, which can be used to implement the Correlation Identifier pattern. One typical scenario in which this pattern can be used is when an approval step with a custom API (with a similar behaviour to the <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-create-api-office365-outlook">Send Approval Email</a> connector) is required in a workflow.

In this post, I will show how to implement the Correlation Identifier enterprise integration pattern on Logic Apps leveraging the webhook action.
<h2>Some background information</h2>
<h3>The Correlation Identifier Pattern</h3>
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/071617_2338_Implementin2.png" alt="" /></p>
<p style="text-align:center;"><span style="color:#44546a;font-size:9pt;"><em>Adapted from <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html">Enterprise Integration Patterns</a>
</em></span></p>
The <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html">Correlation Identifier</a> enterprise integration pattern proposes to add a unique id to the request message on the requestor end and return it as the correlation identifier in the asynchronous response message. This way, when the requestor receives the asynchronous response, it knows which request that response corresponds to. Depending on the functional and non-functional requirements, this pattern can be implemented in a <a href="http://whatisrest.com/state_management_explained/stateless_and_stateful">stateless or stateful</a> manner.
<h3>Understanding webhooks</h3>
A webhook is a service that will be triggered on a particular event and will result on an Http call to a RESTful subscriber. A much more comprehensive definition can be found <a href="https://sendgrid.com/blog/whats-webhook/">here</a>. You might be familiar with the configuration of webhooks with static subscribers. In a <a href="https://www.mexia.com.au/triggering-an-azure-logic-app-with-an-sms-using-twilio/">previous post</a>, I showed how to trigger a Logic App by an SMS message with a Twilio webhook. This webhook will sends all events to the same Http endpoint, i.e. a static subscriber.
<h3>The Correlation Identifier pattern on Logic Apps</h3>
If you have used the <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-create-api-office365-outlook">Send Approval Email Logic App Connector</a>, this implements the Correlation Identifier pattern out-of-the-box in a stateful manner. When this connector is used in a Logic App workflow, an email is sent, and the workflow instance waits for a response. Once the email recipient clicks on a button in the email, the particular workflow instance receives an asynchronous callback with a payload containing the user selection; and it continues to the next step. This approval email comes in very handy in many cases; however, a custom implementation of this pattern might be required in different business scenarios. The webhook action allow us to have a custom implementation of the Correlation Identifier pattern.
<h3>The Logic Apps Webhook Action</h3>
To implement the Correlation Identifier pattern, it's important that you have a basic understanding of the Logic Apps <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-native-webhook">webhook action</a>. Justin wrote some handy notes about it <a href="https://blog.kloud.com.au/2017/05/31/notes-for-logic-apps-around-webhook-actions/">here</a>. The webhook action of Logic Apps works with an instance-based, i.e. dynamic webhook subscription. Once executed, the webhook action generates an instance-based callback URL for the dynamic subscription. This URL is to be used to send a correlated response to trigger the continuation of the corresponding workflow. This applies the <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/ReturnAddress.html">Return Address</a> integration pattern.

We can implement the Correlation Identifier pattern by building a Custom API Connector for Logic Apps following the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-create-api-app">webhook subscribe and unsubscribe pattern of Logic Apps</a>. However, it's also possible to implement this pattern without the need of writing a Custom API Connector, as I'll show below.
<h2>Scenario</h2>
To illustrate the pattern, I'll be using a fictitious company called FarmToTable. FarmToTable provides delivery of fresh produce by drone. Consumers subscribe to the delivery service by creating their personalised list of produce to be delivered on a weekly basis. FarmToTable requires to implement an SMS confirmation service so that an SMS message is sent to each consumer the day before the scheduled delivery date. After receiving the text message, the customer must confirm within 12 hours whether they want the delivery or not, so that the delivery is arranged.
<h2>The Solution Architecture</h2>
As mentioned above, the scenario requires sending an SMS text message and waiting for an SMS response. For sending and receiving the SMS, we will be using <a href="https://www.twilio.com/">Twilio</a>. More details on working with Logic Apps and Twilio on <a href="https://www.mexia.com.au/triggering-an-azure-logic-app-with-an-sms-using-twilio/">one of my previous posts</a>. Twilio provides webhooks that are triggered when SMS messages are received. The Twilio webhooks only allow static subscriptions, i.e. calling one single Http endpoint. Nevertheless, the webhook action of Logic Apps requires the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-create-api-app">webhook subscribe and unsubscribe pattern</a>, which works with an instance-based subscription. Thus, we need to implement a wrapper for the required subscribe/unsubscribe pattern.

The architecture of this pattern is shown in the figure below and explain after.
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/03-Solution-Architecture.png" alt="" width="698" height="292" /></p>
Components of the solution:
<ol>
	<li><strong>Long-running stateful workflow</strong>. This is the Logic App that controls the main workflow, sends a request, pauses and waits for an asynchronous response. This is implememented by using the webhook action.</li>
	<li><strong>Subscribe/Unsubscribe Webhook Wrapper</strong>. In our scenario, we are working with a third-party service (Twilio) that only supports webhooks with static subscriptions; thus, we need to create this wrapper. This wrapper is composed by 4 different parts.</li>
</ol>
<ul style="margin-left:72pt;">
	<li><strong>Subscription store: </strong>A database to store the unique message Id and the instance-based callback URL provided by the webhook action. In my implementation, I'm using <a href="https://azure.microsoft.com/en-au/services/cosmos-db/">Azure Cosmos DB</a> for this. Nevertheless, you can use any other suitable alternative. Because the only message id we can send to Twilio and get back is the phone number, I'm using this as my correlation identifier. We can assume that for this scenario the phone number is unique during the day.</li>
	<li><strong>Subscribe and Start Request Processing API</strong>: this is a RESTful API that is in charge of starting the processing of the request and storing the subscription. I'm implementing this API with a Logic App, but you can use an Azure Function, an API App or a Custom Api App connector for Logic App.</li>
	<li><strong>Unsubscribe and Cancel Request Processing API</strong>: this is another RESTful API that is only going to be called if the webhook action on the main workflow times out. This API is in charge of cancelling the processing and deleting the subscription from the store. The unsubscribe step has a similar purpose to the <a href="https://msdn.microsoft.com/en-us/library/system.threading.cancellationtoken(v=vs.110).aspx">CancellationToken</a> structure used in C# async programming. In our scenario, there is nothing to cancel though. Like the previous API, I'm implementing this with a Logic App, but you can use different technologies.</li>
	<li><strong>Instance-based webhook</strong>: this webhook is to be triggered by the third-party webhook with a static subscription. Once triggered, this Logic App is in charge of getting the instance-based callback URL from the store and invoking it. After making the call back to the main workflow instance, the subscription is to be deleted.</li>
</ul>
<h2>The actual solution</h2>
To implement this solution, I'm going to follow the steps described below:

<strong>1. Configure my Twilio account</strong> to be able to send and receive SMS messages. More details <a href="https://www.mexia.com.au/triggering-an-azure-logic-app-with-an-sms-using-twilio/">here</a>.

<strong>2. Create a Service Bus Namespace and 2 queues</strong>. For my scenario, I'm using one inbound queue (ScheduledDeliveriesToConfirm) and one outbound queue (ConfirmedScheduledDeliveries). For your own scenarios, you can use other triggers and outbound protocols.

<strong>3. Create a Cosmos Db collection</strong> to store the instance-based webhook subscriptions. More details on how to work with Cosmos Db <a href="https://docs.microsoft.com/en-us/azure/cosmos-db/tutorial-develop-documentdb-dotnet">here</a>.
<ul>
	<li>Create Cosmos Db account (with the Document DB API).</li>
	<li>Create database</li>
	<li>Create collection.</li>
</ul>
<strong>4. Create the "Subscribe and Start Request Processing API". </strong>I'm using a Logic App workflow to implement this API as shown below. I hope the steps with their comments are self-explanatory.
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/071617_2338_Implementin4.png" alt="" /></p>

<ul>
	<li>The workflow is Http triggered. It expects, as the request body, the scheduled delivery details and the instance-based callback URL of the calling webhook action.</li>
	<li>The provided Http trigger URL is to be configured later in the webhook action subscribe Uri of the main Logic App.</li>
	<li>It stores the correlation on Cosmos Db. More information on the Cosmos Db connector <a href="https://github.com/jeffhollan/docdb-connector">here</a>.</li>
	<li>
<div>It starts the request processing by calling the Twilio connector to send the SMS message.</div></li>
</ul>
The expected payload for this API is as the one below. This payload is to be sent by the webhook action subscribe call on the main Logic App:
<pre><code>{
    "callbackUrl": "https://prod-00.australiasoutheast.logic.azure.com/workflows/guid/runs/guid/actions/action/run?params",
    "scheduledDelivery": {
        "deliveryId": "2c5c8390-b6c8-4274-b785-33121b01e219",
        "customer": "Paco de la Cruz",
        "customerPreferredName": "Paco",
        "phone": "+61000000000",
        "orderName": "Seasonal leafy greens and fruits",
        "deliveryAddressName": "Home",
        "deliveryDate": "2017-07-20",
        "deliveryTime": "07:30",
        "createdDateTime": "2017-07-19T09:10:03.209"
    }
} 
</code></pre>
You can have a look at the code behind <a href="https://github.com/pacodelacruz/CorrelationIdentifier/blob/master/01%20SubscribeAndProcessRequest.json">here</a>. Please use it just as a reference, as it hasn't been refactored for deployment.

<strong>5. Create the "Unsubscribe and Cancel Request Processing API". </strong>I used another Logic App workflow to implement this API. This API is only going to be called if the webhook action on the main workflow times out. The workflow is show below.
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/071617_2338_Implementin5.png" alt="" /></p>

<ul>
	<li>The workflow is Http triggered. It expects as the request body the message id so the corresponding subscription can be deleted.</li>
	<li>The provided Http trigger URL is to be configured later in the webhook action unsubscribe Uri of the main Logic App.</li>
	<li>
<div>It deletes the subscription from Cosmos Db. More information on the Cosmos Db connector <a href="https://github.com/jeffhollan/docdb-connector">here</a>.</div></li>
</ul>
The expected payload for this API is quite simple, as the one shown below. This payload is to be sent by the webhook action unsubscribe call on the main Logic App:
<pre><code>{
    "id": "+61000000000"
}
</code></pre>
The code behind is published <a href="https://github.com/pacodelacruz/CorrelationIdentifier/blob/master/02%20Unsubscribe.json">here</a>. Please use it just as a reference, as it hasn't been refactored to be deployed.

<strong>6. Create the Instance-based Webhook. </strong>I'm using another Logic App to implement the instance-based webhook as shown below.
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/071617_2338_Implementin6.png" alt="" /></p>

<ul>
	<li>The workflow is Http triggered. It's to be triggered by the Twilio webhook.</li>
	<li>The provided Http trigger URL is to be configured later in the Twilio webhook.</li>
	<li>It gets the message Id (phone number) from the Twilio message.</li>
	<li>It then gets the instance-based subscription (callback URL) from Cosmos Db.</li>
	<li>Then, it posts the received message to the corresponding instance of the main Logic App workflow by using the correlated callback URL.</li>
	<li>After making the callback, it deletes the subscription from Cosmos Db.</li>
</ul>
The code behind for this workflow is <a href="https://github.com/pacodelacruz/CorrelationIdentifier/blob/master/03%20InstanceBasedWebhook.json">here</a>. Please use it just as a reference, as it is not ready to be deployed.

<strong>7. Configure the Twilio static webhook. </strong>Now, we have to configure the Twilio webhook to call the Logic App created above when an SMS message is received. Detailed instructions in my previous <a href="https://www.mexia.com.au/triggering-an-azure-logic-app-with-an-sms-using-twilio/">post</a>.<strong>
</strong>

<strong>8. Create the long-running stateful workflow. </strong>Once we have the implemented the subscribe/unsubscribe webhook wrapper required for the Logic App webhook action, we can start creating the long-running stateful workflow. This is shown below.
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/071617_2338_Implementin7.png" alt="" /></p>
In order to trigger the Unsubscription API, the timeout property of the webhook action must be configured. This can be specified under the settings of the action. The Duration is to be configured the in <a href="https://en.wikipedia.org/wiki/ISO_8601">ISO 8601 duration format</a>. If you don't want to resend the request after the time out, you should turn off the retry policy.
<p style="text-align:center;"><img class="aligncenter" src="https://www.mexia.com.au/wp-content/uploads/2017/07/071617_2338_Implementin8.png" alt="" /></p>

<ul>
	<li>The workflow is triggered by messages on the <strong>ScheduledDeliveriesToConfirm </strong>Service Bus queue.</li>
	<li>
<div>Then the webhook action:</div>
<ul>
	<li>Sends the scheduled delivery message and the corresponding instance-based callback URL to the <strong>Subscribe and Start Request Processing Logic App</strong>.</li>
	<li>Waits for the callback from the <strong>Instance-based webhook</strong>. This would receive as an Http post the response send by the customer. If a response is received before the time out limit, the action will succeed and continue to the next action.</li>
	<li>If the webhook action times out, it calls the <strong>Unsubscribe and Cancel Request Processing Logic App</strong> and sends the message id (phone number); and the action fails so the workflow does not continue. However, if required, you could continue the workflow by configuring the RunAfter property of the subsequent action.</li>
</ul>
</li>
	<li>If a response is received, the workflow continues assessing the response. If the response is 'YES', it sends the original message to the <strong>ConfirmedScheduledDeliveries </strong>queue.</li>
</ul>
The code behind of this workflow is available <a href="https://github.com/pacodelacruz/CorrelationIdentifier/blob/master/04%20LongRunningStatefulWorkflow.json">here</a>. Please use it just as a reference only, as it hasn't been refactored for deployment.

Now, we have finished implementing the whole solution! :) You can have a look at all the Logic Apps JSON definitions <a href="https://github.com/pacodelacruz/CorrelationIdentifier">in this repository</a>.
<h2>Conclusion</h2>
In this post, I've shown how to implement the Correlation Identifier pattern using a stateful Logic App. To illustrate the pattern, I implemented an approval step in a Logic App workflow with a custom API. For this, I used Twilio, a third-party service, that offers a webhook with a static subscription; and created a wrapper to implement the subscribe/unsubscribe pattern, including an instance-based webhook to meet the Logic Apps webhook action requirements.

I hope you find this post useful whenever you have to add a custom approval step or correlate asynchronous messages using Logic Apps, or that I've given you an overview of how to enable the correlation of asynchronous messages in your own workflow or integration scenarios.

Feel free to add your comments or questions below, and happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
