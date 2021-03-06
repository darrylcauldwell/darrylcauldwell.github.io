+++
title = "Calling the NSX REST API with vRO and Navigating The XML Response"
date = "2018-06-20"
description = "vRealize Orchestrator (vRO) Calling The NSX REST API and Navigating The XML Response"
tags = [
    "nsx",
    "vro",
    "rest"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
While the NSX for vSphere plugin for vRealize Orchestrator is useful, occasionally there are limitations. For example when creating an NSX Edge Services Gateway we use this method.

```
NSXEdgeTrinityBasicController.createEdgeV4
```

That method returns void,  so next you need to make a second method call using filter by name to get the object and obtain object ID which is typically the input to other methods.

```
NSXEdgeTrinityBasicController.getAllEdgeSummariesV4
```

When creating NSX Distributed Logical Router it is the same method to create. However the second method to get the object fails as it cannot handle the additional property LogicalRouterScope which DLRs have, so throws an error.

```
java.lang.NoSuchMethodException:com.vmware.vshield.edge.dto.trinity.LogicalRouterScope.<init>()
```

After trying various other NSX plugin methods all have same issue so I changed approach. NSX offers all its capabilities via [REST API](https://docs.vmware.com/en/VMware-NSX-for-vSphere/6.4/nsx_64_api.pdf). vRealize Orchestrator comes with a HTTP-REST plugin, so in theory we can add NSX Managers as HTTP-REST endpoint and call everything direct.

If you have multiple REST HTTP hosts added you would want to bring back a list of these.

```
restHosts = RESTHostManager.getHosts();
```

This returns an array of UUIDs of each REST host, we can use these UUIDs to get the REST host objects. We could normally loop to find the specific [RESThost](http://www.vroapi.com/Class/REST/2.2.2/RESTHost) object we are looking for, but for ease here we would use the first item from array.

```
host = RESTHostManager.getHost(restHosts[0]);
```

So we have the target [RESThost](http://www.vroapi.com/Class/REST/2.2.2/RESTHost) object look at forming the request using the [createRequest](http://www.vroapi.com/Method/REST/2.2.2/RESTHost/createRequest) method.

```
request = host.createRequest('GET','/api/4.0/edges','');
```

Once we have our request formed we can look to make the call using the [executeRequestWithCredentials](http://www.vroapi.com/Method/REST/2.2.2/RESTHost/executeRequestWithCredentials) method. Here I store username and password as Secure String variables.

```
responseStr = request.executeWithCredentials(username,password);
```

This returns a [RESTResponse](http://www.vroapi.com/Class/REST/2.2.2/RESTResponse) object, this has attribute contentAsString to navigate this XML we first convert this string to a [XML DOM Document Object](https://www.w3schools.com/XML/dom_document.asp). vRealize Orchestrator comes with a [XML plugin](http://www.vroapi.com/Plugin/XML/7.0.1) we can use the [XMLManager.fromString](http://www.vroapi.com/Method/XML/7.0.1/XMLManager/fromString) method to perform this converstion.

```
responseXml = XMLManager.fromString(responseStr.contentAsString);
```

If we look inside of the XML which is returned from /api/4.0/edges we can see each the hierachy that each NSX Edge is a XML Node called edgeSummary and within that Node are child nodes for all of its attributes.

```
<pagedEdgeList>
    <edgePage>
        <edgeSummary>
            <objectId>
            <objectTypeName>
            <vsmUuid>
            <nodeId>
            <Revision>
            <type>
            <name>
            ...
        <edgeSummary>
            <objectId>
            <objectTypeName>
            <vsmUuid>
            <nodeId>
            <Revision>
            <type>
            <name>
            ...
```

So we need to bring back all the edgeSummary Nodes and create a [XMLNodeList](http://www.vroapi.com/Class/XML/7.0.1/XMLNodeList) object containing all of out Edges.

```
edgeNodeList = responseXml.getElementsByTagName("edgeSummary");
```

Once we have this we can loop around the children of this and find the correct Edge entry and then get the objectId. We know the child order is always the same so we can choose data from positon in list and output the text content of Node.

```
for (var i = 0 ; i < edgeNodeList.length ; i++) {
    var node = edgeNodeList.item(i) ;
    edgeChildNodeList = node.getChildNodes();
    if (edgeChildNodeList.item(6).textContent == edgeName) {
        var objectId = edgeChildNodeList.item(0).textContent ;
        System.log("Edge named " + edgeName + " has objectId " + objectId) ;
    }
}
```

With this technique we should be able to get around any vRealize Orchestrator NSX Plugin limitation. This would also be useful for calling other RESTful APIs in absence of a specific vRealize Orchestrator plugin.