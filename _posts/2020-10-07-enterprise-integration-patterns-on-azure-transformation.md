---
layout: post
title: Messaging Transformation - Enterprise Integration Patterns on Azure
date: 2020-10-07 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Enterprise Integration Patterns, Azure iPaaS, Logic Apps, Service Bus, Event Grid, Azure Functions]
---

<p><img src="/assets/img/2019/04/Transform8.jpg" alt="Transform8" width="4000" style="width: 4000px;"></p>
<p>It is quite common that integration solutions interact with diverse applications, each with disparate data models and heterogeneous formats. In this post, we will discuss the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageTransformationIntro.html" rel="noopener" target="_blank">Message Transformation</a> patterns and how these can be implemented on Azure to overcome the challenges derived from working with different message structures and representations.</p>
<!--more-->
<p>This post is a part of a series describing how to implement the <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/index.html" rel="noopener" target="_blank"><span>Enterprise Integration Patterns</span></a> using the <a href="/microsoft-azure-ipaas" rel="noopener" target="_blank"><span>Azure Integration Services</span></a>:</p>
<ol>
<li><a href="/enterprise-integration-patterns-on-azure-intro" rel="noopener" target="_blank">Introduction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-message-construction" rel="noopener" target="_blank">Message Construction</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank">Messaging Channels</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-endpoints" rel="noopener" target="_blank">Messaging Endpoints</a></li>
<li><a href="/enterprise-integration-patterns-on-azure-routing" rel="noopener" target="_blank">Message Routing</a></li>
<li>Message Transformation (this)</li>
<li><a href="/enterprise-integration-patterns-on-azure-platform" rel="noopener" target="_blank">Platform Management</a></li>
</ol>
<p>The patterns covered in this article are listed below.</p>
<ul>
<li><a href="#message-translator">Message Translator</a></li>
<li><a href="#envelope-wrapper">Envelope Wrapper</a></li>
<li><a href="#content-enricher">Content Enricher</a></li>
<li><a href="#content-filter">Content Filter</a></li>
<li><a href="#claim-check">Claim-Check</a></li>
<li><a href="#normaliser">Normaliser</a></li>
<li><a href="#canonical-data-model">Canonical Data Model</a></li>
<li><a href="#compression-decompression">Compression/Decompression (*)</a></li>
<li><a href="#encryption-decryption">Encryption/Decryption (*)</a></li>
</ul>
<h1 id="message-translator">Message Translator</h1>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageTranslator.html" rel="noopener" target="_blank"><strong>Message Translator</strong></a> pattern describes how data can be translated from one format to another. Transformation can happen at different levels:</p>
<ul>
<li><strong>Data structure</strong>: related to entity hierarchies, relationships, cardinality, etc.</li>
<li><strong>Data types</strong>: related to cross-references, domains, constraints, data types, field names, etc.</li>
<li><strong>Data representation</strong>: deals with the format, e.g. binary, JSON, XML, CSVs, key-value pairs; industry standards like EDIFACT, X12; vendor-specific formats, character sets, encryption, compression, digital signatures, etc.</li>
<li><strong>Transport</strong>: deals with communication protocols such as HTTP, JMS, AMQP, gRPC, SOAP, etc.</li>
</ul>
<p><strong>Implementation</strong></p>
<table style="height: 550px;">
<tbody>
<tr style="height: 495px;">
<td style="width: 96px; height: 495px;">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="width: 538px; height: 495px;">
<p>There are different ways to transform messages in Logic Apps</p>
<p>For data structure and data type transformations:</p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="/transforming-json-objects-in-logic-apps" rel="noopener" target="_blank">Simple JSON object transformation with built-in actions </a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform" rel="noopener" target="_blank">Advanced JSON object transformations with Liquid templates</a></p>
<p><span><span style="text-decoration: none;">·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span></span><a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-transform" rel="noopener" target="_blank">Transform XML to XML</a></p>
<p>For data representation transformations:</p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference#json" rel="noopener" target="_blank">XML to JSON</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference#xml" rel="noopener" target="_blank">JSON to XML</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-perform-data-operations#create-csv-table-action" rel="noopener" target="_blank">JSON array to CSV</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-perform-data-operations#create-html-table-action" rel="noopener" target="_blank">JSON array to HTML</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference#conversion-functions" rel="noopener" target="_blank">Conversion functions</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-flatfile" rel="noopener" target="_blank">Flat file encoding and decoding</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-x12-encode" rel="noopener" target="_blank">EDI X12 encoding and decoding</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-edifact-encode" rel="noopener" target="_blank">EDIFACT encoding and decoding</a></p>
<p>·&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-rosettanet#send-or-encode-rosettanet-messages" rel="noopener" target="_blank">RosettaNet encoding and decoding</a></p>
</td>
</tr>
<tr style="height: 55px;">
<td style="width: 96px; height: 55px;">
<p><strong> <img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="width: 538px; height: 55px;">
<p>On Azure Functions, custom transformations can be done via custom code.</p>
</td>
</tr>
</tbody>
</table>
<h1 id="envelope-wrapper">Envelope Wrapper</h1>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/EnvelopeWrapper.html" rel="noopener" target="_blank"><strong>Envelope Wrapper</strong></a> pattern outlines how integration solutions can support adding a wrapper to messages so they can contain metadata required by the integration process while being able to deliver messages to the intended receivers in the format they need.</p>
<p><strong>Implementation</strong></p>
<table style="height: 142px;">
<tbody>
<tr style="height: 91px;">
<td style="height: 91px; width: 96px;"><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></td>
<td style="height: 91px; width: 538px;">
<p>On Logic Apps, an <strong>Envelope Wrapper</strong> can easily be added and removed using the <em>Compose</em> built-in action as <a href="/transforming-json-objects-in-logic-apps" rel="noopener" target="_blank">shown here</a> or leveraging <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform" rel="noopener" target="_blank">Liquid templates</a> for more complex scenarios.</p>
</td>
</tr>
<tr style="height: 51px;">
<td style="height: 51px; width: 96px;">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 51px; width: 538px;">
<p>On Azure Functions, <strong>Envelope Wrappers</strong> can be added and removed via custom code.</p>
</td>
</tr>
</tbody>
</table>
<h1 id="content-enricher">Content Enricher</h1>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/DataEnricher.html" rel="noopener" target="_blank"><strong>Content Enricher</strong></a> pattern is useful when the message coming from a source system does not contain all the data required to be processed in the target system. This filter can access an additional data source to augment the message with the missing information.</p>
<p><strong>Implementation</strong></p>
<table style="height: 160px;">
<tbody>
<tr style="height: 69px;">
<td style="height: 69px; width: 96px;"><strong><img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></td>
<td style="height: 69px; width: 538px;">
<p>Logic Apps connectors can be used to retrieve the missing information from the enrichment source(s).</p>
</td>
</tr>
<tr style="height: 91px;">
<td style="height: 91px; width: 96px;">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 91px; width: 538px;">
<p>Azure activity functions can be used to retrieve the missing information from the enrichment source(s) in an integration process orchestrated by Durable Functions.</p>
</td>
</tr>
</tbody>
</table>
<h1 id="content-filter">Content Filter</h1>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/ContentFilter.html" rel="noopener" target="_blank"><strong>Content Filter</strong></a> pattern describes how to remove data elements or hierarchies that are not relevant to the target system. This pattern can also be used to remove sensitive information before a message is delivered to other systems.</p>
<p><strong>Implementation</strong></p>
<table style="height: 114px;">
<tbody>
<tr style="height: 81px;">
<td style="height: 81px; width: 96px;">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 81px; width: 538px;">
<p>See the <strong><a href="#message-translator">Message Translator</a></strong>&nbsp;implementation section above related to data structure and data type transformations.</p>
</td>
</tr>
<tr style="height: 33px;">
<td style="height: 33px; width: 96px;">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 33px; width: 538px;">
<p>On Azure Functions, custom transformations can be done via custom code.</p>
</td>
</tr>
</tbody>
</table>
<h1 id="claim-check">Claim-Check</h1>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html"><strong>Claim-Check</strong></a> pattern is helpful when the message size must be reduced without losing information. This pattern suggests that the full message can be stored in a durable store and a claim can be passed to subsequence components. Those components would use the claim to retrieve the message persisted.</p>
<p>When the main limitation is the <a href="/enterprise-integration-patterns-on-azure-messaging-channels" rel="noopener" target="_blank"><strong>Message Channel</strong></a>’s message size limitation, consider using the <span style="background-color: transparent;"></span><a href="#compression-decompression" style="background-color: transparent;"><strong>Compression/Decompression (*)</strong>&nbsp;</a>pattern when applicable to avoid managing another component for storage and the lifecycle of the persisted messages.</p>
<p><strong>Implementation</strong></p>
<table style="height: 97px;">
<tbody>
<tr style="height: 30px;">
<td style="height: 30px; width: 96px;">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 30px; width: 538px;">
<p>This pattern can be implemented on Logic Apps as described <a href="https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern" rel="noopener" target="_blank">here</a>.</p>
</td>
</tr>
<tr style="height: 67px;">
<td style="height: 67px; width: 96px;">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 67px; width: 538px;">
<p>This pattern can be implemented on Azure Functions using custom code.</p>
</td>
</tr>
</tbody>
</table>
<h1 id="normaliser">Normaliser</h1>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/Normalizer.html" rel="noopener" target="_blank"><strong>Normaliser</strong></a> is a special type of Message Translator that translates semantically equivalent messages but with different structures to a common format. The Normaliser is a combination of a <a href="/enterprise-integration-patterns-on-azure-routing" rel="noopener" target="_blank"><strong>Message Router</strong></a> and a set of <strong>Message Translators</strong>.</p>
<p><strong>Implementation</strong></p>
<p><em>The implementation of each of the part of this composed pattern has been described previously. </em></p>
<h1 id="canonical-data-model">Canonical Data Model</h1>
<p>The <a href="https://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html" rel="noopener" target="_blank"><strong>Canonical Data Model</strong></a> is beneficial when messages semantically equivalent are to be interchanged between multiple applications, each with disparate formats or structures. The value of a <strong>Canonical Data Model</strong> is realised when the number of <strong><span style="background-color: transparent;"></span><a href="#message-translator" style="background-color: transparent;">Message Translators</a></strong><span style="background-color: transparent;">&nbsp;can be reduced by having a common model or when decoupling from proprietary formats is important.</span></p>
<p><strong>Implementation</strong></p>
<table style="height: 62px;">
<tbody>
<tr style="height: 52px;">
<td style="height: 52px; width: 96px;">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 52px; width: 538px;">
<p>See the <strong><span style="background-color: transparent;"></span><a href="#message-translator" style="background-color: transparent;">Message Translator</a></strong>&nbsp;implementation section above related to data structure and data type transformations.</p>
</td>
</tr>
<tr style="height: 10px;">
<td style="height: 10px; width: 96px;">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 10px; width: 538px;">
<p>On Azure Functions, custom transformations can be done via custom code.</p>
</td>
</tr>
</tbody>
</table>
<h1 id="compression-decompression">Compression/Decompression (*)</h1>
<p>The <strong>Compression/Decompression (*)</strong> pattern is a type of data representation transformation. While this is not mentioned as a pattern in the Enterprise Integration Patterns book, I believe it is important highlighting it, as it is a quite common requirement. When dealing with large messages, I prefer to use compression instead of the <strong><a href="#claim-check">Claim-Check</a></strong>&nbsp;pattern when possible because it is typically much simpler to implement.</p>
<p><strong>Implementation</strong></p>
<table style="height: 54px;">
<tbody>
<tr style="height: 54px;">
<td style="height: 54px; width: 96px;">
<p><strong> <img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 54px; width: 538px;">
<p>Compression and decompression can be implemented using custom code on Azure Functions.</p>
</td>
</tr>
</tbody>
</table>
<h1 id="encryption-decryption">Encryption/Decryption (*)</h1>
<p>The <strong>Encryption/Decryption (*)</strong> pattern is another kind of data representation transformation. Although it is not mentioned as a pattern in the Enterprise Integration Patterns book, it is a quite common requirement.</p>
<p>Encryption is usually required for data in transit and data at rest. For encryption at rest, encryption can be either client-side or server-side. Client-side encryption means that encryption is performed in memory by an application before the data is being persisted. Server-side encryption at rest is when the storage service performs encryption/decryption transparently when storing or retrieving the data.</p>
<p><strong>Implementation</strong></p>
<table style="height: 328px;">
<tbody>
<tr style="height: 79px;">
<td style="height: 79px; width: 96px;"><strong><img src="/assets/img/2019/04/Azure%20Service%20Bus_COLOR.png" alt="Azure Service Bus_COLOR" width="80" style="width: 80px;"></strong></td>
<td style="height: 79px; width: 538px;">
<p>Service Bus provides inbuilt <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-security-controls#data-protection" rel="noopener" target="_blank">server-side encryption at rest</a>.</p>
<p>All requests to Service Bus are protected using encryption in transit.</p>
</td>
</tr>
<tr style="height: 79px;">
<td style="height: 79px; width: 96px;"><strong><img src="/assets/img/2019/04/Event%20Grid.png" alt="Event Grid" width="81" style="width: 81px;"></strong></td>
<td style="height: 79px; width: 538px;">
<p>Event Grid provides inbuilt <a href="https://docs.microsoft.com/en-us/azure/event-grid/security-authorization#encryption-at-rest" rel="noopener" target="_blank">server-side encryption at rest</a>.</p>
<p>All requests to Event Grid are protected using encryption in transit.</p>
</td>
</tr>
<tr style="height: 105px;">
<td style="height: 105px; width: 96px;">
<p><strong> <img src="/assets/img/2019/04/Logic%20Apps_COLOR.png" alt="Logic Apps_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 105px; width: 538px;">
<p>For scenarios when client-side encryption is required, Logic Apps supports encrypting data with the “<a href="https://docs.microsoft.com/en-us/connectors/keyvault/#encrypt-data-with-key" rel="noopener" target="_blank">Encrypt data with key</a>” action of the Key Vault connector.</p>
</td>
</tr>
<tr style="height: 65px;">
<td style="height: 65px; width: 96px;">
<p><strong><img src="/assets/img/2019/04/Azure%20Functions_COLOR.png" alt="Azure Functions_COLOR" width="80" style="width: 80px;"></strong></p>
</td>
<td style="height: 65px; width: 538px;">
<p>For scenarios when client-side encryption is required, Azure Functions can encrypt messages using custom code and Key Vault <a href="https://docs.microsoft.com/en-us/rest/api/keyvault/encrypt/encrypt" rel="noopener" target="_blank">encryption</a>.</p>
</td>
</tr>
</tbody>
</table>
<h1>Wrapping Up</h1>
<p>In this post, we have covered the <strong>Messaging Transformation</strong> patterns and how to leverage the Azure Integration Services to implement them. Understanding these patterns allows us to consider different approaches when architecting integration solutions that require dealing with messages with disparate structures and different formats. I hope you have found this post useful. Stay tuned to the next and last instalment of this series covering the <strong>Platform Management</strong> patterns.</p>
<p>Happy integration!</p>

<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>