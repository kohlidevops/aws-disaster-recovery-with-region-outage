# aws-disaster-recovery-with-region-outage

Before start this method, Please deploy workloads using below link

**https://github.com/kohlidevops/aws-disaster-recovery/blob/main/README.md
https://github.com/kohlidevops/aws-disaster-recovery-with-backup-restore-/blob/main/README.md**

**Method – 2: Recover from Region Wide Outage**

Suppose your region (Mumbai) is completely outage. Then we need to recover our application from primary region DynamoDB backup.

Navigate to DynamoDB (NVirginia region) – Backup – select your backup (which is available using AWS Backup) – Restore.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/82a9faeb-032e-4b35-9517-e50551489de7)

Then restore the backup.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/46db0269-b5ff-4014-a239-f4f0cadbe92b)

You can see this restoration process in AWS Backup (NVirginia region) – Jobs.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/12265f0d-8669-4156-bb06-326e1d957103)

Now DynamoDB Table is available state after restoration process in NVirginia region.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/fa0f71e9-52da-42d2-975a-06fd32fe0af2)

Single data is available. Because this backup restoration used AWS Backup (Which Backup plan is Every 12 hours 35 days retention). So the updated data is missing here.

So, If you want to restore the updated data, Then PITR is the solution.

As of now, we go with this process in NVirginia region.

**To create a Lambda Function**

Before, please download the zip file to work with Lambda function.

https://github.com/kohlidevops/aws-disaster-recovery/blob/main/Lambda/productDynamoDB.zip

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/bafd08c2-cbed-4220-9128-e48c68846452)

Let’s go ahead and create a function.

Now upload the productDynamoDB.zip from local to lambda function and save a function.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/85a5c922-fb7e-47b8-8792-3893da4681c3)

Edit the run time and modify the lambda handler name.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/1902c01a-e228-4388-98ea-1af2cd65e90d)

Handler name should be – lambdafunctionname.lambda_handler

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/9c9d9006-e46e-48ad-ad9a-ee20ced95258)

For my case, productDynamoDB.lambda_handler.

Then download the below Payload test event file for Lambda function. Just copy/paste in test event and save. Let's go ahead to test and add a products in a product table.

https://github.com/kohlidevops/aws-disaster-recovery/blob/main/Lambda/AddProductEvent.txt

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/c1db6588-789a-4ceb-9ec8-349d430d41c4)

Here we go, the test is succeeded.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/c21025a0-e9f9-48d8-b08e-8eb4769ebb6f)

To check the DynamoDB table.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/d1dbee2c-261e-4a16-9e9a-a4b29f9d09f1)

Select the table – Explore table names.

**Create an API**

To create a HTTP API – Select - Import.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/888c0c45-b6c9-4362-9654-bccbee74ad78)

Download the file using below link then copy/paste the code.

https://github.com/kohlidevops/aws-disaster-recovery/blob/main/APIGW/APIGatewayProductAPI.json

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/2bbbac2b-5002-4edc-aaa1-23d94aa822ff)

Let’s go ahead to create an API.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/3ab5eed2-3160-4fad-b19e-e066495582b8)

Change integration – update the region and your lambda function just created now

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/5f973029-5ed6-454a-a724-6deedcec9ff1)

Do edit and update the things for all integrations.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/6fd7f20d-bbc8-4bb9-95b1-880f03a98c1b)

Let’s deploy the API. So first create a stage and deploy the API.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/08215e83-77d0-44ab-82a2-8d12b579a71a)
![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/8074cf9c-9c2f-4a1d-acb2-cfce7a0f9394)

Let’s go ahead to create stage as Mumbai. Then deploy this stage

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/f9c1fb64-fb15-42a7-92db-f6143c8c4257)

After deployed, test with API url

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/42a9d5e7-2951-43f4-85c8-2f07ea0b9159)

https://zvr0bj71oi.execute-api.us-east-1.amazonaws.com/virginia/products

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/bcf655e8-c867-47c5-9e03-097059c90e1a)

These items really come from DynamoDB.

We are calling API URL, this API integrates with lambda function, so this API calling lambda function. Then this lambda function will fetch the table values from DynamoDB.

**To launch instance for web server in NVirginia**

Before create 3 security group – EC2 SG (SSH/HTTP) , ALB SG (HTTP) and NullSG (No inbound/outbound).

Launch AMZ Linux2 Ec2 instance and associate DR-EC2SG, then add user data using below link.

https://github.com/kohlidevops/aws-disaster-recovery/blob/main/WebPage/UserData.txt

then let’s go ahead to launch ec2 instance. SSH to machine and update the below API URL in /var/www/html/index.html.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/90c54c56-dd95-4f86-b666-a08b755dfcf5)

Then add the NVirginia in Products Editor line then save.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/b081eae2-ec4c-4bfa-89fb-ec1da64d4501)

**To create a Target Group in AWS**

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/3eff0375-10ac-4b1d-8fa5-dbbeb08b84e8)
![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/5a09cb15-6598-4ebd-b978-10e1dd275a83)

**To create an Application Load balancer**

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/08e05d5c-5d45-443d-b56b-7710caaaffd0)
![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/0fe5b9d7-e006-4014-b773-e2154ce2a3e5)

To attach security group and associate Target group in listener rule

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/9558e7d5-bb5b-41fd-a4b5-71e87e486858)

Let’s go ahead to create a Application Load balancer.

**To update a record in AWS R53**

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/96c7d29b-f97a-4904-bbfc-98df61dec072)

After subdomain creation

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/e515a42b-9d48-4813-86c1-c85aa6a43a4d)

Now add some data. So, we are back to service. NVirginia region has been up and running.

![image](https://github.com/kohlidevops/aws-disaster-recovery-with-region-outage/assets/100069489/73823008-d87a-436b-8ebf-796fa814156f)

That's it.






































