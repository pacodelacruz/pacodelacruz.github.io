---
layout: post
title: Transforming JSON Objects in Logic Apps
date: 2017-05-18 21:22
author: Paco de la Cruz
comments: true
category: Logic Apps
tags: [Enterprise Integration Patterns, Logic Apps]
---
<h2>Introduction</h2>
Many integration scenarios require translating messages from one data model to another. This is described in the <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageTranslator.html">Message Translator</a> Enterprise Integration Pattern. Some of these might be:
<ul>
	<li>Translation between two different proprietary data models</li>
	<li>Translation between a proprietary data model and an industry standard specification, and vice-versa</li>
	<li>Translation between a proprietary data model and a <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html">Canonical Model</a>, and vice-versa</li>
	<li><a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/Normalizer.html">Normalisation</a> from different third-party formats to an internal Canonical Model</li>
	<li><a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/ContentFilter.html">Content Filtering</a> to remove unnecessary or sensitive information</li>
	<li><a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/DataEnricher.html">Content Enrichment</a> which can include different input messages from diverse sources</li>
	<li>Applying an <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/EnvelopeWrapper.html">Envelope Wrapper</a> to add metadata into the message</li>
</ul>
Logic Apps, as an Integration Platform as a Service (iPaaS), offers different capabilities that allow us to transform messages flowing through. The <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-transform">Enterprise Integration Transform Connector</a> allows us to use <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-maps">XSLT-based graphically-designed maps</a> to convert XML messages from one XML format to another. This connector can be used together with <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-flatfile">flat file decoder and encoder</a> to transform from flat files to XML and vice-versa; and with the EDIFACT <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-edifact-encode">encoder</a> and <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-edifact-decode">decoder</a> and X12 <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-x12-encode">encoder</a> and <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-x12-decode">decoder</a> to translate from <a href="https://en.wikipedia.org/wiki/Electronic_data_interchange">EDI</a> formats to XML and vice-versa. Even though, flat files, EDI and XML are quite common in legacy integrations, JSON is now the de-facto standard for data interchange.

While there are clear transformation tools for these legacy formats, I have heard more than a couple of times the question of how to transform JSON objects to different data models within a Logic App. In this blog post, I'll share some tips on how to leverage Logic Apps capabilities to implement JSON transformations in integration workflows.
<h2>Scenario</h2>
To show how to transform JSON objects, I'll be working with an imagined scenario. The Keep Yourself Active Company is organising a corporate Step Challenge across their multiple locations. In addition to the step competition, the company wants to collect data about workout distances and energy burned to show them in the dashboards. Due to the large scale of the competition, they chose to use more than one fitness tracking app and they need to support different units of measure, e.g. calories and kilojoules for energy burned, and kilometres and miles for distances. The integration team has started architecting the solution and have designed a <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html">Canonical Model</a> to represent each participant's data. They have decided to implement the <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/Normalizer.html">Normalisation</a> pattern to standardise the diverse formats coming from the different apps into the Canonical Model. In addition, the team has implemented a routing and tracking framework that requires an <a href="http://www.enterpriseintegrationpatterns.com/patterns/messaging/EnvelopeWrapper.html">Envelope Wrapper</a> to include some metadata. Sample input messages, including one from one particular fitness tracking app, and a canonical sample message are shown below.

Input 1: <strong>Employee Details</strong> coming from the HR System
<pre><code>{
   "firstName": "Paco",
   "lastName": "de la Cruz", 
   "location": "Melbourne",
   "country": "Australia",
   "department": "Information Technologies",
   "email": "<a href="mailto:paco@keepyourselfactive.com.au"><span style="color:#0563c1;text-decoration:underline;">paco@keepyourselfactive.com.au</span></a>"
}
</code></pre>
Input 2: <strong>Tracked Activities</strong> from the Fitness Tracking App 1
<pre><code>{
   "user": "<a href="mailto:paco@keepyourselfactive.com.au"><span style="color:#0563c1;text-decoration:underline;">paco@keepyourselfactive.com.au</span></a>",
   "workouts": [
      {
         "date": "2017-05-22",
         "type": "run",
         "distanceInMiles": 3.73,
         "time": "31:21",
         "energyInCalories": 533,
         "elevationInFeet": 119
      },
      {
         "date": "2017-05-24",
         "type": "run",
         "distanceInMiles": 3.74,
         "time": "32:05",
         "energyInCalories": 529,
         "elevationInFeet": 121
      },
      {
         "date": "2017-05-27",
         "type": "run",
         "distanceInMiles": 3.73,
         "time": "31:12",
         "energyInCalories": 534,
         "elevationInFeet": 118
      }
   ]
}
			</code></pre>
