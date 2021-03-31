+++
title = "DynamoDB With Powershell"
date = "2016-12-20"
description = "DynamoDB With Powershell"
tags = [
    "aws",
    "dynamodb",
    "powershell"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
AWS dynamoDB is a really useful key-value store which is really easy to consume from scripts. However while the AWS Powershell module contains functions for 'Managing Tables' it does not contain functions for 'Reading Data' or 'Modifying Data'.  I had found Julian Biddle had written a [blog post](https://anoriginalidea.wordpress.com/2015/01/20/using-amazon-aws-dynamodb-from-powershell/) about how this might be done by making direct calls to AWS AmazonDynamoDBClient SDK for .net. While this was a useful starting point I had to read around this alot to get it to work how I needed, this post is an explaination of my understanding.

The script this blog post documents is stored [here](https://github.com/darrylcauldwell/dynamoDB-powershell/blob/master/dynamoDB.ps1).

The first thing to do is setup your Powershell session with credentials which have permissions to Read and Write AWS dynamoDB. Here we create a profile and then set this as the default profile to be used.

```powershell
Import-Module AWSPowershell
Set-AWSCredentials -AccessKey <my-access-key> -SecretKey <my-access-key-secret> -StoreAs DynamoDB
Initialize-AWSDefaults -ProfileName DynamoDB -Region eu-west-1
```

We can then use the AWS cmdlets to create a test dynamoDB table.

```powershell
$exampleSchema = New-DDBTableSchema | Add-DDBKeySchema -KeyName "Name" -KeyDataType "S"
$exampleTable = New-DDBTable "myExample" -Schema $exampleSchema -ReadCapacity 5 -WriteCapacity 5
```

Then within our script we add the AmazonDynamoDB .net framework class.

```powershell
Add-Type -Path (${env:ProgramFiles(x86)}+"\AWS SDK for .NET\bin\Net45\AWSSDK.DynamoDBv2.dll")
```

We then create a session to dynamoDB, in this example script we are running from within a Powershell session which already has permissions to DynamoDB. As such we need to specify which region our tables are in so we form a RegionEndpoint object for Ireland and pass this to form a session to that region.

```powershell
$regionName = 'eu-west-1'
$regionEndpoint=[Amazon.RegionEndPoint]::GetBySystemName($regionName)
$dbClient = New-Object Amazon.DynamoDBv2.AmazonDynamoDBClient($regionEndpoint)
```

If we wanted to authenticate within script we would form a credential object and pass that to the command to create the session for example.

```powershell
$dbClient = New-Object Amazon.DynamoDBv2.AmazonDynamoDBClient($creds, $regionEndpoint).
```

As we would typically make various put operations to DynamoDB we create a reuable function which takes parameters. In this example we have a function creating a single Item with two key-value pairs.

```powershell
function putDDBItem{
    param (
        [string]$tableName,
        [string]$key,
        [string]$val,
        [string]$key1,
        [string]$val1
        )
```

We create an object for the PutItemRequest operation, we then assign our tableName parameter as the string value for the TableName property. 

```powershell
$req = New-Object Amazon.DynamoDBv2.Model.PutItemRequest
$req.TableName = $tableName
```

We also need to populate the Item property, this is a dictionary which requires both a string value for the key name and an AttributeValue object. Here we add two key-value pairs to the item object request.

```powershell
$req.Item = New-Object 'system.collections.generic.dictionary[string,Amazon.DynamoDBv2.Model.AttributeValue]'
$valObj = New-Object Amazon.DynamoDBv2.Model.AttributeValue
$valObj.S = $val
$req.Item.Add($key, $valObj)
$val1Obj = New-Object Amazon.DynamoDBv2.Model.AttributeValue
$val1Obj.S = $val1
$req.Item.Add($key1, $val1Obj)
```

Once our object item request is formed we run this against the PutItem method of our dynamoDB database connection

```powershell
$dbClient.PutItem($req)
}
```

As we would typically make various read operations to DynamoDB we create a reuable function which takes parameters. 

```powershell
function getDDBItem{
    param (
            [string]$tableName,
            [string]$key,
            [string]$keyAttrStr
            )

## We create an object for the GetItemRequest operation, we then assign our tableName parameter as the string value for the TableName property. 

$req = New-Object Amazon.DynamoDBv2.Model.GetItemRequest
$req.TableName = $tableName

## We also need to populate the Key property, this is a dictionary which requires both a string value for the key name and an AttributeValue object for the value matching the Item we want to extract. The full command syntax can be found [here](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/DynamoDBv2/TDynamoDBv2GetItemRequest.html).

$req.Key = New-Object 'system.collections.generic.dictionary[string,Amazon.DynamoDBv2.Model.AttributeValue]'
$keyAttrObj = New-Object Amazon.DynamoDBv2.Model.AttributeValue
$keyAttrObj.S = $keyAttrStr
$req.Key.Add($key, $keyAttrObj.S)

## Once we have our request object populated we then run the GetItem method and pass it the object we have formed. Here I adjust the scope of the object to script so this can be used within the script outside of the function.

$script:resp = $dbClient.GetItem($req)
}
```

If we now call the set function and ask it to create an item,

```powershell
putDDBItem -tableName 'myExample' -key 'Name' -val 'Bob' -key1 'Age' -val1 '21'
```

We can repeat this to populate a little more data into the table.

```powershell
putDDBItem -tableName 'myExample' -key 'Name' -val 'Bert' -key1 'Age' -val1 '22'
putDDBItem -tableName 'myExample' -key 'Name' -val 'Sid' -key1 'Age' -val1 '23'
```

Oncew we have some data in we can then call the query function to pull back the the Item where Name matches Bob. 

```powershell
getDDBItem -tableName 'myExample' -key 'Name' -keyAttrStr 'Bob'
```

The item is returned as an object, we can then display the contents of any key value pair such as Hugh's age,  note the key value pair are also objects so we view the .S string attribute.

```powershell
$script:resp.Item.'Age'.S
```

As for this example I scoped the object to script it will not be cleaned so we reset this to $null once we have consumed it.

```powershell
$script:resp = $null
```