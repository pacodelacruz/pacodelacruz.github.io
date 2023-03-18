---
layout: post
title: Assessing Azure API Management authorisation approaches
date: 2023-03-17 10:00
author: Paco de la Cruz
comments: true
category: Azure API Management
tags: [Azure API Management, OAuth 2.0, Security]
---

<h2><img src="/assets/img/passport-01.jpeg" alt="passport-01" width="1200" height="600" loading="lazy" style="height: auto; max-width: 100%; width: 1200px;"></h2>
<h2>Introduction</h2>
<p>As part of our consulting engagements, it is common that our customers ask us to provide guidance on the different authentication and authorisation approaches available on API Management and how they meet their security needs while offering support to existing legacy API clients. While many people now know that <a href="https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/auth-oauth2">OAuth 2.0</a> is the industry protocol for HTTP authorisation, there are cases where some API consumers don’t provide support for it. Thus, organisations need to know when alternative controls can be implemented.</p>
<!--more-->
<p>In this post, I will share our starting position for this type of assessment. It’s worth noting that each organisation has different needs and there is always the need to tailor this assessment accordingly.</p>
<h2>Context</h2>
<p>Before assessing each of the different authorisation approaches available on API Management, it’s important to understand the <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/authentication-vs-authorization">difference between authentication and authorisation</a>. We also need to understand who is meant to be calling the API and the type of data being exposed. Let’s explore these topics in more detail.</p>
<h3>API client authentication use cases</h3>
<p>To classify an API consumer, first, we can consider different use cases. Do we need to authorise an application or an end user? Shall we allow an anonymous client to call the API? Three main use cases are described in the table below.</p>
<table style="border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: 1px solid #99acc2;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Use case</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Description</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Examples</span></strong></p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Anonymous</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>No authentication is required to call an API.&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Any unauthenticated client</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>End-user authentication&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Permissions are granted at user level.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>A user accessing their own data via an API</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Application authentication&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>An application needs to authenticate to gain access to an API.&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>A web application needs to authenticate against a database to get access to data.</p>
<p>A service running in the background needs to authenticate to call an API.</p>
</td>
</tr>
</tbody>
</table>
<h3>API client trust level</h3>
<p>Depending on who owns and manages the API client and where it is deployed, different security controls will need to be in place. We can use the model below to classify API clients based on their trust level.</p>
<table style="border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: 1px solid #99acc2;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p><strong><span style="color: black;">Trust level</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p><strong><span style="color: black;">Description</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Examples</span></strong></p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p>First-party internal</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p>A trusted application that belongs to the organisation and is deployed within a trusted private network, either on-premises or on the cloud.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>A service running in the background and deployed on-premises</p>
<p>A serverless function deployed in the organisation’s cloud private network.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p>First-party external&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p>A trusted system that is owned and controlled by the organization but deployed outside the private network. &nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>A cloud-hosted SaaS application owned by the organisation, e.g., a SaaS CRM or ERP.</p>
<p>A corporate mobile app developed internally.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p>Third-party&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p>Any system that is not trusted.&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>A strategic customer’s application</p>
<p>A vendor’s system</p>
<p>An application configured by an end user.</p>
</td>
</tr>
</tbody>
</table>
<h3>API client types</h3>
<p>It is important that we also consider if the API client is a mobile application, a client-side web application, or a server-side application. This is relevant to understand how the API credentials are going to be managed and stored.</p>
<table style="border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: 1px solid #99acc2;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p><strong><span style="color: black;">Trust level</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p><strong><span style="color: black;">Description</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Examples</span></strong></p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p>Client-side web application&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p>A single-page web application that directly calls the APIs.&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>A javascript application that runs in the user's browser</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p>Native mobile application&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p>A native mobile application that calls the APIs.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>An Android app</p>
<p>An iOS app</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;" width="141">
<p>Server-side application&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;" width="259">
<p>A server application that calls APIs in another system.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>A cloud based serverless application that calls APIs hosted on another service</p>
</td>
</tr>
</tbody>
</table>
<h3>Data confidentiality and integrity</h3>
<p>When protecting an API, it’s crucial to understand the type of information that is being exposed by the API and the type of operations the API consumer can perform against the data.</p>
<p>When implementing authorisation on an API, we are dealing with two key data security pillars.</p>
<ul>
<li><strong>Confidentiality</strong> ensures that only allowed consumers can access the exposed data. We should have a <a href="https://learn.microsoft.com/en-us/compliance/assurance/assurance-data-classification-and-labels#what-is-a-data-classification-framework">data-classification framework</a> in place to understand the level of sensitivity of the information being exposed and an access control model that dictates who has access to what.</li>
<li><strong>Integrity</strong> ensures that the data exposed by the API cannot be altered by clients that are not authorised to do so. We need to understand if different API consumers have different access levels to the information being exposed. For instance, while some users have read-only access, others can modify their own records, and some privileged users are able to perform read and write operations on all records.</li>
</ul>
<p>A secure API must be able to guarantee the confidentiality and integrity of the information it exposes. Understanding the confidentiality level of the data and the data operations being exposed by the API would help us to better define the authorisation approach required for the API.</p>
<p>We have explored the different authentication use cases, client trust levels and types, and discussed the importance of understanding the data classification and the type of operations API clients can perform against the exposed data. Let’s now explore the different authentication and authorisation approaches available on API Management.</p>
<h2>Available Approaches</h2>
<p>Let’s start by describing the authentication and authorisation approaches available on API Management. In the table below I describe different authentication and authorisation approaches and give a summary of how they can be implemented on API Management.</p>
<table style="border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: 1px solid #99acc2;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Approach</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Description</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Authentication</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Authorisation</span></strong></p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Anonymous</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Anonymous access allows any user or application to access an API without requiring authentication</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>N/A</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>N/A</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Client IP address</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>The API client authenticates against the API using its own IP address</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Source IP address in the IP packets or the <strong>X-Forwarded-For</strong> header for requests going through a proxy.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><a href="https://learn.microsoft.com/en-us/azure/api-management/ip-filter-policy">ip-filter</a>&nbsp;policy for direct calls.</p>
<p>Using the&nbsp;<a href="https://learn.microsoft.com/en-us/azure/api-management/check-header-policy">check-header</a>&nbsp;policy for calls going through a proxy to validate the <strong>X-Forwarded-For</strong> header</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>API key</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>The client application authenticates by adding a key to each API request.</p>
<p>API keys can be sent as query parameters or HTTP headers.</p>
<p>API keys are relatively easy for legacy API clients to implement.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>API key in the query parameters or HTTP headers</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Requires a <a href="https://learn.microsoft.com/en-us/azure/api-management/api-management-subscriptions">subscription</a> at product or API level and the corresponding subscription&nbsp;<a href="https://learn.microsoft.com/en-us/azure/api-management/api-management-subscriptions#scope-of-subscriptions">scope assignment</a>.&nbsp;</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>HTTP basic authentication</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>The basic&nbsp;HTTP&nbsp;authentication scheme transmits credentials as user-id/password pairs, encoded in Base64.</p>
<p>The standard is documented in the&nbsp;Internet Engineering Task Force's&nbsp;<a href="https://datatracker.ietf.org/doc/html/rfc7617">RFC7617</a>.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>User-id/password pair encoded in Base64 via the <strong>Authorization</strong> header as described in the&nbsp;<a href="https://datatracker.ietf.org/doc/html/rfc7617#section-2">official documentation</a>.&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Using the&nbsp;<a href="https://learn.microsoft.com/en-us/azure/api-management/check-header-policy">check-header</a>&nbsp;policy and checking that the “Authorization” header contains “Basic” + the base64 encoded user-id/password pair.</p>
<p>&nbsp;</p>
<p>When dealing with secrets on API Management, these must be <a href="https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-properties?tabs=azure-portal#key-vault-secrets">stored as secret name values in Key Vault</a>.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Client certificate</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Client certificate authentication is an authentication approach, where the client provides its client certificate to the server to prove its identity as a part of the TLS handshake.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>The client provides a client certificate message excluding the private key. The client is authenticated by using its private key to sign a hash of all the messages. The recipient verifies the signature using the public key of the signer, thus ensuring it was signed with the client’s private key. Refer to&nbsp;<a href="https://datatracker.ietf.org/doc/html/rfc5246#section-7.4.6">RFC 5246</a>&nbsp;for more details.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Using the&nbsp;<a href="https://learn.microsoft.com/en-us/azure/api-management/validate-client-certificate-policy">validate-client-certificate</a>&nbsp;policy or&nbsp;<a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-mutual-certificates-for-clients#certificate-validation-with-context-variables">validating context variables</a>.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>OAuth 2.0 client credentials flow</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>The <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow">OAuth 2.0 client credentials grant flow</a> is part of the <a href="https://en.wikipedia.org/wiki/OAuth">open authorisation standard</a> and permits an application to use its own credentials – instead of impersonating a user – to authenticate when calling an API. It is commonly&nbsp;used for server-to-server interactions that must run in the background.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>The client application authenticates against an identity provider to get an access token that is passed to the API for authorisation following a standard&nbsp;<a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow#protocol-diagram">flow</a>.&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Using the <a href="https://learn.microsoft.com/en-au/azure/api-management/validate-azure-ad-token-policy">validate-azure-ad-token</a> policy for validating JWTs provided by Azure AD and <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT">validate-jwt</a>&nbsp;policy for third-party identity providers.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>OAuth 2.0 authorization code flow</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>The OAuth 2.0 authorization code flow is described in&nbsp;<a href="https://tools.ietf.org/html/rfc6749">section 4.1 of the OAuth 2.0 specification</a>. It's used to perform authentication and authorization in the majority of application types, including&nbsp;<a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-app-types#single-page-apps-javascript">single-page applications</a>,&nbsp;<a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-app-types#web-apps">server-side web applications</a>, <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-app-types#mobile-and-native-apps">native mobile apps</a>, and <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-app-types#daemons-and-server-side-apps">daemons and server-side apps</a>.</p>
<p>The flow enables applications to securely acquire end-user access tokens that can be used to access secured resources.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>An end user authenticates against an identity provider to get a token that is passed to the API for authorisation following a standard <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow#protocol-diagram">flow</a>.&nbsp;&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Using the <a href="https://learn.microsoft.com/en-au/azure/api-management/validate-azure-ad-token-policy">validate-azure-ad-token</a> policy for validating <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/id-tokens#claims-in-an-id-token">JWTs or ID tokens</a> provided by Azure AD and <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT">validate-jwt</a>&nbsp;policy for third-party identity providers.</p>
<p>&nbsp;</p>
</td>
</tr>
</tbody>
</table>
<h2>Assessment</h2>
<p>Once we understand the different authorisation approaches available on API Management, let’s assess them against different scenarios and define when these approaches are recommended or if not, when they can be leveraged as tolerated approaches under certain circumstances. In the table below, I describe when the scenarios for which these approaches are not applicable, when they are recommended, the conditions under which they can be tolerated and additional considerations for each. As mentioned previously, this assessment should be used just as a starting position but must be tailored to each organisation’s needs.</p>
<table style="border-collapse: collapse; table-layout: fixed; margin-left: auto; margin-right: auto; border: 1px solid #99acc2;">
<tbody>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Approach</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Not applicable scenarios</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Recommended scenarios</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Tolerated scenarios</span></strong></p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><strong><span style="color: black;">Additional notes</span></strong></p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Anonymous</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>N/A</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Not recommended</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>For 1st party internal clients and read-only APIs that do not expose sensitive or confidential information.</p>
<p>When required for external clients, for read-only and minimal risk APIs that only expose information classified as public.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>When exposing anonymous APIs, additional security controls must be in place, such as a firewall for thread detection &amp; protection, DDoS protection, a web application firewall, rate-limiting policies, etc.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Client IP address</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>End-user authentication</p>
<p>Client-side web application</p>
<p>Native mobile applications</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Not recommended</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>1st party internal clients using a private IP address when recommended authorisation approaches are not available to the API consumer.</p>
<p>As an additional security control on top of endorsed authorisation approaches or security measures.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p><a href="https://en.wikipedia.org/wiki/IP_address_spoofing">IP spoofing</a> can be easily leveraged to overcome security controls.</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>API key</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>For client-side web applications or native mobile apps, as API keys can be easily leaked.&nbsp;</p>
<p>While end users can get API keys, keys are typically used for application authentication scenarios.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Not recommended</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>For APIs that do not expose sensitive or confidential information and a recommended authorisation method is not supported by the clients.</p>
<p>Not as an authorisation approach, but for rate limiting or usage statistics.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>When used for authentication purposes, always use HTTP headers to pass an API key. Secrets must never be passed in URL query strings as there are <a href="https://owasp.org/www-community/vulnerabilities/Information_exposure_through_query_strings_in_url">many risk factors</a>.</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>HTTP basic authentication</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>For client-side web applications or native mobile apps, as secrets can be easily leaked.&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Not recommended</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>For APIs that do not expose sensitive or confidential information and a recommended authorisation method is not supported by API clients.</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>When using this approach, avoid storing plain text credentials in an API Management policy. Instead, use <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-properties?tabs=azure-portal#key-vault-secrets">secret named values stored on Key Vault</a>.&nbsp;</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>Client certificate</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>End-user authentication</p>
<p>Client-side web application</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>For application authentication scenarios, where certificates are preferred over client ids and client secrets.</p>
<p>&nbsp;</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>N/A</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>When client certificates are preferred over client secrets, consider leveraging the <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow#second-case-access-token-request-with-a-certificate">OAuth 2.0 client credentials flow with certificate-based request</a>.</p>
<p>While certificate authentication can be used for end-users, it is not recommended as authentication policies on API Management expect to authorise a single certificate. The same restriction applies to client-side web applications.</p>
<p>Consider the operational overhead of managing client certificates.&nbsp;</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>OAuth 2.0 client credentials flow</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>End-user authentication</p>
<p>Client-side web application</p>
<p>Native mobile application</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Recommended for server-side application authorisation</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>N/A</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>When validating JWTs, it is important to validate not only the audience but role claims as well to ensure the application has the right level of access as documented&nbsp;<a href="https://platform.deloitte.com.au/articles/oauth2-client-credentials-flow-on-azure-api-management">here</a>.&nbsp;</p>
</td>
</tr>
<tr>
<td style="border: solid windowtext 1.0pt;">
<p>OAuth 2.0 authorization code flow</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Application authentication</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>Recommended for end-user authorisation</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>N/A</p>
</td>
<td style="border: solid windowtext 1.0pt;">
<p>When validating JWTs, it is important to validate not only the audience but role claims as well to ensure the end user has the right level of access.</p>
</td>
</tr>
</tbody>
</table>
<h2>An additional note on OAuth 2.0 client credentials.</h2>
<p>When discussing the different authorisation approaches with customers, a couple of times I’ve faced some skepticism about the OAuth 2.0 client credentials flow. Their main arguments have typically been:</p>
<ul>
<li>MFA cannot be enabled for applications. Once the client ID and client secret are leaked, then it’s practically the same level of security as basic authentication.</li>
<li>TLS protects the basic authentication credentials on every request.</li>
<li>JWT access tokens cannot be revoked on Azure AD.</li>
<li>Service principals in the Azure Active Directory tenant can be over-permissioned</li>
</ul>
<p>So, what are really the benefits of the OAuth 2.0 client credentials flow over basic authentication? Below are the arguments I present.</p>
<ul>
<li>Even though TLS is protecting every single request, it’s always better to ensure that credentials do not travel very frequently. The less frequently secrets travel over the wire, the lower the chances of getting them compromised by a man-in-the-middle (MITM) attack.</li>
<li>If an access token gets compromised, the access token will expire. On the contrary, when credentials get compromised by a MITM attack, they could be unnoticedly used for a longer period.</li>
<li>Delegating the authentication to a trusted and robust identity provider brings many benefits.
<ul>
<li>Centralised and built-in sign-in audit logs and threat detection and protection features.</li>
<li>Centralised secret strength and expiry policies.</li>
<li>Secret rotation policies and capabilities.</li>
<li>Secrets don’t have to be managed and asserted on the API layer.</li>
<li>Besides validating the token’s audience and the role claims (authorisation), the API is fully abstracted from the authentication process.</li>
</ul>
</li>
<li>Azure AD and other identity providers support not only a shared secret but also <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow#get-a-token">certificates and federated credentials</a> for the client credentials flow.</li>
<li>Azure AD <a href="https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token#overview">supports managed identities</a> for the OAuth 2.0 client credentials flow. When the API consumers are an Azure resource within the same tenant, <a href="https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview#managed-identity-types">managed identities</a> remove the need to manage secrets.</li>
<li>The scoping features of the JWTs. Having well-defined audience and roles claims, allows for fine-grained privilege bracketing. When authorising an API client, we can validate not only the target audience in the JWT, but the roles the client has against to on the API, e.g., read, write, etc.</li>
<li>It’s also worth recognising that currently on Azure AD, JWT access tokens <a href="https://learn.microsoft.com/en-us/answers/questions/672062/how-to-revoke-token">cannot be revoked</a>. And while <a href="https://learn.microsoft.com/en-us/azure/active-directory/enterprise-users/users-revoke-access#azure-active-directory-environment">refresh tokens (long-lived) can be revoked</a>, the OAuth 2.0 client credentials flow <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow">only relies on access tokens</a>. Thus, considering that we can’t revoke access tokens, we must <a href="https://learn.microsoft.com/en-us/azure/active-directory/develop/access-tokens#access-token-lifetime">configure the access token lifetime</a> according to the organisation’s needs. Furthermore, if an access token gets compromised by a MITM attack, it’s highly likely that it will go unnoticed or if we become aware of it, by the time we can action a revocation procedure, the access token is most likely expired.</li>
<li>To avoid over-permissioned service principals, we need to follow the least privilege principle and apply the right audience and role claims when granting access to the service principal.</li>
</ul>
<h2>Conclusions</h2>
<p>In this post, I discussed and assessed the different authentication and authorisation approaches available in API Management based on general principles. If you’re planning to endorse authorisation approaches for your own project or organisation, please consider this as a baseline. Always make your own assessment based on the organisation’s unique situation and needs.</p>
<p>Furthermore, API authorisation is just one of the many security controls you must consider when protecting your APIs on API Management. I’d recommend you to be familiar with the <a href="https://learn.microsoft.com/en-us/security/benchmark/azure/baselines/api-management-security-baseline#logging-and-threat-detection">API Management security baseline</a> and make sure you have the right security controls in place. Other key security controls to consider are:</p>
<ul>
<li>Implement <strong>network protection:</strong>
<ul>
<li>Segregate <a href="https://learn.microsoft.com/en-us/azure/architecture/example-scenario/apps/publish-internal-apis-externally">public-facing and internal-facing APIs</a>.</li>
<li>Protect against common web exploits and vulnerabilities by deploying a <a href="https://learn.microsoft.com/en-us/azure/web-application-firewall/ag/ag-overview">Web Application Firewall</a> (WAF) <a href="https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway">in front of API Management</a><span>.</span></li>
<li>Implement <a href="https://learn.microsoft.com/en-us/azure/api-management/protect-with-ddos-protection">DDoS protection</a><span>.</span></li>
<li>For higher security, consider <a href="https://learn.microsoft.com/en-us/azure/firewall/threat-intel">thread detection and protection</a> via a firewall.</li>
</ul>
</li>
<li>Always ensure data encryption in transit.</li>
<li>Implement rate-limiting or <a href="https://learn.microsoft.com/en-us/azure/api-management/api-management-sample-flexible-throttling">throttling</a></li>
<li>Define and implement <a href="https://learn.microsoft.com/en-us/azure/api-management/api-management-role-based-access-control">role-based access control (RBAC)</a> that includes privilege access management.</li>
<li>Ensure <a href="https://learn.microsoft.com/en-us/azure/architecture/example-scenario/security/apps-zero-trust-identity">zero trust end-to-end protection</a> for APIs and backend services.</li>
</ul>
<p>I hope you’ve found this post useful and that it has provided some guidance to make your APIs more secure.</p>
<p>Happy clouding!</p>
<p>
<p style="text-align:center;"><span style="font-style:italic;">Cross-posted on </span><a href="https://platform.deloitte.com.au/articles/author/paco-de-la-cruz"><span style="font-style:italic;">Deloitte Platform Engineering</span></a><br/>
<span style="font-style:italic;">Follow me on </span><a href="https://twitter.com/pacodelacruz"><span style="font-style:italic;">@pacodelacruz</span></a></p>