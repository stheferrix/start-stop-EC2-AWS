# Schedule start and stop AWS EC2 instance using TAG

In this article we abording how to create a schedule to start and stop AWS EC2 instance.
Basically, that operations help admins and operations have more control under costs. That way, it's easier to predict consuming and you can use the calculator:  [AWS Pricing Calculator](https://calculator.aws/#/)

```mermaid
graph LR
B[Lambda Functions] -- have access -- --> A[IAM Role and Policy]
B -- send commd -- --> C[TAG Instance EC2]
D[CloudWatch] -- call -- --> B
```

First of all, it's necessary you have permissions in some services on AWS portal:
- IAM
- EC2
- Lambda
- CloudWatch

**IAM**  - AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources. You can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

**Lambda**  -With Lambda, you can run code for virtually any type of application or backend service - all with zero administration. Just upload your code and Lambda takes care of everything required to run and scale your code with high availability. You can set up your code to automatically trigger from other AWS services or call it directly from any web or mobile app.

**TAG EC2** -  A tag is a label that you assign to an AWS resource. Each tag consists of a _key_ and an optional _value_, both of which you define.

**AWS CloudWatch**  - Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and the applications you run on AWS in real time. You can use CloudWatch to collect and track metrics, which are variables you can measure for your resources and applications. In this case, we using CW to realize actions based in events.

## Create a role
Acess the your AWS account, and go to IAM.
Go to Rules in the left panel and then click in "Create Role".
Choose Lambda Service and the next screen choose "next permissions".

![Identity and Access Management (IAM)](https://www.lazarete.com.br/images/uploads/funcao-lambda-step-02.1.png)

You should redirect to create a new Policy.
In the next screen, choose the format "json" and copy the code below:
    
    {
    "Version": "2012-10-17",
    "Statement": [
        {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": "arn:aws:logs:*:*:*"
        },
           {
                "Effect": "Allow",
               "Action": [
                    "ec2:DescribeInstances",
                    "ec2:DescribeRegions",
                    "ec2:StartInstances",
                    "ec2:StopInstances"
                ],
                "Resource": "*"
                 }
         ]
     }

Click in Review policy, and insert a name and description like **start-stop-ec2-instance**.
If you already finish the create policy, close this windows and go back to screen create a role.
Do a refresh page, and choose the policy that you create.
Click in next, and next.
Now, insert a name and description of your Role, and can be the same name like **start-stop-ec2-instance**.
We are finish the first part of configuration.

## AWS Lambda
Now we create Lambda function. 
Go to AWS Lambda Services, and click "Create a function". This part is responsible to start and stop of EC2 instance. Fill the options like image below:

![null](https://www.lazarete.com.br/images/uploads/lambda-step-03.png)

Keep the configuration default, except Basic Settings. Change the value to 5 minutes:

![null](https://www.lazarete.com.br/images/uploads/lambda-step-07.png)

Fill the Lambda Function code, according options below.

For start functions, fill with this code: https://github.com/lazarete/aws-lambda-start-stop-ec2/blob/master/start-instance.py

And repeat the creation process for stop function using this code: https://github.com/lazarete/aws-lambda-start-stop-ec2/blob/master/stop-instance.py

## TAGs

Now, go to EC2 and configure the tags. The tags we use in this sample, is start the instance at 9am and shutdown the instance at 18pm. Check the image sample below:

![null](https://www.lazarete.com.br/images/uploads/add-tags.png)

We fill key and value exactly we put in the lambda function code.

To do some tests, and feel certain that tags is working, go to Lambda functions and create a event test. Follow the default options and click in create.

![null](https://www.lazarete.com.br/images/uploads/stop-test.png)

Check the state of the instance, an you can see the tags is working.
**Troubleshooting**: If you have some problems to execute this step and return error, check again the Rule and Policy IAM created before, and make sure that you have access to run this operations.

## CloudWatch Rules

Now, we configure Event Rules in CloudWatch to help us schedule the Lambda Functions to start and stop exactly when you want. 

First, go to CloudWatch Service, and choose the options Rules under Events:

![null](https://www.lazarete.com.br/images/uploads/create-rule-cloudwatch.png)

Select Schedule option, and CRON Expression. This way is easier to schedule because you can use the same form you use in crontab Linux OS. Don't forget the name of lambda function that you want to call. (Repeat this steps to create a schedule for stop instance):

![null](https://www.lazarete.com.br/images/uploads/create-rule-cloudwatch-step-1.png)

The AWS timezone is UTC, so convert in your local timezone.
In the next step, configure details like name and description and create a rule:

![null](https://www.lazarete.com.br/images/uploads/create-rule-cloudwatch-step-2.png)

Ready! We finish all the steps and the schedule should working with automatic start and stop instance.
