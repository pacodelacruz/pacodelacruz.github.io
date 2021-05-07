---
layout: post
title: Azure Icons Libraries for Diagrams.net (Draw.io)
date: 2020-06-11 10:00
author: Paco de la Cruz
comments: true
category: Architecture
tags: [Architecture, Design, Draw.io, diagrams.net, stencils]
redirect_from:
  - /2020/06/11/azure-icons-library-diagrams-net
---

<base target="_blank"/>

When architecting software solutions it's common to rely on diagrams as a means to bring clarity and consensus across different stakeholders and to document what is being implemented. Over the years I've used many different tools to create architectural diagrams, from Visio, to Power Point, to [Gliffy](https://www.gliffy.com), to [Lucidchart](https://www.lucidchart.com) and more recently [diagrams.net](https://app.diagrams.net/) (better known as Draw.io, its former name)

I've found diagrams.net very convenient and versatile. It offers a cross-platform [desktop app](https://get.diagrams.net), and an [online diagraming tool](https://app.diagrams.net/) that securely works locally in your browser and then you can save your diagram on Google Drive, OneDrive or your device. There is even a [VS Code extension](https://www.diagrams.net/blog/embed-diagrams-vscode) so that you can create and edit these digrams within the VS Code environment. It also has many different [integrations](https://www.diagrams.net/integrations) with third-party tools.

Furthermore, after reading this thread from [@davidfowl](https://twitter.com/davidfowl/), I realised how popular diagrams.net (draw.io) is.

<blockquote class="twitter-tweet" data-dnt="true" data-theme="light"><p lang="en" dir="ltr">What tools are people using to draw architectural boxes and lines diagrams? Visio style. <a href="https://twitter.com/hashtag/lazyweb?src=hash&amp;ref_src=twsrc%5Etfw">#lazyweb</a></p>&mdash; David Fowler #BlackLivesMatter (@davidfowl) <a href="https://twitter.com/davidfowl/status/1252684476125044736?ref_src=twsrc%5Etfw">April 21, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Azure's Refreshed Icons

I work heavily with Azure, and while diagrams.net provides a [icon library for Azure](https://app.diagrams.net/?splash=0&clibs=Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fjgraph%2Fdrawio-libs%2Fmaster%2Flibs%2Fintegration%2Fazure.xml), it is based on the legacy icons.

On November 2019, Microsoft [refreshed their Azure service icons](https://azure.microsoft.com/en-us/blog/10-user-experience-updates-to-the-azure-portal/#Improved%20Icons) with a more modern look and a better visual consistency. I wanted to create diagrams with the new Azure icons, so I decided to create a set of libraries for [Diagrams.net](https://app.diagrams.net/) with them. These libraries were created leveraging the [Azure Icon Collection](https://code.benco.io/icon-collection/) maintained by [benc-uk](https://github.com/benc-uk)

## How to Use the Azure Libraries

To use the libraries on the **online editor**, [click on this link](https://app.diagrams.net/?splash=0&clibs=Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Analytics.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Blockchain.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Compute.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Containers.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Databases.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-DevOps.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Favorites.xml.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-General.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Identity.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Integration.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Intune.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-IoT.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Machine-Learning.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Manage.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Migrate.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Miscellaneous.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Networking.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Security.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Stack.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Storage.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FAzure-Web.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FCommands.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FLogos.xml;Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fpacodelacruz%2Fdiagrams-net-azure-libraries%2Fmaster%2FEnterprise.xml;)

The online editor should look like the screenshot below. You can then close those libraries that you don't need.

![diagrams.net online App](/assets/img/2020/06/diagrams-online.png)

To use the libraries on the **desktop app**, the simplest is to download the [xml-file libraries on GitHub](https://github.com/pacodelacruz/diagrams-net-azure-libraries) and drag the library files that you need into the app as shown in the GIF below.

![diagrams.net desktop app](/assets/img/2020/06/diagrams-desktop.gif)

The same approach can be used for the VS Code extension.

![diagrams.net VS Code extension](/assets/img/2020/06/diagrams-vscode.gif)

## Ownership & Copyright

I do not attribute ownership to any of these icons & images. No copyright infringement is intended.  
All files have been sourced from the [Azure Icon Collection](https://code.benco.io/icon-collection/) maintained by [benc-uk](https://github.com/benc-uk), which in turn were collected by scrapping the public internet and various Microsoft sites. They are included here under fair use.

I hope this set of libraries helps you on your work as an Azure cloud solution architect. Enjoy and happy architecting!
