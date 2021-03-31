+++
title = "vCloud Director REST API"
date = "2016-01-16"
description = "vCloud Director REST API"
tags = [
    "vcd"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++

If you find the vCloud Director GUI a little annoying a nice way to get the information you need is via the REST API. Using the REST API you can gather information (GET), update information (PUT) and call operational methods like (POST).

There are many ways you would call the REST API,  if you want to control this programmatically you would be aiming to orchestrate a series of curl commands to issue your GET, PUT and POST commands with the parameters you want to achieve the task you need.

In order to build the syntax for your curl commands there are browser plugins which allow you to work out what you need to do manually.  If you use Chrome the best I’ve found is [PostMAN](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm),  the only thing which annoys me about this is it doesn’t seem to have an option to save your custom header info, so I tend to use Firefox and the [RESTClient plugin](https://addons.mozilla.org/en-US/firefox/addon/restclient/W0Sqc4ZY642oFbg&bvm=bv.83829542,d.d2s) and with that you can save favorites and add and remove these as needed quickly and easily.

I’ll explain remaining of this assuming you are using Firefox RESTclient.

Within RESTclient click Authentication and choose Basic Authentication then complete form
Username:    user@organization
Password:    password

Select Remember Me if you want to save this for future use.

Within RESTclient click Headers choose Custom Header

```bash
Name:        Accept
Value:       application/*+xml;version=5.5
```

Click “Save To Favorites“.

Note where reads 5.5 could read 1.5 or 5.1 if your connecting to earlier versions of vCD.

Change Method To POST and enter URL https://{vcloud ip}/api/sessions

Your completed form should look a little like this.

![vCloud Director Sessions](/images/vcd-sessions.png)

This should return a 200 OK and forms your a session token with VCD,  at this point your now authorized to perform any action via REST as your account allows via the GUI.

To test this you might want to list all vApps,  to do this you would ensure method is set to GET and enter a URL to:
    https://{vcloud ip}/api/vApps/query

For a full list of GET, PUT and POST API calls VMware documents them 
[here.](http://pubs.vmware.com/vcd-55/topic/com.vmware.vcloud.api.reference.doc_55/doc/index.html)