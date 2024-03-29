# Serverless Web App Deployed on AWS
A Serverless Web Application using AWS Lambda, Amazon API Gateway, AWS Amplify, Amazon DynamoDB, and Amazon Cognito
![Serverless_Architecture d930970c77b382db6e0395198aacccd8a27fefb7](https://user-images.githubusercontent.com/66325142/214476231-1be6e453-65d8-4766-bd3f-6c5348dafe52.png)

User story: WildRydes is a fictional interface that enables users to request unicorn rides. Users visit the WildRydes site which is a HTML based user inteface deployed using AWS Amplify. A user can sign up and log into the application, and this authentication is enabled by Amazon Cognito. When users are authenticated, they can request rides and indicate the location where they would like to be picked up on the application. The javascript running on the site invokes the request and interfaces on the AWS backend via an Amazon API Gateway to dispatch a nearby unicorn. An AWS Lambda function is reponsible for selecting a unicorn from the fleet, recording the request in a DynamoDB table, and then responding to the frontend application (via the REST API) with details about the unicorn being dispatched. Easy peasy! 
So let's see how I implemented this!

## Phase 1: Static Web Hosting using AWS Amplify
The Wild-Rydes Site is a HTML based fictional interface, created by AWS for practice. The website's source code is stored in an S3 bucket and needs to be copied to local machine and online repository.
AWS Amplify hosts static web resources including HTML, CSS, JavaScript, and image files which are loaded in the user's browser with continuous deployment (CI/CD) built in. The Amplify Console provides a git-based workflow for continuous deployment and hosting of full-stack web apps.
To begin, I picked a region that has all the services that will be used i.e. eu-central-1 (Frankfurt)

![1](https://user-images.githubusercontent.com/66325142/214478029-27ec038f-670f-4052-ab17-9ca6a2dd3bed.png)

I then created a repository on AWS CodeCommit and set up my IAM user with Git Credentials

![2](https://user-images.githubusercontent.com/66325142/214478969-e7935a00-d8b4-48a2-bd5e-4e8523c9c443.png)

These username and password need to be copied or downloaded somewhere because they are used to connect to CodeCommit on CLI.
Then on CodeCommit, I selected clone HTTPS from the Clone URL dropdown, and ran the following command in CLI to clone the empty codecommit repo to my local machine:

``` $git clone https://git-codecommit.eu-central-1.amazonaws.com/v1/repos/reponame ```

I was prompted to fill in my username and password.
For the next step, I needed to install AWS on my local machine and configure my iam credentials on CLI. You can simply download AWS for CLI (I did this via my company portal). Then I generated an access key for my IAM user and filled in the credentials on CLI.

![3](https://user-images.githubusercontent.com/66325142/214480412-9d5f811f-2f23-404c-b684-dc4a2374558e.png)
       
To copy the wild-rydes site content from the publicly accessible s3 bucket to my local machine, i ran this command:

``` $ aws s3 cp s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website ./ --recursive ```
            
Then pushed the files to my codecommit repo:
```
            $ git add .
            $ git commit -m 'new'
            $ git push
```
#### Next up ----> AWS Amplify Console

Advantage of using Amplify to host the site is that the Amplify Console will automatically rebuild and redeploy the app when it detects changes to the connected repository.

To proceed: I chose 'Get started with Amplify Hosting' since I have the (frontend) and just need to host. 

![4](https://user-images.githubusercontent.com/66325142/214481569-27d33639-f6cd-4167-9451-a33b14efb618.png)

After choosing my repo and branch, i left all other fields as default and deployed.
![5](https://user-images.githubusercontent.com/66325142/214484363-6e9f010e-fda7-4841-8225-6e2f81a745fa.png)


## Phase 2: User Management using Amazon Cognito

Amazon Cognito provides two different mechanisms for authenticating users. Cognito User Pools allows to add sign-up and sign-in functionality to your application while Cognito Identity Pools allows to authenticate users through social identity providers such as Facebook, Twitter, or Amazon, with SAML identity solutions, or by using your own identity system. I used the User Pool for this exercise.

On the Cognito service dashboard, I clicked 'Create User Pool'
<img width="1101" alt="Screenshot 2023-03-17 at 16 54 24" src="https://user-images.githubusercontent.com/66325142/225956153-39f9e3a4-f12e-4316-9651-47526515fbe5.png">
There are 6 steps involved in creating the user pool. I will describe what i did in each step.
1. Configure Sign in Experience: I left as default but selected 'User name' and 'Email' under user pool sign in options.
2. Configure Security Requirements: Selected 'No MFA' but left others as default.
3. Configure Sign-up Experience: I left as default.
4. Configure Message Delivery: I selected 'Send Email with Amazon SES' and selected a FROM Email address. This email address needs to be created and verified on the SES service dashboard for it to be visible. Also extra attention has to be paid to the SES region because the region needs to correspond to the region the FROM Email was created in for it to be visible. I also configured a FROM Sender Name but this is optional.
5. Integrate Your App: I inputed a User Pool name, and under the Initial App Client, I selected 'Public Client' and inputed an App Client Name. Everything else as default.
6. Review and Create: Reviewed the information and Created the user pool.

Next, I opened the js/config.js file on my local machine and input the pool ID, client ID and region, and then commit the updated file back to aws.
<img width="854" alt="Screenshot 2023-03-18 at 10 00 25" src="https://user-images.githubusercontent.com/66325142/226095954-afc07675-4e5d-443f-b84f-fd9352520fd9.png">

```
       $ git add js/config.js 
       $ git commit -m "updated js config"
       $ git push
```
Users can now register on the /register.html page of the site!
<img width="854" alt="Screenshot 2023-03-19 at 02 34 19" src="https://user-images.githubusercontent.com/66325142/226149207-9990bd1d-412b-40d9-b5fc-e361d3b1f235.png">
<img width="854" alt="Screenshot 2023-03-19 at 02 39 31" src="https://user-images.githubusercontent.com/66325142/226149213-c419df72-9ab9-4aaa-9377-473a28dbecd9.png">
Cool thing is I can verify users on the Cognito user pool backend, comes in handy if i used a dummy email address to register.

## Phase 3: Build Serverless Backend using DynamoDB and AWS Lambda
Next step is to implement the Lambda function that will be invoked each time a user requests a unicorn. The function will select a unicorn from the fleet, record the request in a DynamoDB table, and then respond to the frontend application with details about the unicorn being dispatched.

First, I created a DynamoDB table with name Rides and partition key RideId of type string. When the table was created, I copied its ARN.
For a Lambda function to be performed, an IAM role needs to be defined for it. This role defines what other AWS services the function is allowed to interact with. So, I needed to create an IAM role that grants the Lambda function permission to write logs to Amazon CloudWatch Logs and access to write items to my DynamoDB table.

<img width="1079" alt="Screenshot 2023-03-19 at 04 18 02" src="https://user-images.githubusercontent.com/66325142/226151750-58a50ade-db38-419e-85f6-3a5cdaedcc7c.png">

Then i added the AWSLambdaBasicExecutionRole permissions policy, inputed a role name and created the Role. After the role was created, I navigated to the role's permissions page to Create Inline Policy as shown below.

<img width="1260" alt="Screenshot 2023-03-19 at 04 28 52" src="https://user-images.githubusercontent.com/66325142/226152027-1c49f230-0cda-4893-9054-2ba81c543c84.png">

After reviewing the policy and giving it a name, the policy was created. Basically, the role I created, along with its policies, allows lambda functions to be executed and gives the functions write access to my specified DynamoDB table. 

Next, I created a Lambda function from scratch using source code from https://webapp.serverlessworkshops.io/serverlessbackend/lambda/requestUnicorn.js
This function uses the Node.js 16.x version for Runtime and the existing role with necessary permissions I created. Every other setting was left as default. Caution: Watch out for the region that this function is created in, that it aligns with the role and database that you want it to write with/to.
After deploying the code, i tested it via this test event (since the rest api gateway has not yet been configured).
````
{
    "path": "/ride",
    "httpMethod": "POST",
    "headers": {
        "Accept": "*/*",
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        "content-type": "application/json; charset=UTF-8"
    },
    "queryStringParameters": null,
    "pathParameters": null,
    "requestContext": {
        "authorizer": {
            "claims": {
                "cognito:username": "the_username"
            }
        }
    },
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
````
And success!

<img width="1310" alt="Screenshot 2023-03-20 at 01 39 34" src="https://user-images.githubusercontent.com/66325142/226221357-979fc3d4-0cff-4316-a9b6-5a7386e6e0be.png">

## Phase 4: Deploy a REST API with Amazon API Gateway and Lambda

This is the step that ties all the components together. The static page at /ride.html has a simple map-based interface for requesting a unicorn ride. After authenticating using the /signin.html page, users will be able to select their pickup location by clicking a point on the map and then requesting a ride by choosing the "Request Unicorn" button in the upper right corner.
The Amazon API Gateway created in this step will expose the Lambda function built in the previous phase as a RESTful API. This API will be accessible on the public Internet. It will be secured using the Amazon Cognito user pool created in the previous phases. This turns my statically hosted website into a dynamic web application by adding client-side JavaScript that makes AJAX calls to the exposed APIs.

First things first, I navigated to API gateway and created my REST api, named WildRydes, with edge-optimized as the endpoint type. This is because Edge optimized are best for public services being accessed from the Internet. Regional endpoints are typically used for APIs that are accessed primarily from within the same AWS Region.

Next, I created a 'ride' Resource, enabling API Gateway CORS. Under that resource, I created a POST method with Lanbda function as the integration type. Then i checked the box for Use Lambda Proxy integration. Selecting the region that i created my lambda function in, I selected my lambda function from the dropdown. Then saved the method.
The next thing is to select the authorizer for this method. However, there is one important thing I had to do. I had to create the authorizer that controls the interaction between the user pool and the api. So, on the left side of the API explorer, i clicked on authorizer and created one, with Cognito as the type. https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-enable-cognito-user-pool.html

<img width="457" alt="Screenshot 2023-03-26 at 04 47 47" src="https://user-images.githubusercontent.com/66325142/227752471-cce8cb75-90fe-491b-9c4b-e23c0b5d2cac.png">

Next, I navigated back to my API POST method, to the Method Request card, and selected my new authorizer under Authorization. Then I deployed the API by creating a new stage called prod. The invoke URL was copied and updated in the config.js file, under api invoke url. Then pushed back to my repo.

One thing to note is if you make any changes to the API, you'd need to redploy it to see changes. For code changes to the site, Amplify automatically deploys them.

### And we're live! 

This is what the unicorn request process is, after the user logs in.
<img width="1436" alt="Screenshot 2023-03-21 at 01 11 34" src="https://user-images.githubusercontent.com/66325142/226608066-56b85baa-5711-4766-b4d5-f4e700cebb22.png">
<img width="1436" alt="Screenshot 2023-03-26 at 04 35 43" src="https://user-images.githubusercontent.com/66325142/227752192-abd0f08c-a7ea-4a4e-8135-6993b842f9c8.png">

## Phase 5: Terminate Resources
Unfortunately, I had to delete all deployed resources so i don't get charged too much. But I spent quite a number of days on this so charges were unavoidable. I had fun though.


This is an AWS Hands on project https://aws.amazon.com/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/