Input 3: <strong>Step Count</strong> from the Fitness Tracking App 1
<pre><code>{
   "user": "<a href="mailto:paco@keepyourselfactive.com.au"><span style="color:#0563c1;text-decoration:underline;">paco@keepyourselfactive.com.au</span></a>",
   "steps": [
      {
         "date": "2017-05-22",
         "steps": 11813
      },
      {
         "date": "2017-05-23",
         "steps": 8340
      },
      {
         "date": "2017-05-24",
         "steps": 10980
      },
      {
         "date": "2017-05-25",
         "steps": 9753
      },
      {
         "date": "2017-05-26",
         "steps": 8798
      },
      {
         "date": "2017-05-27",
         "steps": 12531
      },
      {
         "date": "2017-05-28",
         "steps": 7689
      }
   ]
}

			</code></pre>
Expected output in the <strong>Canonical Model</strong>, including an Envelope with metadata.
<pre><code>{
   "metadata": {
      "messageId": "6468f980-a167-4307-888e-874a843aebe4", 
      "timestamp": "2017-05-29T01:00:00:000Z", 
      "entityType": "ActiveChallengeParticipantWeekRecords",
      "version": "2017-04-01"
   }
   "payload": {
      "participant": {
        "givenName": "Paco",
        "familyName": "de la Cruz", 
        "office": "Melbourne",
        "country": "Australia",
        "department": "Information Technologies",
        "email": "<a href="mailto:paco@keepyourselfactive.com.au"><span style="color:#0563c1;text-decoration:underline;">paco@keepyourselfactive.com.au</span></a>"
      },
      "steps": [
         {
            "date": "2017-05-22",
            "steps": 11813
         },
         {
            "date": "2017-05-23",
            "steps": 8340
         },
         {
            "date": "2017-05-24",
            "steps": 10980
         },
         {
            "date": "2017-05-25",
            "steps": 9753
         },
         {
            "date": "2017-05-26",
            "steps": 8798
         },
         {
            "date": "2017-05-27",
            "steps": 12531
         },
         {
            "date": "2017-05-28",
            "steps": 7689
         }
      ],
      "workouts": [
         {
            "date": "2017-05-22",
            "type": "run",
            "distanceInKms": 6.00,
            "time": "31:21",
            "energyInKJ": 2230
         },
         {
            "date": "2017-05-24",
            "type": "run",
            "distanceInKms": 6.02,
            "time": "32:05",
            "energyInKJ": 2213
         },
         {
            "date": "2017-05-27",
            "type": "run",
            "distanceInKms": 6.00,
            "time": "31:12",
            "energyInKJ": 2234
         }
      ]
   }
}
</code></pre>
As we can see, this scenario includes several Enterprise Messaging Patterns, including Message Translator, Enrichment (assuming the information is coming from different APIs), Normalisation, Canonical Model and Envelope Wrapper. Let's have a look at how to implement this transformation in Logic Apps.
<h2>Preparing my scenario in a Logic App</h2>
For demonstration purposes, I'm implementing this scenario using a simple Logic App with <strong>Data Operations - Compose</strong> actions to create the input JSON messages, as shown below. However, in real-life scenarios, you would expect to receive these payloads from other APIs.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/05/051817_1155_Transformin1.png" alt="" />

