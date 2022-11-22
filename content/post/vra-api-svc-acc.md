+++
title = "Service accounts with VMware Aria Automation API"
date = "2022-11-22"
description = "Service accounts with VMware Aria Automation API (formerly vRealize Automation)"
tags = [
    "vra",
    "devops",
    "cloud-assembly",
    "pipeline",
    "postman",
    "python",
    "rest"
]
categories = [
    "automation"
]
+++
VMware [VMware Aria Automation](https://www.vmware.com/products/vrealize-automation.html) is a product suite comprising three products, [Cloud Assembly](https://docs.vmware.com/en/vRealize-Automation/SaaS/Using-and-Managing-Cloud-Assembly/GUID-B9291A02-985E-4BD3-A11E-BDC839049072.html), [Service Broker](https://docs.vmware.com/en/vRealize-Automation/SaaS/Getting-Started-Service-Broker/GUID-8DDBB69B-6316-40FC-B584-C4F89643FA27.html) and [Code Stream](https://docs.vmware.com/en/vRealize-Automation/SaaS/Using-and-Managing-CodeStream/GUID-3625AE99-C60C-4517-803B-18C526ADCFF1.html). These and other products are available 'as a Service' via the [VMware Cloud Services Portal](https://console.cloud.vmware.com).

##  User to Cloud Services API Credentials

Most of the API documentation is centred around a user making call using API tokens. API tokens are issued by users in an organization and are associated with the user’s account.

The first step of a programatic flow is to pass the API token details to the Authorization Server. A Bearer Token is returned it is not intended to have any meaning to clients using it. The name “Bearer authentication” can be understood as “give access to the bearer of this token.” The Bearer Token is used in subsequent calls to the REST API.

## Service to Cloud Services API Credentials

A common usecase is for servers and application services to make programmatic calls to REST APIs. If user aligned API tokens this can cause account lifecycle management headaches. VMware Cloud Services supports OAuth2 apps, the owner of the OAuth app is the organization in which it is created. These credentials can be managed by  user assigned the organization owner or developer roles.

VMware Cloud Services supports the creation of different OAuth2 apps:

* Server to server app - Tokens are issued directly to your app
* Web app - Users of your app authorize access to protected resources
* Native/Mobile app - Authorization Code Grant (Public client)

Some requirements will drive tightly coupled interactions and the building of client web or native applications. More typically cross product infrastructure managements tasks are loosely coupled.  OAuth2 Server to server app tokens are useful for these loosely coupled integrations. The programatic flow for these is very similar to that of the API token. The first step is to pass the OAuth2 API ID and APP Secret to the Authorization Server. A Bearer Token is returned which can be used in subsequent calls to the REST API.

## Creating A Cloud Services OAuth2 Server to server app

To create a OAuth2 Server to server app you must connect to the [VMware Cloud Services Portal](https://console.cloud.vmware.com) with user account with sufficient priviliges.

Navigate to Organization and then OAuth Apps and follow the Create App wizard.

![Creating A Cloud Services OAuth2 Server to server app](/images/vra-api-svc-acc-create-app-1.png)

Add any Organization Role memberships which offer the right priviliges to the application calling the API.

![Adding a Organization Role to a Server to server app](/images/vra-api-svc-acc-create-app-2.png)

Add any Service Role memberships which offer the right priviliges to the application calling the API.

![Adding a Service Role to a Server to server app](/images/vra-api-svc-acc-create-app-3.png)

The creation wizard will output an Application ID and a Application Secret, store these securely.

## Calling vRealize Automation API Using A OAuth2 Server to server app

In this test of functionality I am using a simple REST client.  The Application ID and a Application Secret need to be securely stored.  To similuate secret management solution I store these as environment variables along with the URL of the VMware Cloud Services API.

![REST client environment variables](/images/vra-api-svc-acc-env-var.png)

With the secrets stored the next step is to form a POST to the Authorization Server.  This needs to have Authorization Username value set as the Application ID and the Password value set as the Application Secret.  As in my test these are environment variables I simply reference these.

![VMware Cloud Services API Authorization Server POST](/images/vra-api-svc-acc-rest-post.png)

The Authorization Server also requires a POST body to body in 'x-www-form-unicode' where a key value pair is passed.  The key needs to set to 'grant_type' and the value set to 'client_credentials'.

![VMware Cloud Services API Authorization Server POST Body](/images/vra-api-svc-acc-rest-post-body.png)

When POST is made if Application ID and a Application Secret are accepted by the API a Bearer token is returned in the response.

![VMware Cloud Services API Bearer Token](/images/vra-api-svc-acc-bearer.png)
