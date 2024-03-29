# AWS_Batch_sample

## 1. Create a C# console application and create the Dockerfile and container

Let's break down the process of creating a C# console sample, writing the Dockerfile, and pushing it to ECR.

**Prerequisites**:

Docker installed and configured on your system

AWS account with permissions to create ECR repositories

**Create a C# Console Sample**

Here's a basic C# console application:

```csharp
using System;

namespace BatchDemoApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello from AWS Batch!");
        }
    }
}
```

Create a .NET project using your preferred method (IDE or dotnet new console). Add this code and save it

**Write the Dockerfile**

Create a file named Dockerfile in the same directory as your C# project. Here's a suitable Dockerfile:

**Dockerfile**

```
# Use a base .NET runtime image
FROM mcr.microsoft.com/dotnet/runtime:6.0 

# Set working directory
WORKDIR /app

# Copy project files
COPY . .

# Build the project
RUN dotnet publish -c Release -o out

# Set the entry point for the container 
ENTRYPOINT ["dotnet", "out/BatchDemoApp.dll"]
```

**Build the Docker Image**

```
docker build -t my-batch-app .
```

**Create an ECR Repository**

```
aws ecr create-repository --repository-name my-batch-app
```

**Authenticate to ECR**

```
aws ecr get-login-password | docker login --username AWS --password-stdin your-aws-account-id.dkr.ecr.your-region.amazonaws.com
```

**Tag and Push the Image**

```
docker tag my-batch-app:latest your-aws-account-id.dkr.ecr.your-region.amazonaws.com/my-batch-app:latest
```

```
docker push your-aws-account-id.dkr.ecr.your-region.amazonaws.com/my-batch-app:latest
```

Now, your image is in ECR, ready for your AWS Batch job!

Explanation of the Dockerfile:

FROM: Starts with a base .NET runtime image

WORKDIR: Sets the working directory within the container.

COPY: Copies your project code into the container

RUN: Executes the command to publish your .NET application in Release mode

ENTRYPOINT: Specifies the command to run when the container starts

## 2. Simple AWS Batch sample

**A simple example** would be:

Here are the steps to provide a sample AWS Batch job using AWS CLI commands:

**Prerequisites**

An AWS account configured with AWS CLI.

Basic understanding of Docker and containerization.

**Create a Compute Environment**

```
aws batch create-compute-environment \
    --compute-environment-name my-batch-compute-env \
    --type MANAGED \
    --state ENABLED \
    --compute-resources type=EC2,minvCpus=0,maxvCpus=20,desiredvCpus=4,instanceTypes=optimal \ 
    --service-role my-batch-service-role 
```

Explanation:

We use a managed compute environment for automatic scaling.

Adjust instance types and vCPUs to suit your workload.

Replace my-batch-service-role with an IAM role providing AWS Batch permissions.

**Create a Job Queue**

```
aws batch create-job-queue \
    --job-queue-name my-batch-job-queue \
    --state ENABLED \
    --priority 1 \
    --compute-environment-order order=1,compute-environment=my-batch-compute-env 
```

**Build and Push Your Docker Image**

This assumes you have:

A Dockerfile ready

An ECR (Elastic Container Registry) repository created

```
docker build -t my-batch-app .
```

```
aws ecr get-login-password | docker login --username AWS --password-stdin your-aws-account-id.dkr.ecr.your-region.amazonaws.com
```

```
docker tag my-batch-app:latest your-aws-account-id.dkr.ecr.your-region.amazonaws.com/my-batch-app:latest
```

```
docker push your-aws-account-id.dkr.ecr.your-region.amazonaws.com/my-batch-app:latest
```

**Create a Job Definition**

```
aws batch register-job-definition \
    --job-definition-name my-batch-job-def \
    --type container \
    --container-properties image=your-aws-account-id.dkr.ecr.your-region.amazonaws.com/my-batch-app:latest,vcpus=2,memory=4096
```

Explanation:

Set image URI to your pushed Docker image

Modify vcpus and memory requirements as needed

**Submit a Job**

```
aws batch submit-job \
    --job-name my-batch-job \
    --job-queue my-batch-job-queue \
    --job-definition my-batch-job-def
```

**Monitor the Job**

```
aws batch describe-jobs --jobs <job-id>  # Insert job ID returned from the previous step
```

**Clean Up**

```
aws batch delete-job-queue --job-queue my-batch-job-queue
```

```
aws batch delete-compute-environment --compute-environment my-batch-compute-env 
```

**Important Notes**

**Error Handling**: Include robust error handling in production cases

**IAM Roles**: Create appropriate roles with necessary permissions

**AWS Batch Documentation**: Thoroughly consult the AWS Batch documentation for additional options, advanced use cases, and best practices: https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html



