---
layout: post
title: Implementing a WCF Client with Certificate-Based Mutual Authentication without using Windows Certificate Store
date: 2015-11-23 20:23
author: Paco de la Cruz
comments: true
categories: [Azure WebJobs, Development, mTLS, WCF, WCF Client]
---

<p style="text-align:center;"><em>Cross-posted on <a href="https://blog.kloud.com.au/author/pacodelacruzag/">Kloud's blog</a>.
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>

Windows Communication Foundation (WCF) provides a relatively simple way to implement Certificate-Based Mutual Authentication on distributed clients and services. Additionally, it supports interoperability as it is based on WS-Security and X.509 certificate standards. This [blog post](http://blogs.msdn.com/b/bradleycotier/archive/2011/12/14/mutual-authentication-with-a-iis-hosted-wcf-data-service-installed-in-a-workgroup-environment.aspx) briefly summarises mutual authentication and covers the steps to implement it with an IIS hosted WCF service.

Even though WCF's out-of-the-box functionality removes much of the complexity of Certificate-Based Mutual Authentication in many scenarios, there are cases in which this is not what we need. For example, by default, WCF relies on the Windows Certificate Store for accessing the own private key and the counterpart's public key when implementing Certificate-Based Mutual Authentication.

Having said so, there are scenarios in which using the Windows Certificate Store is not an option. It can be a deployment restriction or a platform limitation. For example, what if you want to create an [Azure WebJob](https://azure.microsoft.com/en-us/documentation/articles/websites-webjobs-resources/) which calls a SOAP Web Service using Certificate-Based Mutual Authentication? (At the time of writing this post) there is no way to store a certificate containing the counterpart's public key in the underlying certificate store for an Azure WebJob. And just because of that, we cannot enjoy all the built-in benefits of WCF for building our client.

[Here](https://msdn.microsoft.com/en-us/library/ms733806.aspx), they explain how to create a **WCF service** that implements custom certificate validation be defining a class derived from **X509CertificateValidator** and implementing an abstract "**Validate**" override method. Once defined the derived class, the **CertificateValidationMode** has to be set to "Custom" and the **CustomCertificateValidatorType** to be set to the derived class' type. This can easily be extended to implement mutual authentication on the service side without using the Windows Certificate Store.

My purpose in this post is to describe how to implement a **WCF client** with Certificate-Based Mutual Authentication without using Windows Certificate Store by compiling the required sources and filling the gaps of the available documentation.

What to consider
-----------------

Before we start thinking about coding, we need to consider the following:

- The WCF client must have access to the client's private key to be able to authenticate with the service.
- The WCF client must have access to the service's public key to authenticate the service.
- Optionally, the WCF client should have access to the service's certificate issuer's certificate (Certificate Authority public key) to validate the service's certificate chain.
- The WCF client must implement a custom service's certificate validation, as it cannot rely on the built-in validation.
- We want to do this, without using the Windows Certificate Store.

Accessing public and private keys without using Windows Certificate Store
--------------------------------------------------------------------------

First we need to access the client's private key. This can be achieved without any problem. We could get it from a local or a shared folder, or from a binary resource. For the purpose of this blog, I will be reading it from a local [Personal Information Exchange (pfx)](https://technet.microsoft.com/en-au/library/dd261744.aspx) file. For reading a pfx file we need to specify a password; thus you might want to consider encrypting or implementing additional security. There are various [X509Certificate2 constructor overloads](https://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509certificate2.x509certificate2(v=vs.110).aspx) which allow you to load a certificate in different ways. Furthermore, reading a public key is easier, as it does not require a password.

Implementing a custom validator method
--------------------------------------

On the other hand, implementing the custom validator requires a bit more thought and documentation is not very detailed. The [**ServicePointManager**
**class**](https://msdn.microsoft.com/en-us/library/System.Net.ServicePointManager(v=vs.110).aspx) has a property called "[**ServerCertificateValidationCallback**](https://msdn.microsoft.com/en-us/library/system.net.servicepointmanager.servercertificatevalidationcallback(v=vs.110).aspx)" of type [**RemoteCertificateValidationCallback**](https://msdn.microsoft.com/en-us/library/system.net.security.remotecertificatevalidationcallback(v=vs.110).aspx) which allows you to specify a custom service certificate validation method. [Here](https://msdn.microsoft.com/en-us/library/system.net.security.remotecertificatevalidationcallback(v=vs.110).aspx) is defined the contract for the delegate method.

In order to authenticate the service, once we get its public key, we could do the following:

- Compare the service certificate against a preconfigured authorised service certificate. They must be the same.
- Validate that the certificate is not expired.
- Optionally, validate that the certificate has not been revoked by the issuer (Certificate Authority). This does not apply for self-signed certificates.
- Validate the certificate chain, using a preconfigured trusted Certificate Authority.

For comparing the received certificate and the preconfigured one we will use the [**X509Certificate.Equals** Method](https://msdn.microsoft.com/en-us/library/e8ey9k04(v=vs.110).aspx). For validating that the certificate has not expired and not been revoked we will use the [**X509Chain.Build** Method](https://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509chain.build(v=vs.110).aspx). And finally, to validate that the certificate has been issued by the preconfigured trusted CA, we will make use of the [**X509Chain.ChainElements** Property](https://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509chain.chainelements(v=vs.110).aspx).

Let's jump into the code.
--------------------------

To illustrate how to implement the WCF client, what can be better than code itself J? I have implemented the WCF client as a Console Application. Please pay attention to all the comments when reading my code. With the provided background, I hope it is clear and self-explanatory.

<script src="https://gist.github.com/pacodelacruz/366e841890b68151bb1d.js"></script>

And here is the **App.config**

<script src="https://gist.github.com/pacodelacruz/9d039a5daf9e99615b36.js"></script>

I hope you have found this post useful, allowing you to implement a WCF client with Mutual Authentication without relying on the Certificate Store, and making your coding easier and happier!

<p style="text-align:center;"><em>Cross-posted on <a href="https://blog.kloud.com.au/author/pacodelacruzag/">Kloud's blog</a>.<br/>
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>