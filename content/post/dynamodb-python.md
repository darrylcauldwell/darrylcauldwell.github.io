+++
title = "DynamoDB Python Test Environment with Cloud Formations"
date = "2017-03-31"
description = "DynamoDB Python Test Environment with Cloud Formations"
tags = [
    "aws",
    "dynamodb",
    "python"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
The goal of this post is to walk through the creation of a AWS test environment which I use to explore the Python SDK interactions with DyanmoDB. For learning I use the free tier and so used to create the environment as needed manually. After doing this once I decided to encapsulate the configuration in a Cloud Formations template and then deploy the stack when I needed it.

# Environment Configuration
The resources I require the Cloud Formation template to create
- DynamoDB Table
- Identify Access Management Policy Documents with rights to Put items into DDB and Scan items in DDB
- Identify Access Management Role linked to the policy document
- EC2 Security Group allowing SSH inbound from any IP address
- EC2 Instance with the IAM Role and EC2 Security Group attached, which on boot performs a yum update and installs the AWS Python SDK

The Cloud Formation template I created to meet this requirement is [here on s3](https://s3-eu-west-1.amazonaws.com/cfpythdynamo/PythonDynamoDict.json),  so we can first get this.

```
curl https://s3-eu-west-1.amazonaws.com/cfpythdynamo/PythonDynamoDict.json -o PythonDynamoDict.json
```

To note in order to use this CF template it requires an EC2 access key,  if you pull down the json then search and replace "MyEC2" with the name of your EC2 access key before creating a stack. Assuming your laptop has AWS credentials configured to allow you rights to deploy Cloud Formation templates and create IAM.

```bash
aws cloudformation deploy --template-file PythonDynamoDict.json --stack-name "PythonDDB" --capabilities "CAPABILITY_IAM"
```

This should take less than five minutes and output

```bash
Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - PythonDDB
```

# Connect Python session to DDB
Open an SSD session to the EC2 instance, once session is open,  open a interactive python session
 
```bash
python
```

Then within the python session we can connect to and do stuff with DDB, I've included some simple transactions.

```python
# Import the boto3 and json library

import boto3, json

# Create an object to DynamoDB created by Cloud Formations in Ireland

dynamodb = boto3.resource('dynamodb', region_name='eu-west-1')

# Create an object to the DDB Table created by Cloud Formations

table = dynamodb.Table('myTestDB')

# Clear any previously created values from items object

items = []
item_dict = dict()

# Define a helper class to convert a DynamoDB item to JSON

class DecimalEncoder(json.JSONEncoder):
    def default(self, o):
        if isinstance(o, decimal.Decimal):
            if o % 1 > 0:
                return float(o)
            else:
                return int(o)
        return super(DecimalEncoder, self).default(o)

# Create some DDB Table items

table.put_item(
   Item={
        'ForeName': 'Bobby',
        'FamilyName': 'Johnson',
    },
)
table.put_item(
    Item={
        'ForeName': 'Sam',
        'FamilyName': 'Smith',
    }
)

# Perform a table scan to return all items

response = table.scan()
items = response['Items']

# Cycle through item decobe the json and store the decoded item in a python dictionary named item_dict
# here also updating formatting so only the values are stored.

for item in items:
    decoded_item = (json.dumps(item, cls=DecimalEncoder))
    item_part_format = decoded_item.replace('"FamilyName": ', '')
    item_str_formatted = item_part_format.replace(', "ForeName": ', ':')
    item_dict.update(json.loads(item_str_formatted))

# If we print the dictionary we get the two key pairs

print item_dict
```