---
layout: post
title:  "Microservice HTTP Endpoint For DynamoDB"
date:   2016-12-29 11:20:56 +0100
tags:
    - Public Cloud
permalink: lambda-dynamodb/
---
My goal here is to setup a simple RESTful API which accepts GET and POST methods to trigger a Lambda Function to read and put information into DynamoDB.

First we'll create a test DynamoDB table and put some items into it,

    aws dynamodb create-table --table-name 'MyTable' --attribute-definitions '[{"AttributeName": "Name", "AttributeType": "S"}]' --key-schema '[{"AttributeName": "Name", "KeyType": "HASH"}]' --provisioned-throughput '{"ReadCapacityUnits": '5', "WriteCapacityUnits": '5'}'

    aws dynamodb put-item --table-name 'MyTable' --item '{"Name": {"S": "David"},"Age": {"S": "42"}}'

    aws dynamodb put-item --table-name 'MyTable' --item '{"Name": {"S": "Brian"},"Age": {"S": "22"}}'

    aws dynamodb put-item --table-name 'MyTable' --item '{"Name": {"S": "Sean"},"Age": {"S": "35"}}'

Then if we select Lambda in AWS console, and create a new function, then at 'Select blueprint' select the blueprint named 'Blank Function'. At next page click dashed box to configure a API Gateway trigger and change it to have Open security. Specify the name as 'myTestLambda', the runtime as 'Python 2.7', the Role as 'Create new role from template(s), the role name as 'myTestLambdaRole', the Policy template as 'Simple Microservice permissions' and the handler as 'lambda_function.handler'.

Paste the following as Lambda code

    from __future__ import print_function
    import boto3
    import json
    print('Loading function')
    def handler(event, context):
        operation = event['operation']
        if 'tableName' in event:
            dynamo = boto3.resource('dynamodb').Table(event['tableName'])
        operations = {
            'create': lambda x: dynamo.put_item(**x),
            'read': lambda x: dynamo.get_item(**x),
            'update': lambda x: dynamo.update_item(**x),
            'delete': lambda x: dynamo.delete_item(**x),
            'list': lambda x: dynamo.scan(**x),
            'echo': lambda x: x,
            'ping': lambda x: 'pong'
        }
        if operation in operations:
            return operations[operation](event.get('payload'))
        else:
            raise ValueError('Unrecognized operation "{}"'.format(operation))

We can then test that we can pull back a specific database item for example the Brian record, select 'Action \ Configure test event' and paste in the following.

    {
        "operation": "read",
        "tableName": "MyTable",
        "payload": { "Key": {"Name": "Brian"}}
    }

By clicking Test we can do an internal test of the function and pull back the DynamoDB item.

    {
    "Item": {
        "Age": "22",
        "Name": "Brian"
    }
    }

To do a table scan rather than targetted get,

    {
        "operation": "list",
        "tableName": "MyTable",
        "payload": { "TableName": "MyTable"}
    }