Once we have the JSON messages in the Logic App, I'm using the <strong>Data Operations - Parse</strong> Action to be able to use Dynamic Content tokens from these JSON objects later. I've used the same payload as a sample message to generate the JSON schema.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/05/051817_1155_Transformin2.png" alt="" />
<h2>Mapping a flat JSON object</h2>
The easiest part of the mapping is to map a flat JSON object to another one. So let's start with the participant object within the payload. To transform one JSON object in a particular data model to a different one, we can leverage the <strong>Data Operations - Compose</strong> action to create an object with the required data model. We can use the dynamic content tokens from the previous <strong>Data Operations – Parse</strong> actions.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/05/051817_1155_Transformin3.png" alt="" />

And below is the code behind. There, you can see that we are constructing a JSON object with the required properties and we are using the properties of the parsed input payload to do so.
<pre><code>"Transform_Participant_by_Using_Compose": {
   "inputs": {
      "country": "@{body('Parse_Input_Employee_Details')?['country']}",
      "department": "@{body('Parse_Input_Employee_Details')?['department']}",
      "email": "@{body('Parse_Input_Employee_Details')?['email']}",
      "familyName": "@{body('Parse_Input_Employee_Details')?['lastName']}",
      "givenName": "@{body('Parse_Input_Employee_Details')?['firstName']}",
      "office": "@{body('Parse_Input_Employee_Details')?['location']}"
   },
   "runAfter": {
      "Parse_Input_Steps": [
         "Succeeded"
       ]
   },
   "type": "Compose"
}

			</code></pre>
That was too easy! This is how you map a JSON object with a flat structure in Logic Apps. Of course you can use any of the functions available in the <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language">Workflow Definition Language</a>
<h2>Mapping repeating records or an array in a JSON object</h2>
It's a common scenario that we have repeating records, or an array of objects in our messages and we need to translate them to a different data model.

To do so, we can use the <strong>For Each Loop </strong>available in Logic Apps, and use a <strong>Compose</strong> action inside. I'm showing how to transform the workouts array to the required data model using this approach. I'm utilising the <strong>@mul</strong> (multiply) function to calculate the distance in Kms and energy in KJ. As mentioned before, you can use any of the available functions in your mappings.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/05/051817_1155_Transformin4.png" alt="" />

The code behind is shown as follows.
<pre><code>"For_each_Workout": {
    "actions": {
        "Transform_Workouts_by_Using_Compose": {
            "inputs": {
                "date": "@{item()?['date']}",
                "distanceInKms": "@mul(item()?['distanceInMiles'], 1.60934)",
                "energyInKJ": "@mul(item()?['energyInCalories'], 4.184)",
                "time": "@{item()?['time']}",
                "type": "@{item()?['type']}"
            },
            "runAfter": {},
            "type": "Compose"
        }
    },
    "foreach": "@body('Parse_Input_Workouts')?['workouts']",
    "runAfter": {
        "Transform_Participant": [
            "Succeeded"
        ]
    },
    "type": "Foreach"
}
			</code></pre>
Another way to map an array of objects is by using the <strong>Data Operations – Select</strong> action. The main advantage of this approach is that we don't need to explicitly implement a For Each loop. Designer and code views of this action are shown below.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/05/051817_1155_Transformin5.png" alt="" /><strong>
</strong>
<pre><code>"Transform_Workouts_by_Using_Select": {
    "inputs": {
        "from": "@body('Parse_Input_Workouts')?['workouts']",
        "select": {
            "date": "@item()?['date']",
            "distanceInKms": "@mul(item()?['distanceInMiles'],1.60934)",
            "energyInKJ": "@mul(item()?['energyInCalories'],4.184)",
            "time": "@item()?['time']",
            "type": "@item()?['type']"
        }
    },
    "runAfter": {
        "For_each_Workout": [
            "Succeeded"
        ]
    },
    "type": "Select"
}
			</code></pre>
