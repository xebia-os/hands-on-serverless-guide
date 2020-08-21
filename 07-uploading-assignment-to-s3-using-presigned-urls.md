# Uploading assignment to Amazon S3 using Pre-signed URLs

In the last post, you learnt how to send an email to candidate with assignment. If you remember the flow, you would have noticed that candidate will be submitting assignment from his/her machine. How will candidate know where to submit the assignment? One solution could be to ask candidate to email his/her assignment. We don't want to go this way as there it leads to following issues: — 1) some candidates send assignment in custom compression formats like rar, or 7z . 2) The code will be stored in email and it become difficult to track after sometime. We want to reduce manual process to minimum so email is a not the right option.

The solution that we came up is to add a build task that packages the code in a zip and upload the assignment to a S3 bucket. We send candidates a project template in the assignment zip. We expect them to code using that template and submit assignment using the Gradle task `gradle submitAssignment`. The build task is made aware of the zip name and S3 location it should upload candidate source code to. As soon as assignment is dropped in a S3 bucket, a lambda function will be notified and will evaluate the assignment.

Workflow for today's use case is as mentioned below:

1. Candidate receives the assignment zip. The assignment zip contains the problem statement in a PDF format as well as the template Java Gradle project that they should use. Later, we can add other project templates as well like Maven.
2. Candidate works on the assignment on his/her machine and when done submits the assigment using a Gradle task `gradle submitAssigmment`. 
3. Gradle `submitAssignment` task packages the source code in a zip and uploads it to a S3 bucket. 
4. `submitAssignment` task relies on couple of properties — `assignmentName` and `assignmentSubmissionUrl`. `assignmentName` is used to give unique name to the assignment and assignmentSubmissionUrl is a pre-signed S3 object URL that will be used to upload the assigment. You can read more about [Pre-Signed S3 URL in the documentation](http://docs.aws.amazon.com/AmazonS3/latest/dev/PresignedUrlUploadObject.html). You can set expiration time to these URLs so that they expire after a defined time. This can be used to enforce requirement that candidates can only submit assignment within a week. Candidate receives these properties in the email and he has to manually replace them in `gradle.properties`.

In today's post, we will extend the code that we wrote last post. Let's get started.

## Step 1: Generating a Pre-signed URL 

Amazon S3 provides the capability to allow clients to upload content to your bucket without them using AWS security credentials or permissions. This is achieved using Amazon S3 Pre-Signed URL. Owner of the S3 bucket creates the Pre-Signed URL using their security credentials, bucket name, an object key, an HTTP method (PUT for uploading objects), and an expiration date and time. The pre-signed URLs are valid only for the specified duration.

Let's create a generatePresignedURL function in the candidate.js file as shown below.

```javascript
const generatePresignedUrl = (candidateId) => {
  const candidateSubmissionBucket = process.env.CANDIDATE_SUBMISSIONS_S3_BUCKET;
  const zipName = `assignment-${candidateId}.zip`;
  const signedUrlExpireSeconds = 7 * 24 * 60 * 60;

  return s3.getSignedUrl('getObject', {
    Bucket: candidateSubmissionBucket,
    Key: zipName,
    Expires: signedUrlExpireSeconds
  })
};
```

The code shown above first creates a unique name for the zip. We uses candidateId to create the unique zip name. Next, we called getSignedURL of the S3 API passing it bucket name, zip name, and duration. We want to keep URL valid for 7 days so we set that duration in seconds.

## Step 2: Updating email template with assigmentName and assignmentSubmissionUrl

Now, we just have to update the sendEmail function so that it also sends the assignmentName and assignmentSubmissionUrl in the email.

```javascript
const sendEmail = (candidateEmail, assignment, signedUrl, candidateId) => {
  var params = {
    Destination: {
      ToAddresses: [
        candidateEmail,
      ]
    },
    Message: {
      Body: {
        Text: {
          Data: `
            Hello,
            Thanks for applying position with us. Please download assignment from ${assignment}.
            Please note that before you submit assignment add following two properties to gradle.properties file.
            
            assigmentName=assignment-${candidateId}.zip
            assignmentSubmissionUrl=${signedUrl}

            Thanks,
            Recruitment Team
          `
        }
      },
      Subject: {
        Data: 'Coding Round'
      }
    },
    Source: '<source_email>',
    ReplyToAddresses: [
      '<reply_to_email>',
    ],
  };
  ses.sendEmail(params).promise();
}
```

We changed `sendEmail` signature to accept signedUrl and candidateId. In the email body, we specifically asked candidate to add two propeties to his/her `gradle.properties` file.

## Step 3: Gradle task for assignment submission

Last, we will add a Gradle task to create assigment zip and upload the zip to S3. This assumes candidate has  already updated gradle.properties with the `assignmentName` and `assignmentSubmissionUrl` properties. We will add the following to assignment zip that we will send to candidate.

```groovy
import java.nio.file.Files
import java.nio.file.Paths

task srcZip(type: Zip) {
    archiveName = project.property("assignmentName")
    from projectDir
    include sourceSets*.allSource.srcDirs*.collect { relativePath(it) }.flatten()
    include 'LICENCE', 'README', 'NOTICE', 'gradlew*', 'build.gradle', 'gradle/**/*'
}

task submitAssignment(dependsOn: srcZip) {

    doLast {
        HttpURLConnection connection = (HttpURLConnection) new URL(project.property("assignmentSubmissionUrl")).openConnection()
        connection.setDoOutput(true)
        connection.setRequestMethod("PUT")
        BufferedOutputStream out = new BufferedOutputStream(
                connection.getOutputStream())
        out.write(Files.readAllBytes(Paths.get("build", "distributions", project.property("assignmentName"))))
        out.close()
        int responseCode = connection.getResponseCode()
        if (responseCode != 200) {
            throw new GradleException("Unable to upload artifact")
        }
    }
}
```

In the code shown above:

1. Firstly,  we created `srcZip` task that zips the candidate source code and puts it in the `build/distribution` folder. Please note that we only package source code and related files. We exclude the `build` and IDE specific directories.
2. Then, we created `submitAssignment` task that takes the zip we created using the `srcZip` task and uploads it to the S3. Please note that `submitAssigment` task is dependent on `srcZip` task so when `srcZip` will be execute before we submit assignment.

## Conclusion

In today's post we extended our application so that candidates can submit assignment from their machine. Today' you learnt about S3 Pre-Signed URL that allows client to upload objects. In the [next post](./08-evaluating-assignment-using-java-lambda-function.md), we will learn how to run the build inside our lambda function and evaluate the assignment.