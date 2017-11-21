## Todo
Note 11/21/2017, there is a pending [bug] (https://github.com/serverless/serverless/issues/2233) with 
[Serverless Application Model (SAM)] (https://github.com/awslabs/serverless-application-model) and Usage Plan's in cloudformation.  There is a work around that will be added shortly

## Known Issues
1. Based on AWS SAM bug, this template will not currently deploy until work around is implemented
2. If Deconfigure.js is run multiple times in succession it will overrwrite some files in the Cloudformation template
## Example

This project is based on the AWSLabs Servless project for Nodejs Express applications and AWS Lambda
[awslabs aws-serverless-express] (https://github.com/awslabs/aws-serverless-express) 

It builds on top of the quick start guide found under the example folder
[example] (https://github.com/awslabs/aws-serverless-express/tree/master/example)

In addition to a basic Lambda function and Express server, the  project includes a [Swagger file](http://swagger.io/specification/), [CloudFormation template](https://aws.amazon.com/cloudformation/aws-cloudformation-templates/) with [Serverless Application Model (SAM)](https://github.com/awslabs/serverless-application-model), and helper scripts to help you set up and manage your application.

## Enhancements from awslabs 

In additional to the awslabs boilerplate this also contains the needed cloudformation configuration to create separate environments (stages) for a CI process, integration with aws Route 53 for friendly API naming, as well as an example Usage Plan for distributing API's

This required changes to the scripts [configure.js] (https://github.com/vernonsoftwaresolutions/aws-serverless-nodejs-boilerplate/blob/master/scripts/configure.js), [deconfigure.js] (https://github.com/vernonsoftwaresolutions/aws-serverless-nodejs-boilerplate/blob/master/scripts/deconfigure.js), and [package.json] (https://github.com/vernonsoftwaresolutions/aws-serverless-nodejs-boilerplate/blob/master/package.json) files to add the additional parameters (hostedzone, dnsname, and stage) to the [cloudformation template] (https://github.com/vernonsoftwaresolutions/aws-serverless-nodejs-boilerplate/blob/master/cloudformation.yaml) for deployment

### Pre-Req's for running the example
This guide assumes you already have
1. [set up an AWS account](http://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/AboutAWSAccounts.html) 
2. have the latest version of the [AWS CLI](https://aws.amazon.com/cli/) installed.
3. A hosted zone named created in AWS Route 53

### Steps for running the example


1. From your preferred project directory: `git clone https://github.com/vernonsoftwaresolutions/aws-serverless-nodejs-boilerplate.git && cd aws-serverless-nodejs-boilerplate`.
2. Run `npm run config -- --account-id="<accountId>" --bucket-name="<bucketName>" --dnsname="<dnsname>" --hostedzonename="<hostedzone>" --stage="<stage>" [--region="<region>" --function-name="<functionName>"]` to configure the example, eg. `npm run config -- --account-id="123456789012" --bucket-name="my-unique-bucket"`. This modifies `package.json`, `simple-proxy-api.yaml` and `cloudformation.yaml` with your account ID, bucket, region and function name (region defaults to `us-east-1` and function name defaults to `AwsServerlessExpressFunction`). If the bucket you specify does not yet exist, the next step will create it for you. This step modifies the existing files in-place; if you wish to make changes to these settings, you will need to modify `package.json`, `simple-proxy-api.yaml` and `cloudformation.yaml` manually.
3. Run `npm run setup` (Windows users: `npm run win-setup`) - this installs the node dependencies, creates an S3 bucket (if it does not already exist), packages and deploys your serverless Express application to AWS Lambda, and creates an API Gateway proxy API.
4. After the setup command completes, open the AWS CloudFormation console https://console.aws.amazon.com/cloudformation/home and switch to the region you specified. Select the `AwsServerlessExpressStack` stack, then click the `ApiUrl` value under the __Outputs__ section - this will open a new page with your running API. The API index lists the resources available in the example Express server (`app.js`), along with example `curl` commands.

See the sections below for details on how to migrate an existing (or create a new) Node.js project based on this example. If you would prefer to delete AWS assets that were just created, simply run `npm run delete-stack` to delete the CloudFormation Stack, including the API and Lambda Function. If you specified a new bucket in the `config` command for step 1 and want to delete that bucket, run `npm run delete-bucket`.

### Creating or migrating a Node.js project based on the example

To use this example as a base for a new Node.js project:

1. Copy the files in the `example` directory into a new project directory (`cp -r ./example ~/projects/my-new-node-project`). If you have not already done so, follow the [steps for running the example](#steps-for-running-the-example) (you may want to first modify some of the resource names to something more project-specific, eg. the CloudFormation stack, Lambda function, and API Gateway API).
2. After making updates to `app.js`, simply run `npm run package-deploy` (Windows users: `npm run win-package-deploy`).

To migrate an existing Node server:

1. Copy the following files from the `example` directory: `api-gateway-event.json`, `cloudformation.yaml`, `lambda.js`, and `simple-proxy-api.yaml`. Additionally, copy the `scripts` and `config` sections of `example/package.json` into your existing `package.json` - this includes many helpful commands to manage your AWS serverless assets and perform _basic_ local simulation of API Gateway and Lambda. If you have not already done so, follow the [steps for running the example](#steps-for-running-the-example) (be sure to copy over `scripts/configure.js`. You may want to first modify some of the resource names to something more project-specific, eg. the CloudFormation stack, Lambda function, and API Gateway API).
2. From your existing project directory, run `npm install --save aws-serverless-express`.
3. Modify `lambda.js` to import your own server configuration (eg. change `require('./app')` to `require('./server')`). You will need to ensure you export your app configuration from the necessary file (eg. `module.exports = app`). This library takes your app configuration and listens on a Unix Domain Socket for you, so you can remove your call to `app.listen()` (if you have a `server.listen` callback, you can provide it as the second parameter in the `awsServerlessExpress.createServer` method).
4. Modify the `CodeUri` property of the Lambda function resource in `cloudformation.yaml` to point to your application directory (e.g. `CodeUri: ./src`). If you are using a build tool (e.g. Gulp, Grunt, Webpack, Rollup, etc.), you will instead want to point to your build output directory.
5. Run `npm run package-deploy` (Windows users: `npm run win-package-deploy`) to package and deploy your application.

To perform a basic, local simulation of API Gateway and Lambda with your Node server, update `api-gateway-event.json` with some values that are valid for your server (`httpMethod`, `path`, `body` etc.) and run `npm run local`. AWS Lambda uses NodeJS 4.3 LTS, and it is recommended to use the same version for testing purposes.

If you need to make modifications to your API Gateway API, modify `simple-proxy-api.yaml` and run `npm run package-deploy`. If your API requires CORS, be sure to modify the two `options` methods defined in the Swagger file, otherwise you can safely remove them. To modify your other AWS assets, make your changes to `cloudformation.yaml` and run `npm run package-deploy`. Alternatively, you can manage these assets via the AWS console.

## Node.js version

This example was written against Node.js version 6.10
