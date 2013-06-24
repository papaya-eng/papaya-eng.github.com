---
layout: post
title: "Organize Markdown Documents in Google Drive and Google Apps"
description: ""
category: 
tags: []
author: <a href="https://github.com/socrateslee">Chun Li</a>
---
{% include JB/setup %}

##Google Drive and Google Apps
This a just amazoning thing when [Google Drive](https://drive.google.com) connecting with [Google Apps](http://www.google.com/enterprise/apps/business/). And that's exactly when I search something in Google Drive with the scope of [papayamobile.com](http://papayamobile.com).

Google Drive let you store all kinds of documents in your own network file system. You may upload or create plenty of things, integrating with Google Drive SDK, many third party developer have built wonderful apps for viewing, creating, and collaborating via Google Drive. And more important, you can share you documents with others and even editing it with others realtimely.

Despite of the service from Google Marketplace, Google Apps let you organize everything in your Google Drive with regards of the scope of a company or an organization. The best thing for Google Apps I loved is that a document may be shared with "Anyone in the company can search this document" permission. This is just fantastic! You have a document in the cloud, anyone in and only in the company can simply search and access it.

##Markdown in Google Service
Writing [markdown](http://daringfireball.net/projects/markdown/) documents is another joyful process with its' simple but efficient grammer(especially using [dillinger.io](http://dillinger.io/)). So how about manage your markdown documents in Google Drive and Google Apps? I have tested several steps about this.

The first thing is syncing your files to Google Drive. I wrote a simple tool called [gdrivesync](https://github.com/socrateslee/gdrivesync) with [Google Api Python Client](https://code.google.com/p/google-api-python-client/), let you sync local files to a Google Drive folder(shared with proper permissions), yet maintaining the folder structure. Note that by adjusting the uploading parameters, the markdown files can be actually fulltext indexed(yes, you can search the content of the documents).

The second thing is viewing or rendering the markdown documents, the Google Drive support only preview with the source of the documents. I also wrote a little pure browser markdown viewer using [marked javascript lib](https://github.com/chjj/marked) to viewing markdown files in Google Drive, which is [mdReader](https://github.com/socrateslee/mdreader). It would be pretty simple when you set it as your default app for opening .md or .markdown files.
