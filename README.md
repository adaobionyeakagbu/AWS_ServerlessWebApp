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
Next step is to implement a Lambda function that will be invoked each time a user requests a unicorn. The function will select a unicorn from the fleet, record the request in a DynamoDB table, and then respond to the frontend application with details about the unicorn being dispatched.

## Phase 4: Deploy a REST API with Amazon API Gateway and Lambda

## Phase 5: Terminate Resources

This is an AWS Hands on project https://aws.amazon.com/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/
