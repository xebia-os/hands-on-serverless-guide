# Building a REST API in Node.js with Lambda, API Gateway, DynamoDB, and Serverless framework

Welcome to part 3 of this series. Today, we will take our journey to the next level and build the CRUD API for candidates using Serverless framework.

## What is Serverless framework?

It is a framework that makes it very easy to build applications using AWS Lambda. It does provisioning via Cloud formations. It also scaffolds project structure and take care of deploying functions.

## Getting started with Serverless framework

```shell
$ npm install serverless -g
```

 This will install Serverless command-line on your machine.  You can also use an alias `sls` instead of `serverless` as well.

Now, we will build application in a step by step manner.

## Step 1: Create a node.js Serverless project

Navigate to a convinient location on your filesystem and create a directory `coding-round-evaluator`.

```shell
$ mkdir coding-round-evaluator && cd coding-round-evaluator
```

Once, inside the `coding-round-evaluator` directory,  we will scaffold our first microservice for working with candidates. This will be responsible for saving candidate details, listing candidates, and fetching a single candidate details.

```shell
$ serverless create --template aws-nodejs --path candidate-service --name candidate
```

This will create a directory `candidate-service` with following structure.

```Text
.
├── .npmignore
├── handler.js
└── serverless.yml
```

Let's look at each of these three files one by one.

1. **.npmignore**: This file is used to tell npm which files should be kept outside of the package. 
2. **handler.js**: This declares your lambda function. The created lambda function returns a returns a body with `Go Serverless v1.0! Your function executed successfully!` message. 
3. **serverless.yml**: This file declares configuration that Serverless framework uses to create your service. serverless.yml file has three sections — provider, functions, and resources.
   1. provider: This section declares configuration specific to a cloud provider. You can use it to specify name of the cloud provider, region, runtime etc.
   2. functions: This section is used to specify all the functions that your service is composed off. A service can be composed of one or more functions.
   3. resources: This section declares all the resources that your functions use. Resources are declared using AWS Cloudformation.

## Step 2: Create a REST resource for submitting candidates

Update serverless.yml to the shown below.

```yaml
service: candidate

frameworkVersion: ">=1.1.0 <2.0.0"

provider:
  name: aws
  runtime: nodejs4.3
  stage: dev
  region: us-east-1

functions:
  candidateSubmission:
    handler: api/candidates.submit
    memorySize: 128
    description: Submit candidate information and starts interview process.
    events:
      - http: 
          path: candidates
          method: post

```

Next, create a new directory api inside the candidate-service directory. Move the handler.js to the api directory. Rename handler.js to candidates.js and rename handle to submit.

```javascript
'use strict';

module.exports.submit = (event, context, callback) => {
  const response = {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Go Serverless v1.0! Your function executed successfully!',
      input: event,
    }),
  };

  callback(null, response);

};
```

To deploy the function, you can use `serverless deploy` command. 

```
$ sls deploy
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
.....
Serverless: Stack create finished...
Serverless: Packaging service...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading service .zip file to S3 (524 B)...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
....................................
Serverless: Stack update finished...
Service Information
service: candidate
stage: dev
region: us-east-1
api keys:
  None
endpoints:
  POST - https://05ccffiraa.execute-api.us-east-1.amazonaws.com/dev/candidates
functions:
  candidate-dev-candidateSubmission
```

Now, POST operation of your service is available. You can use tools like cURL to make a POST request.

```shell
$ curl -X POST https://05ccffiraa.execute-api.us-east-1.amazonaws.com/dev/candidates
```

```
{"message":"Go Serverless v1.0! Your function executed successfully!", "input":{...}}
```

## Step 3: Integration with DynamoDB 

Let's write integraton with DynamoDB.

Update provider section of `serverless.yml`

```yaml
provider:
  name: aws
  runtime: nodejs4.3
  stage: dev
  region: us-east-1
  environment:
    CANDIDATE_TABLE: ${self:service}-${opt:stage, self:provider.stage}
    CANDIDATE_EMAIL_TABLE: "candidate-email-${opt:stage, self:provider.stage}"
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:Query
        - dynamodb:Scan
        - dynamodb:GetItem
        - dynamodb:PutItem
      Resource: "*"
```



Next, we will create a new resource that will create DynamoDB table as shown belowl

```yaml
resources:
  Resources:
    CandidatesDynamoDbTable:
      Type: 'AWS::DynamoDB::Table'
      DeletionPolicy: Retain
      Properties:
        AttributeDefinitions:
          -
            AttributeName: "id"
            AttributeType: "S"   
        KeySchema:
          -
            AttributeName: "id"
            KeyType: "HASH"
        ProvisionedThroughput:
          ReadCapacityUnits: 1
          WriteCapacityUnits: 1
        StreamSpecification:
          StreamViewType: "NEW_AND_OLD_IMAGES"
        TableName: ${self:provider.environment.CANDIDATE_TABLE}
```

Now, install couple of dependencies

```Shell
$ npm install --save bluebird
$ npm install --save uuid
```