<h2>Including an array of objects in the Compose action</h2>
To create the JSON object in the Canonical Model, as detailed at the beginning of this post, we need to create the participant object and insert two arrays of objects (steps and workouts) while creating the JSON message. Again, this is possible by using the <strong>Data Operations – Compose</strong> action. I'm creating the Canonical Model, without the Envelope first for demonstration purposes. The Canonical Model includes, the transformed participant, the steps array and the translated workouts array. I'm composing again the participant, inserting the steps array as they are from the inputs, and inserting the workouts array as output from the Select action.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/05/051817_1155_Transformin6.png" alt="" />

This is the code behind, As you can see, we can insert arrays of objects while using the <strong>Data Operations - Compose</strong> action.
<pre><code>"Transform_Payload_to_Canonical_Model": {
   "inputs": {
        "participant": {
            "country": "@{body('Parse_Input_Employee_Details')?['country']}",
            "department": "@{body('Parse_Input_Employee_Details')?['department']}",
            "email": "@{body('Parse_Input_Employee_Details')?['email']}",
            "familyName": "@{body('Parse_Input_Employee_Details')?['lastName']}",
            "givenName": "@{body('Parse_Input_Employee_Details')?['firstName']}",
            "office": "@{body('Parse_Input_Employee_Details')?['location']}"
        },
        "steps": "@body('Parse_Input_Steps')?['steps']",
        "workouts": "@outputs(Transform_Workouts_by_Using_Select)"
    },
    "runAfter": {
        "For_each_Workout": [
            "Succeeded"
            ]
        },
    "type": "Compose"
}
</code></pre>
<h2>Adding an Envelope Wrapper to the JSON Payload</h2>
We can use a <strong>Compose</strong> Action also for adding the Envelope Wrapper as shown below. We just need to insert the payload within the composed JSON as follows.

<img src="https://www.mexia.com.au/wp-content/uploads/2017/05/051817_1155_Transformin7.png" alt="" />

And this is the code behind.
<pre><code>{
  "metadata": {
    "entityType": "ActiveChallengeParticipantWeekRecords",
    "messageId": "@{guid()}",
    "timestamp": "@{utcnow('o')}",
    "version": "2017-04-01"
  },
  "payload": "@outputs('Transform_Payload_to_Canonical_Model')"
}
			</code></pre>
<h2>Other mapping scenarios</h2>
There are other JSON mapping scenarios which can easily be solved using the same actions, such as:
<ul>
	<li><strong>Sending more than two payloads to an http triggered Azure function</strong>. You might have some scenarios in which you need to send more than one JSON payload to an http triggered Azure function. To do so, you can use a variation of the Wrapper pattern inserting the different payloads in a wrapping request body.</li>
	<li><strong>Transforming a payload which includes an array of objects or repeating records. </strong>In the scenario above, I showed how to include an array into a new message. However, you can apply the same principles to map a JSON object which includes an array of objects into a different data model. <strong>
</strong></li>
</ul>
<h2>More complex scenarios?</h2>
Chances are that you would have transformation requirements that go beyond the current capabilities in Logic Apps. For those cases, you have two main alternatives:
<ul>
	<li>Create an Azure Function to handles the transformation using code or</li>
	<li>Transform the JSON object into XML using the @xml() function; then implement the mapping using an <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-maps">XML</a> map; and finally transform the XML back to JSON using the @json() function. I'm not a big fan of this approach, as transforming JSON into XML and back might have an impact on your data model due to the restrictions of each standard.</li>
</ul>
<h2>Conclusion</h2>
In this post, I've shown how to use Logic Apps capabilities to transform JSON objects to a different data model. We can leverage Data Operation actions to do so. The <strong>Parse</strong> action allow us to use the different properties of the JSON object as dynamic content tokens in subsequent actions. The <strong>Compose</strong> action allows us to create a new JSON message, and it can be used within a For Each loop to work with Arrays. And the <strong>Select</strong> action is quite handy to map arrays of objects into a different model. Furthermore, for more complex scenarios you can always make use of Azure Functions or, if you are familiar with BizTalk Maps, XML Transforms.

How are you implementing your JSON object transformations in Logic Apps? Feel free to share your ideas or post your questions below.

HTH and happy clouding!
<p style="text-align:center;"><span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering Blog</span></a></p>
