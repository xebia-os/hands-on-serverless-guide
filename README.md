# Hands-on Guide For Building Serverless Applications with AWS Lambda and the Serverless Framework

Serverless is an overloaded word. Serverless means different things depending on the context. It could mean using third party managed services like Firebase, an event-driven architectural style, the next generation of compute service offered by cloud providers or it could mean a framework to build Serverless applications. This series will start with an introduction to a Serverless compute architecture. Once we understand the basics, we will start developing an application step by step.

In this hands-on article series, you will learn how to build an online code evaluator. In my current organization, one of the interview rounds is a coding round. Candidates are emailed an assignment that they must submit within a week. The assignment is then evaluated by an existing employee who decides whether candidate has passed or failed the round. I wanted to automate this process so that we can filter out inappropriate candidates without any human intervention. A task that can be automated should be automated. This is how the flow will work:

1. Recruitment team submits candidate details to the system.
2. System sends an email with an assignment zip file to the candidate based on the candidate's skills and experience. The zip file contains the problem as well as a Gradle or Maven project.
3. Candidate writes the code and submits the assignment using a Maven or Gradle task like `gradle submitAssignment`. The task zips the candidate's source code and submits it to the system.
4. On receiving assignment, the system builds the project and run all test cases.
   1. If the build fails, the candidate's status is updated to _failed_ in the system and the recruitment team is notified.
   2. If build succeeds, we find the test code coverage and if it is less than a threshold then we mark candidate status to _failed_ and recruitment team is notified.
5. If build succeeds and code coverage is above a threshold, then we run static analysis on the code to calculate the code quality score. If code quality score is below a threshold the candidate is marked _failed_ and a notification is sent to the recruitment team. Otherwise, the candidate passes the round and a human interviewer will now evaluate candidate's assignment.

The source code for application is public under an Apache license. You can find the source code of the project on Github at [lambda-coding-round-evaluator](https://github.com/xebiaww/lambda-coding-round-evaluator).

## Table of Contents

* [Introduction to Serverless computing](./01-introduction-to-serverless.md)
* [Building a REST API in Node.js with Lambda and API Gateway using web console](./02-building-rest-api-in-nodejs-with-lambda-gateway.md)
* [Building a REST API in Node.js with Lambda, API Gateway, DynamoDB, and the Serverless framework](./03-building-rest-api-in-nodejs-with-lambda-gateway-dynamodb-serverless.md)
* [Enabling CORS for your REST API](./04-enable-cors-for-your-rest-api.md)
* [Securing your REST API with API Keys](./05-securing-rest-api-with-api-keys.md)
* [Sending email notifications with Amazon SES on DynamoDB Stream events](./06-sending-email-with-ses-on-dynamodb-stream-events.md)
* [Uploading assignments to Amazon S3 using pre-signed URLs](./07-uploading-assignment-to-s3-using-presigned-urls.md)
* [Evaluating code assignment using a Java based lambda function](./08-evaluating-assignment-using-java-lambda-function.md)

This guide is a living document and additions to it will be made over time as new patterns emerge or we learn new stuff.