Update the candidate/api.js as shown below.

```javascript
'use strict';

const uuid = require('uuid');
const AWS = require('aws-sdk'); 

AWS.config.setPromisesDependency(require('bluebird'));

const dynamoDb = new AWS.DynamoDB.DocumentClient();

module.exports.submit = (event, context, callback) => {
  const requestBody = JSON.parse(event.body);
  const fullname = requestBody.fullname;
  const email = requestBody.email;
  const experience = requestBody.experience;

  if (typeof fullname !== 'string' || typeof email !== 'string' || typeof experience !== 'number') {
    console.error('Validation Failed');
    callback(new Error('Couldn\'t submit candidate because of validation errors.'));
    return;
  }

  submitCandidateP(candidateInfo(fullname, email, experience))
    .then(res => {
      callback(null, {
        statusCode: 200,
        body: JSON.stringify({
          message: `Sucessfully submitted candidate with email ${email}`,
          candidateId: res.id
        })
      });
    })
    .catch(err => {
      console.log(err);
      callback(null, {
        statusCode: 500,
        body: JSON.stringify({
          message: `Unable to submit candidate with email ${email}`
        })
      })
    });
};


const submitCandidateP = candidate => {
  console.log('Submitting candidate');
  const candidateInfo = {
    TableName: process.env.CANDIDATE_TABLE,
    Item: candidate,
  };
  return dynamoDb.put(candidateInfo).promise()
    .then(res => candidate);
};

const candidateInfo = (fullname, email, experience) => {
  const timestamp = new Date().getTime();
  return {
    id: uuid.v1(),
    fullname: fullname,
    email: email,
    experience: experience,
    submittedAt: timestamp,
    updatedAt: timestamp,
  };
};
```

Now, you can deploy the function as shown below.

```
$ serverless deploy -v
```

This will create the DynamoDB table.

To test the API, you can use cURL again.

```
$ curl -H "Content-Type: application/json" -X POST -d '{"fullname":"Shekhar Gulati","email": "shekhargulati84@gmail.com", "experience":12}' https://05ccffiraa.execute-api.us-east-1.amazonaws.com/dev/candidates
```

The response you will receive from the API is shown below.

```json
{
  "message":"Sucessfully submitted candidate with email shekhargulati84@gmail.com",
 "candidateId":"5343f0c0-f773-11e6-84ed-7bf29f824f23"
}
```

## Step 4: Get all candidates

Define a new function in the serverless.yml as shown below.

```yaml
  listCandidates:
    handler: api/candidates.list
    memorySize: 128
    description: List all candidates
    events:
      - http: 
          path: candidates
          method: get  
```

Create new function in the `api/candidates.js` as shown below.

```javascript
module.exports.list = (event, context, callback) => {
    var params = {
        TableName: process.env.CANDIDATE_TABLE,
        ProjectionExpression: "id, fullname, email"
    };

    console.log("Scanning Candidate table.");
    const onScan = (err, data) => {

        if (err) {
            console.log('Scan failed to load data. Error JSON:', JSON.stringify(err, null, 2));
            callback(err);
        } else {
            console.log("Scan succeeded.");
            return callback(null, {
                statusCode: 200,
                body: JSON.stringify({
                    candidates: data.Items
                })
            });
        }

    };

    dynamoDb.scan(params, onScan);

};
```

Deploy the function again.

```
$ sls deploy
```

Once deployed you will be able to test the API using cURL.

## Step 5: Get candidate details by id

Define a new function in serverless.yml as shown below.

```Yaml
  candidateDetails:
    handler: api/candidates.get
    events:
      - http:
          path: candidates/{id}
          method: get
```

Define a new function in `api/candidates.js`

```javascript
module.exports.get = (event, context, callback) => {
  const params = {
    TableName: process.env.CANDIDATE_TABLE,
    Key: {
      id: event.pathParameters.id,
    },
  };

  dynamoDb.get(params).promise()
    .then(result => {
      const response = {
        statusCode: 200,
        body: JSON.stringify(result.Item),
      };
      callback(null, response);
    })
    .catch(error => {
      console.error(error);
      callback(new Error('Couldn\'t fetch candidate.'));
      return;
    });
};
```

Now, you can test the API using cURL.

```
curl https://05ccffiraa.execute-api.us-east-1.amazonaws.com/dev/candidates/5343f0c0-f773-11e6-84ed-7bf29f824f23
{"experience":12,"id":"5343f0c0-f773-11e6-84ed-7bf29f824f23","email":"shekhargulati84@gmail.com","fullname":"Shekhar Gulati","submittedAt":1487598537164,"updatedAt":1487598537164}
```



## Working with local DynamoDB

Download the jar and run locally.

## Invoking functions locally and remotely

```
sls invoke local -f function-name -p event.json
```

## Tailing the logs

```
sls logs -f candidateDetails -t
```

## Conclusion

In this part, you learnt about Serverless framework and how you can use it to create services. In the [next part](./04-enable-cors-for-your-rest-api.md), we will learn how to enable CORS for our REST API.