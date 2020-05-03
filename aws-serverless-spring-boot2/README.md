# Serverless Spring Boot 2 example
A basic pet store written with the [Spring Boot 2 framework](https://projects.spring.io/spring-boot/). The `LambdaHandler` object is the main entry point for Lambda.

The application can be deployed in an AWS account using the [Serverless Application Model](https://github.com/awslabs/serverless-application-model). The `sam.yaml` file in the root folder contains the application definition

## Installation
To build and install the sample application you will need [Maven](https://maven.apache.org/) and the [AWS CLI](https://aws.amazon.com/cli/) installed on your computer.

In a shell, navigate to the sample's folder and use maven to build a deployable jar.
```
$ mvn package
```

This command should generate a `serverless-spring-boot-example-1.0-SNAPSHOT.jar` in the `target` folder. Now that we have generated the jar file, we can use the AWS CLI to package the template for deployment. 

You will need an S3 bucket to store the artifacts for deployment. Once you have created the S3 bucket, run the following command from the sample's folder:

```
$ aws cloudformation package --template-file sam.yaml --output-template-file output-sam.yaml --s3-bucket <YOUR S3 BUCKET NAME>
Uploading to xxxxxxxxxxxxxxxxxxxxxxxxxx  6464692 / 6464692.0  (100.00%)
Successfully packaged artifacts and wrote output template to file output-sam.yaml.
Execute the following command to deploy the packaged template
aws cloudformation deploy --template-file /your/path/output-sam.yaml --stack-name <YOUR STACK NAME>
```

As the command output suggests, you can now use the cli to deploy the application. Choose a stack name and run the `aws cloudformation deploy` command from the output of the package command.
 
```
$ aws cloudformation deploy --template-file output-sam.yaml --stack-name ServerlessSpringBootSample --capabilities CAPABILITY_IAM
```

Once the application is deployed, you can describe the stack to show the API endpoint that was created. The endpoint should be the `SpringBootPetStoreApi` key of the `Outputs` property:

```
$ aws cloudformation describe-stacks --stack-name ServerlessSpringBootSample
{
    "Stacks": [
        {
            "StackId": "arn:aws:cloudformation:us-west-2:xxxxxxxx:stack/JerseySample/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx", 
            "Description": "Example Pet Store API written with spark with the aws-serverless-java-container library", 
            "Tags": [], 
            "Outputs": [
                {
                    "Description": "URL for application", 
                    "OutputKey": "SpringBootPetStoreApi", 
                    "OutputValue": "https://xxxxxxx.execute-api.us-west-2.amazonaws.com/Prod/pets"
                }
            ], 
            "CreationTime": "2016-12-13T22:59:31.552Z", 
            "Capabilities": [
                "CAPABILITY_IAM"
            ], 
            "StackName": "JerseySample", 
            "NotificationARNs": [], 
            "StackStatus": "UPDATE_COMPLETE"
        }
    ]
}

```

Copy the `OutputValue` into a browser to test a first request.

## Fixing Cold Start Issue
Cold start issue of lambda containers can be fixed using WarmingSchedule property which will keep the lamda container always warm and container never get destroyed. Refer below entries in sam.yml for its configuration.

```
Resources:
  PetStoreFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: com.amazonaws.serverless.sample.springboot2.StreamLambdaHandler::handleRequest
      Runtime: java8
      CodeUri: target/serverless-springboot2-example-1.0-SNAPSHOT-lambda-package.zip
      MemorySize: 1512
      Policies: AWSLambdaBasicExecutionRole
      Timeout: 60
      Events:
        WarmingSchedule:
          Type: Schedule
          Properties:
            Schedule: rate(5 minutes)
            Input: '{ "warmer":true,"concurrency":3 }'
        GetResource:
          Type: Api
          Properties:
            Path: /pets
            Method: any
```
