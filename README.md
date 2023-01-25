# AWS_ServerlessWebApp
A Serverless Web Application with AWS Lambda, Amazon API Gateway, AWS Amplify, Amazon DynamoDB, and Amazon Cognito
![Serverless_Architecture d930970c77b382db6e0395198aacccd8a27fefb7](https://user-images.githubusercontent.com/66325142/214476231-1be6e453-65d8-4766-bd3f-6c5348dafe52.png)

The application will present users with an HTML based user interface for indicating the location where they would like to be picked up and will interface on the backend with a RESTful web service to submit the request and dispatch a nearby unicorn. The application will also provide facilities for users to register with the service and log in before requesting rides.

## Phase 1: Static Web Hosting using AWS Amplify
The Wild-Rydes Site is a HTML based fictional interface that enables users to request unicorn rides, created by AWS for practice. The website's source code is stored in an S3 bucket and needs to be copied to your local machine and online repository.
AWS Amplify hosts static web resources including HTML, CSS, JavaScript, and image files which are loaded in the user's browser with continuous deployment (CI/CD) built in. The Amplify Console provides a git-based workflow for continuous deployment and hosting of full-stack web apps.
To begin, I picked a region that has all the services that will be used i.e. eu-central-1 (Frankfurt)

![1](https://user-images.githubusercontent.com/66325142/214478029-27ec038f-670f-4052-ab17-9ca6a2dd3bed.png)

I then created a repository on AWS CodeCommit and set up my IAM user with Git Credentials

![2](https://user-images.githubusercontent.com/66325142/214478969-e7935a00-d8b4-48a2-bd5e-4e8523c9c443.png)

These username and password need to be copied or downloaded somewhere because they are used to connect to CodeCommit on CLI.
Then on CodeCommit, select clone HTTPS from the Clone URL dropdown, and run the following command in CLI to clone your empty codecommit repo to your local machine:

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

I chose 'Get started with Amplify Hosting' since I have the (frontend) and just need to host. 

![4](https://user-images.githubusercontent.com/66325142/214481569-27d33639-f6cd-4167-9451-a33b14efb618.png)

After choosing my repo and branch, i left all other fields as default and deployed.
![5](https://user-images.githubusercontent.com/66325142/214484145-4dcf27e2-f388-4896-a6ed-6f04d2c2d41f.png)


## Phase 2: User Management using Amazon Cognito

## Phase 3: Build Serverless Backend using DynamoDB and AWS Lambda

## Phase 4: Deploy a REST API with Amazon API Gateway and Lambda

## Phase 5: Terminate Resources
