---
layout: post
title: Interacting with Azure Web Apps Virtual File System using PowerShell and the Kudu API
date: 2016-08-30 22:12
author: Paco de la Cruz
comments: true
categories: [App Service, Azure, Kudu API, PowerShell]
---

![](/assets/img/2016/08/083016_1331_interacting1.png)

Introduction
============

Azure Web Apps or App Services are quite flexible [regarding deployment](https://azure.microsoft.com/en-us/documentation/articles/web-sites-deploy/). You can deploy via FTP, OneDrive or Dropbox, different cloud-based source controls like VSTS, GitHub, or BitBucket, your on-premise Git, multiples IDEs including Visual Studio, Eclipse and Xcode, and using MSBuild via Web Deploy or FTP/FTPs. And this list is very likely to keep expanding.

However, there might be some scenarios where you just need to update some reference files and don't need to build or update the whole solution. Additionally, it's quite common that corporate firewalls restrictions leave you with only the HTTP or HTTPs ports open to interact with your Azure App Service. I had such a scenario where we had to automate the deployment of new public keys to an Azure App Service to support client certificate-based authentication. However, we were restricted by policies and firewalls.

The [Kudu REST API](https://github.com/projectkudu/kudu/wiki/REST-API) provides a lot of handy features which support Azure App Services source code management and deployments operations, among others. One of these is the Virtual File System (VFS) API. This API is based on the [VFS HTTP Adapter](https://github.com/c9/vfs-http-adapter) which wraps a VFS instance as an HTTP RESTful interface. The Kudu VFS API allows us to upload and download files, get a list of files in a directory, create directories, and delete files from the virtual file system of an Azure App Service; and we can use PowerShell to call it.

In this post I will show how to interact with the Azure App Service Virtual File Sytem (VFS) API via PowerShell.

Authenticating to the Kudu API.
===============================

To call any of the Kudu APIs, we need to authenticate by adding the corresponding *Authorization* header. To create the header value, we need to use the Kudu API credentials, as detailed [here](https://github.com/projectkudu/kudu/wiki/Deployment-credentials). Because we will be interacting with an API related to an App Service, we will be using site-level credentials (a.k.a. publishing profile credentials).

Getting the Publishing Profile Credentials from the Azure Portal
----------------------------------------------------------------

You can get the publishing profile credentials, by downloading the publishing profile from the portal, as shown in the figure below. Once downloaded, the XML document will contain the site-level credentials.

![](/assets/img/2016/08/083016_1331_interacting2.png)

Getting the Publishing Profile Credentials via PowerShell
---------------------------------------------------------

We can also get the site-level credentials via PowerShell. I've created a PowerShell function which returns the publishing credentials of an Azure App Service or a Deployment Slot, as shown below.

<script src="https://gist.github.com/pacodelacruz/83a14ee7a81887d52bf7525b439c9500.js"></script>

Bear in mind that you need to be logged in to Azure in your PowerShell session before calling these cmdlets.

Getting the Kudu REST API Authorisation header via PowerShell
-------------------------------------------------------------

Once we have the credentials, we are able to get the *Authorization* header value. The instructions to construct the header are described [here](https://github.com/projectkudu/kudu/wiki/REST-API). I've created another PowerShell function, which relies on the previous one, to get the header value, as follows.

<script src="https://gist.github.com/pacodelacruz/d89cff7d94087bb5755eb1c02c7897b9.js"></script>

Calling the App Service VFS API
===============================

Once we have the *Authorization* header, we are ready to call the VFS API. As shown in [the documentation](https://github.com/projectkudu/kudu/wiki/REST-API), the VFS API has the following operations:

- **GET /api/vfs/{path}**    *(Gets a file at path)*
- **GET /api/vfs/{path}/**    *(Lists files at directory specified by path)*
- **PUT /api/vfs/{path}**    *(Puts a file at path)*
- **PUT /api/vfs/{path}/    ***(Creates a directory at path)*
- **DELETE /api/vfs/{path}**    *(Delete the file at path)*

So the URI to call the API would be something like:

- **GET** https://{webAppName}.scm.azurewebsites.net/api/vfs/

To invoke the REST API operation via PowerShell we will use the [Invoke-RestMethod](https://technet.microsoft.com/en-us/library/hh849971.aspx) cmdlet.

We have to bear in mind that when trying to overwrite or delete a file, the web server implements [ETag behaviour](https://en.wikipedia.org/wiki/HTTP_ETag) to identify specific versions of files.

Uploading a File to an App Service
----------------------------------

I have created the PowerShell function shown below which uploads a local file to a path in the virtual file system. To call this function you need to provide the App Service name, the Kudu credentials (username and password), the local path of your file and the kudu path. The function assumes that you want to upload the file under the *wwwroot* folder, but you can change it if needed.

<script src="https://gist.github.com/pacodelacruz/537cca5666f9ebd81ce36d4b7b6264da.js"></script>

As you can see in the script, we are adding the *"If-Match"="*"* header to disable *ETag* version check on the server side.

Downloading a File from an App Service
--------------------------------------

Similarly, I have created a function to download a file on an App Service to the local file system via PowerShell.

<script src="https://gist.github.com/pacodelacruz/35ed6993b7a1c9e1322c51244def2569.js"></script>

Using the ZIP API
-----------------

In addition to using the VFS API, we can also use the Kudu ZIP Api, which allows to upload zip files and expand them into folders, and compress server folders as zip files and download them.

- **GET /api/zip/{path}**    *(Zip up and download the specified folder)*
- **PUT /api/zip/{path}    ***(Upload a zip file which gets expanded into the specified folder)*

You could create your own PowerShell functions to interact with the ZIP API based on what we have previously shown.

Conclusion
==========

As we have seen, in addition to the multiple deployment options we have for Azure App Services, we can also use the Kudu VFS API to interact with the App Service Virtual File System via HTTP. I have shared some functions for some of the provided operations. You could customise these functions or create your own based on your needs.

I hope this has been of help and feel free to add your comments or queries below.

<p style="text-align:center;"><em>Cross-posted on <a href="https://blog.kloud.com.au/author/pacodelacruzag/">Kloud's blog</a>.<br/>
Follow me on <a href="https://twitter.com/pacodelacruz" target="_blank" rel="noopener noreferrer">@pacodelacruz</a>.</em></p>

