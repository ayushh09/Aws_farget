# Retrieve Secrets Manager secrets through environment variables in ECS fargate

## Introduction
This document shows how to create a Secrets Manager secret, reference the secret in an Amazon ECS task definition, and then verify it worked by querying the environment variable inside a container showing the contents of the secret.

***

## Implementation
### **Step 1:** Create a Secrets Manager Secret

Created basic secret and stored two key/value pairs in the secrets. As shown below.

![image](https://github.com/ayushh09/Aws_farget/assets/150698783/7ea1e4b2-bfd0-4dca-a24a-840e946192f0)
***

![image](https://github.com/ayushh09/Aws_farget/assets/150698783/da9a0506-3325-41d4-a970-7d39322d98b9)
***

The key/value pairs to be stored in this secret are the environment variable values in our ECS container.

> [!NOTE]  
> see [Creating a Basic Secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/managing-secrets.html) in the AWS Secrets Manager User Guide.

### **Step 2:** Update Your Task Execution IAM Role
For Amazon ECS to retrieve the sensitive data from your Secrets Manager secret, we must have the Amazon ECS task execution role and reference it in our task definition. This allows the container agent to pull the necessary Secrets Manager resources.
To update your task execution IAM role. Use the IAM console to update your task execution role with the required permissions.
1. Open the [IAM console](https://console.aws.amazon.com/iam/).
2. In the navigation pane, choose Roles.
3. Search the list of roles for ecsTaskExecutionRole and select it.
4. Choose Permissions and add inline policy.
5. Choose the JSON tab and specify the following JSON text, ensuring that you specify the full ARN of the Secrets Manager secret you created in step 1.

```shell 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "kms:Decrypt"
      ],
      "Resource": [
        "arn:aws:secretsmanager:region:aws_account_id:secret:secret_name",
        "arn:aws:kms:region:aws_account_id:key/key_id"
      ]
    }
  ]
}


```
- **secretsmanager**:**GetSecretValue**– with this permission to retrieve the secret from Secrets Manager.
- **kms**:**Decrypt**–Required only if your secret uses a customer managed key and not the default key.

### **Step 3:** Create an Amazon ECS Task Definition
We can use the Amazon ECS console to create a task definition that references a Secrets Manager secret. To create a task definition that specifies a secret
Use the IAM console to update your task execution role with the required permissions.
1. Open the [Console](https://console.aws.amazon.com/ecs/v2).
2. In the navigation pane, choose Task definitions.
3. Choose Create new task definition, Create new task definition with JSON.

```shell
{
    "taskDefinitionArn": "arn:aws:ecs:ap-south-1:319425611096:task-definition/ecs-secrets-poc",
    "containerDefinitions": [
        {
            "name": "container-a",
            "image": "public.ecr.aws/nginx/nginx:mainline-alpine3.18-perl",
            "cpu": 0,
            "portMappings": [],
            "essential": true,
            "command": [
                "/bin/sh",
                "-c",
                "while true; do env; sleep 60; done"
            ],
            "environment": [],
            "mountPoints": [],
            "volumesFrom": [],
            "secrets": [
                {
                    "name": "env",
                    "valueFrom": "arn:aws:secretsmanager:ap-south-1:319425611096:secret:poc/service-a/test-r1lGak:env::"
                },
                {
                    "name": "owner",
                    "valueFrom": "arn:aws:secretsmanager:ap-south-1:319425611096:secret:poc/service-a/test-r1lGak:owner::"
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "/ecs/ecs-secrets-poc",
                    "awslogs-region": "ap-south-1",
                    "awslogs-stream-prefix": "ecs"
                }
            },
            "systemControls": []
        }
    ],
    "family": "ecs-secrets-poc",
    "executionRoleArn": "arn:aws:iam::319425611096:role/ecsTaskExecutionRole",
    "networkMode": "awsvpc",
    "volumes": [],
    "status": "ACTIVE",
    "placementConstraints": [],
    "compatibilities": [
        "EC2",
        "FARGATE"
    ],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "1024",
    "memory": "2048",
    "runtimePlatform": {
        "cpuArchitecture": "X86_64",
        "operatingSystemFamily": "LINUX"
    },
    "tags": []
}

```

4. Choose Create.


### **Step 4:** Create an Amazon ECS Cluster

We can use the Amazon ECS console to create a cluster with fargate options. As shown below.

![image](https://github.com/ayushh09/Aws_farget/assets/150698783/aedbaa9d-5179-4465-b17f-b392f7823f0d)
***

### **Step 5:** Run an Amazon ECS Task
Create a service with the same task definition which are created in **Step 3** with below configuration. As shown in the below image.

![image](https://github.com/ayushh09/Aws_farget/assets/150698783/fabd95f9-7ed0-4684-bb53-129c9103b101)


### **Step 6:** Verify

We can verify all of the steps were completed successfully and the environment variable was created properly in our container.
Check the logs. Refer below image.

![image](https://github.com/ayushh09/Aws_farget/assets/150698783/a42c003a-a9d6-4a64-bba2-bf07423c697c)


Secrets are print in container 


