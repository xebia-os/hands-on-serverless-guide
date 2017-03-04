# Hands-on Guide For Building Serverless Applications with AWS Lambda and Serverless Framework

Serverless is an overloaded word. Serverless means different things depending on the context. It could mean using third party managed services like Firebase, or it could mean an event driven architecture style or it could mean next generation compute service offered by cloud providers or it could mean a framework to build Serverless applications. This series will start with an introduction to Serverless compute and architecture. Once we learned the basics, we will start developing application in a step by manner.

In this hands-on article series, you will learn how to build an online code evaluator. In my current organization, one of the interview rounds is a coding round. Candidate is emailed an assignment that he/she has to submit in a week time. The assignment is then evaluated by an existing employee who then makes the decision on whether candidate passed or failed the round. I wanted to automate this process so that we can filter out bad candidates without any human intervention. A task that can be automated should be automated. This is how the flow will work:

1. Recruitment team submit candidate details to the system.
2. System sends an email with assignment zip to the candidate based on candidate skills and experience. The zip contains the problem as well as a Gradle or Maven project.
3. Candidate writes the code and submits the assignment using Maven or Gradle task like `gradle submitAssignment`. The task zips the source code of the candidate and submits it to the system.
4. On receiving assignment, systems builds the project and run all test cases. 
   1. If the build fails, then candidate status is updated to failed in the system and recruitment team is notified. 
   2. If build succeeds, then we find the test code coverage and if it is less than a threshold then we mark candidate status to failed and recruitment team is notified.
5. If build succeeds and code coverage is above a threshold, then we run static analysis on the code to calculate the code quality score. If code quality score is below a threshold then candidate is marked failed and notification is sent to the recruitment team. Else, candidate passes the round and a human interviewer will now evaluate candidate assignment.

The source code for application is public under Apache license. You can find source code of the project on Github at [lambda-coding-round-evaluator](https://github.com/xebiaww/lambda-coding-round-evaluator)

## Table of Contents

* [Introduction to Serverless computing](./01-introduction-to-serverless.md)
* [Building a REST API in Node.js with Lambda and API Gateway using web console](./02-building-rest-api-in-nodejs-with-lambda-gateway.md)
* [Building a REST API in Node.js with Lambda, API Gateway, DynamoDB, and Serverless framework](./03-building-rest-api-in-nodejs-with-lambda-gateway-dynamodb-serverless.md)
* [Enabling CORS for your REST API](./04-enable-cors-for-your-rest-api.md)
* [Securing your REST API with API Keys](./05-securing-rest-api-with-api-keys.md)
* [Sending email notification with Amazon SES on DynamoDB Stream events](./06-sending-email-with-ses-on-dynamodb-stream-events.md)
* [Uploading assignment to Amazon S3 using Pre-signed URLs](./07-uploading-assignment-to-s3-using-presigned-urls.md)
* [Evaluating coding assignment using a Java based lambda function](./08-evaluating-assignment-using-java-lambda-function.md)


This guide is a living document and additions to it will be made over time as new patterns emerge or we learn new stuff.

## Authors

1. [Shekhar Gulati](https://twitter.com/shekhargulati)

## 🙏 

Thanks to my employer [Xebia](https://xebia.com/) for giving me opportunity to work with latest technologies. If you are interested in working with us, then you can share your resume at sgulati@xebia.com.

