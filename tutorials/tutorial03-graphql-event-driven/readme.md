
# Tutorial 03: Event driven architecture using AWS EventBridge & GraphQL subscriptions

Amazon EventBridge is a serverless event bus that makes it simple to build large-scale event-driven systems with events from your apps, SaaS apps, and AWS services. EventBridge streams real-time data from event sources to AWS Lambda and other SaaS applications in a continuous stream.

In this tutorial we'll use GraphQL subscriptions to receive a continuous stream of data from our source. We'll develop event driven architecture using **Panacloud CLI** and deploy to AWS using CDK.

## Prerequisites

Before getting started you need to install the following packages and libraries:

1. Install [Node.js](https://nodejs.org/en/)
2. Install [AWS  CLI  Version  2.x](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
3. Install [AWS  CDK  Version  2](https://docs.aws.amazon.com/cdk/latest/guide/work-with-cdk-v2.html)
4. Install Globally [TypeScript](https://www.typescriptlang.org/download)

This tutorial assumes that you already have **Panacloud CLI** installed globally, if not then please go ahead and install it first using the following command:

    npm install @panacloud/cli -g

To check the installed version and confirm the successful installation of the Panacloud CLI:

    panacloud version

## Getting Started

For a quick start, we have developed a simple schema (`todoSchema.graphql`) in this directory, you'll need to provide this schema path to CLI in later steps.

Open your favourite shell/terminal in an empty directory and run `panacloud init`, choose the following options as defaults:
<pre><code>? GraphQL Schema File Path <b>./todoSchema.graphql</b>
? API Name <b>MyApi</b>
? Nested Resolver <b>n</b>
? Select Database? <b>Aurora Serverless (Relational)</b>
? Select Database Engine <b>MySQL</b>
</code></pre>

After successfully running the above commands you'll see that CLI has generated an [AWS Cloud Development Kit (CDK)](https://docs.aws.amazon.com/cdk/latest/guide/home.html) project using TypeScript. It comes with all the necessary code to develop and deploy a Serverless GraphQL API in the AWS Cloud. This includes the provisioning of cloud infrastructure in code and Serverless stubs where developers may easily include their business logic. The project also provides pre-built mock lambda functions and unit tests to test your deployed APIs.

## Understanding the schema and the project structure
Before we move forward and deploy the project, let's take a look at the sample schema and the project generated by the CLI.

#### The schema
In order to fully understand the code generated by the CLI, we'll need to take a quick look at the GraphQL schema that's available on the root of this tutorial's directory.
open the file (`todoSchema.graphql`), the file contains a very simple GraphQL schema but one thing you might notice is annotations like `@async` or `@microService` in some places, now if you're familiar with GraphQL then there's a higher chance that you already know that these are GraphQL Directives which are preceded by the `@` character, `@async` and `@microService` are two special custom schema directives that are provided by the Panacloud CLI, these directives tell the CLI to generate the lambda microservices scaffold to handle the EventBridge events for the marked query/mutation, these events will be sent to AppSync via Lambda functions.
you can learn more about GraphQL Directives in this excellent [article](https://stepzen.com/blog/graphql-directives)
#### The project structure

Now let's take a look at what's happening inside the project generated by the CLI, a broad overview of the project strucutre is already covered in the [tutorial 00](https://github.com/panacloud/cli/tree/zia-dev/tutorials/tutorial00-getting-started-graphql#understand-the-project-generated-by-the-cli), it is recommended to read it first to get a good understanding of various directories and files generated by the CLI.
Our main focus for this section will be lmabda files inside `./editable_src/lambdas_stubs/`, open this directory and you'll see that it contains the following folders:
```
📦lambda_stubs  
 ┗ 📂todo  
 ┃ ┣ 📂getTodos   
 ┃ ┃
 ┃ ┣ 📂addTodo 
 ┃ ┣ 📂addTodo_consumer 
 ┃ ┃
 ┃ ┣ 📂deleteTodo  
 ┃ ┣ 📂deleteTodo_consumer
 ┃ ┃ 
 ┃ ┣ 📂updateTodo    
 ┃ ┗ 📂updateTodo_consumer  
 ```
 Every folder inside this directory contains a TypeScript file (`index.ts`) that represents lambda functions where you will write the business logic, you should go ahead and inspect these files carefully.
 Let's try to understand these folders inside `lambdas_stubs/`, if you remember we gave the CLI a GraphQL schema file earlier when generating the project, the schema file had 1 query (`getTodos`) and 3 mutations `(addTodo, updateTodo and deleteTodo)`, now if you take a look in the `lambdas_stubs` directory, you'll notice the CLI has generated one lambda for each of these operations, these can be easily identified by matching the names but wait... there's more, you can also see that there are 3 additional folders with suffix `_consumer` in the same directory, these additional lambdas were generated by the CLI as a result of custom schema directives (as described above) that were part of the schema file (`todoSchema.graphql`), these cosumer lambdas will be consumed by EventBridge to send events to AppSync (generated as a resource by the CLI).
## Deployment
Let's deploy the project generted by the CLI to see what we get, the deployment process is already covered [here](https://github.com/panacloud/cli/tree/zia-dev/tutorials/tutorial00-getting-started-graphql#deploying-the-serverless-api-mocks-in-the-aws-cloud) in great details but we'll quickly summerize it again here:

 1. make sure you already have an AWS account set-up and ready to go.
 2. run the following command to deploy the project `npm run deploy-dev`.
 3. If you get an error regarding `cdk bootstrap` then you need to run the `cdk bootstrap` command before above command.
 
 If everything goes fine then you should be able to see the generated GraphQL API in the terminal, alternatively you can also peek in `cdk-dev-outputs.json` file for the generated API and API key.

## Project Overview in the AWS Console
In this section we'll learn to check the deployed resources and the testing of our GraphQL API.
Open the AWS Console and navigate to [Cloudformation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringStatus=active&filteringText=&viewNested=true&hideStacks=false) page, there you should see your deployed stack **dev-[your-dir-name]Stack**, open this stack and navigate to the **Resources** tab, on this tab you'll see all the various resrouces generated and deployed by the Panacloud CLI on your behalf.

![alt AWS CloudFormation Console](static/cloudformation "AWS CloudFormation Console")

#### Testing the deployed GraphQL API on AWS Console
Again in the **Resources** tab you'll notice that there are some resources of Type **AWS::AppSync::**, these are the resources that are related to your GraphQL API, since we don't have any frontend client yet to talk to our APIs, we'll use the [AWS Console AppSync](https://console.aws.amazon.com/appsync/home?region=us-east-1#/apis) to quickly test our [Queries, Mutations and Subscriptions](https://www.telerik.com/blogs/your-first-look-at-graphql-queries-mutations-and-subscriptions) that were generated by the Panacloud CLI.

 1. Open the [AppSync Console](https://console.aws.amazon.com/appsync/home?region=us-east-1#/apis) 
 2. click on the API that was deployed by the CLI, it should be **[env][api-name-that-you-specified-during-init]** (devmyApi in our case).
 3. on the left sidebar you'll see that there's a `Schema` option, you can click on it to see your GraphQL API schema.
 4. also there should be a `Queries` option, click on it and it will show a GraphQL playground to quickly test all of your Queries and Mutations, you can listen or subscribe to your subscriptions as well.
 5. Before moving forward, test your APIs to make sure they are working fine.

![alt AWS AppSync Console](static/appsync "AWS AppSync Console")

#### Testing your GraphQL API without AWS Console
The Panacloud CLI also comes with a testing client so you can alternatively test all of your APIs if you don't have access to the AWS console, simply run the following command to launch an in-browser instance of [GraphiQL](https://github.com/graphql/graphiql):
``````
panacloud client
``````
![alt GraphiQL instance](static/graphiql "GraphiQL instance")