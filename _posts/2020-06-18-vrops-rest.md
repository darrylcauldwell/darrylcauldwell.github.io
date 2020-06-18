---
layout: post
title:  "vRealize Operations Manager (vROps) - REST API"
date:   2020-04-29 20:20:56 +0100
tags:
    - DevOps
permalink: vrops-rest/
---
vRealize Operations Manager (vROps) exposes some of its configuration via RESTful API. With this we can look to programatically control its configuration. My workflow is to use a REST client to explore API and validate payload formatting before moving to script the steps. My REST client of choice is [Postman](https://www.postman.com/).

Performing any REST configuration in vRealize Operations Manager requires at least two steps:

1) Authenticate
2) Perform action, like configure a new vCenter adapter

To faciliate running a sequence of actions I start by creating a POSTMAN collection. Requests stored within the collection can be played in sequence. Using the example of configuring a new vCenter adapter we might create a collection called 'vROps - New vCenter Adapter'.

In case we have multiple environments and want to export our collection and reuse we  want to store environmental specifics as variables. To faciliate this we can create a POSTMAN envrionment. For example we might create a environment called 'Homelab' which has variable vrops-fqdn and both initial and current value of vrops01.example.local
 
vRealize Operations Manager REST API documentation describes how we can [aquire an authentication token](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.api.doc/GUID-C3F0A911-A587-40F7-9998-13D4880A0C2B.html).

Create new reques in Postman with name 'Aquire token', set the request method to POST and set URL to use environment variable like:

```
https://{{vrops-fqdn}}/suite-api/api/auth/token/acquire
```

The documentation shows that two headers are required to be supplied so we must configure our request to pass these:

```
Content-Type: application/json
Accept: application/json
```

The documentation also describes that we need to POST the vROps logon credentials as the body. Within Postman change body type to raw and paste below substrituing credentials appropriate to your environment.

```
{
  "username": "vRealize-username",
  "password": "vRealize-password"
}
```

Successul processing of credential by the REST method returns a JSON body containing the token parameter. We want to capture this to use in the next step in our process. We can do this in Postman by running a script after the request is made which extracts the token value and sets this an environment variable. Change to the Test tab and add the following script.

```
var jsonData = JSON.parse(responseBody);
postman.setEnvironmentVariable("bearerToken", jsonData.token);
```

We can now look at the next step in our example is to add a vCenter adapter which is documented [here](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.api.doc/GUID-18D17D09-628F-4974-AFE4-E94446E3462D.html).

Create new reques in Postman with name 'Aquire token', set the request method to POST and set URL to use environment variable like:

```
https://{{vrops-fqdn}}/suite-api/api/adapters
```

We require to apply three headers to this request, the ones to describe payload is in JSON format and an authentication header which passes the token value environment variable from previous step.

```
Content-Type: application/json
Accept: application/json
Authentication: vRealizeOpsToken {{bearerToken}}
```

The documentation supplies an example body to post with some optional parameters. Similar to aquire step we place this in request body tab and ensure raw format is selected (ensuring you set envioronmentally values).

```
{
  "name" : "New VC Adapter Instance",
  "collectorId" : "1",
  "adapterKindKey" : "VMWARE",
  "resourceIdentifiers" : [ 
    {
      "name" : "VCURL",
      "value" : "https://vcenter.example.locak"
    } 
  ],
  "credential" : {
    "id" : null,
    "name" : "VC-Credential-1",
    "adapterKindKey" : "VMWARE",
    "credentialKindKey" : "PRINCIPALCREDENTIAL",
    "fields" : [ 
      {
        "name" : "USER",
        "value" : "administrator@vsphere.local"
      }, 
      {
        "name" : "PASSWORD",
        "value" : "VC-dummy-passwd"
      } 
    ],
  },
}
```

It is not essential but it is useful for troubleshooting of our request to output the token environment variable value to log prior to attempting to make the request. To do this we can use the 'Pre-request script' tab and add the follwoing script.

```
console.log(pm.environment.get("bearerToken"));
```

We can now run the collection which will open 'Postman runner' which will make the two requests. The summary detail is output to 'Postman running' and you can get more detailed output in 'Postman console' view.