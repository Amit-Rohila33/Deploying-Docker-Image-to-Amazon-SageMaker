## Deploying Docker Image to Amazon SageMaker

### Step 1: Create a Simple Docker Image

1. Create a file named `Dockerfile` (without any file extension) in a directory with the following content:
(Random example as we discussed)

`Dockerfile`
  ```
    FROM ubuntu:latest
    CMD echo "Hello, World!"
  ```
In our scenario, we'll need to create a `Dockerfile` that encapsulates the ComfyUI API we've developed. Once you've crafted the Dockerfile, save and close it.


### Step 2: Build and Test the Docker Image Locally

I've generated the Docker image named `distillerydummydocker`. Everything is organized accordingly in that manner, `Felipe`.

1. Build the Docker image using the following command:

    ```
    docker build -t distillerydummydocker .
    ```

2. Run the Docker container locally to test it:

    ```
    docker run distillerydummydocker
    ```
This command will display the contents of your Docker image, Felipe. I assume you are already acquainted with these procedures.

### Step 3: Push Docker Image to Amazon ECR

1. Make sure you have the AWS CLI installed and configured with the necessary credentials.

- Attempt utilizing the `Root` user as I encountered several authentication issues with the IAM user. 
- Retrieve the `aws cli` and verify the latest versions accordingly.
- Utilize the `aws configure` command to set up your configuration as the root user, as it will be necessary for the subsequent commands.
- It will prompt you with something similar to the following:

```
AWS Access Key ID [****************BMGB]:
AWS Secret Access Key [****************pGsE]:
Default region name [us-east-1]:
Default output format [json]:
```

- Now, obtain the Access Key ID and Secret Key by navigating to:

```AWS Console → Security Credentials → Create new credentials```
- `IMPORTANT`:- Please configure yourself as a `root` user.

2. Create an Amazon ECR repository:

    ```
    aws ecr create-repository --repository-name distillerydummydocker
    ```
- An ECR repository has been established, and it should appear like this in the `AWS ECR` repositories:

  ![image](https://github.com/Amit-Rohila33/Deploying-Docker-Image-to-Amazon-SageMaker/assets/103894389/f682e958-9efe-40c7-9329-a15e76d5f411)

3. Get the login command to authenticate your Docker client to your registry, this is really an important command to authenticate the `docker` with the `AWS ECR`.

    ```
    aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.<your-region>.amazonaws.com

    In my case, the command was:

    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 610885425359.dkr.ecr.us-east-1.amazonaws.com
    ```

    Replace `<your-region>` and `<your-account-id>` with your AWS region and account ID, respectively.

4. Tag your Docker image with the ECR repository information:

    ```
    docker tag my-docker-image:latest <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/my-docker-repo:latest

    In my case, the command was:

    docker tag distillerydummydocker:latest 610885425359.dkr.ecr.us-east-1.amazonaws.com/distillerydummydocker:latest
    ```

5. Push the Docker image to ECR:

    ```
    docker push <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/my-docker-repo:latest

    In my Case the command was:

    docker push 610885425359.dkr.ecr.us-east-1.amazonaws.com/distillerydummydocker:latest
    ```

Finally, the image will be published to our repository. You have the option to include manual or automated scanning features. The image will appear as follows:

![image](https://github.com/Amit-Rohila33/Deploying-Docker-Image-to-Amazon-SageMaker/assets/103894389/44ca0771-cdd8-41c5-b5bb-c69f39f90153)

### Step 4: Deploy Docker Image to Amazon SageMaker

Before proceeding, `Felipe`, it's essential to create a `domain` and a `role` for SageMaker. I'll guide you through the process. Please note that the configuration should be done with the `Root` user and not an `IAM` one.

- Navigate to `SageMaker`, select `Domain` from the left menu bar, and then click on `Create Domain`. This process may take a few minutes, so please be patient.
- In the left menu bar, go to `Roles`, `create a role`, and fill in the prompts according to your requirements. Proceed to create the role. During this process, you may need to provide details about the previously created `Docker container` and an `S3 bucket` (AWS usually has `three default buckets`). Note that there are some default roles available, so you don't need to worry too much about them.  
- This will look like the following:-
  
  ![image](https://github.com/Amit-Rohila33/Deploying-Docker-Image-to-Amazon-SageMaker/assets/103894389/59b2309e-2353-4bc7-9f71-e06af48546a9)


1. Create a SageMaker model using the ECR URI:

    ```
    aws sagemaker create-model --model-name my-sagemaker-model --primary-container Image=<your-account-id>.dkr.ecr.<your-region>.amazonaws.com/my-docker-repo:latest --execution-role-arn arn:aws:iam::<your-account-id>:role/service-role/your-role-name

    In my case the command was:
    
    aws sagemaker create-model --model-name distillerydummysagemakermodel --primary-container Image=610885425359.dkr.ecr.us-east-1.amazonaws.com/distillerydummydocker:latest --execution-role-arn arn:aws:iam::610885425359:role/service-role/SageMaker-MLOpsEngineer
    ```

    - Navigate to "Inferences" in the left menu, and then proceed to "Models." You should now see a new model listed.

    ![image](https://github.com/Amit-Rohila33/Deploying-Docker-Image-to-Amazon-SageMaker/assets/103894389/db21ef2c-95ce-4707-a7e0-f92d429fccbe)


2. Create an endpoint configuration:

    ```
    aws sagemaker create-endpoint-config --endpoint-config-name my-endpoint-config --production-variants VariantName=my-variant,ModelName=my-sagemaker-model,InstanceType=<your-instance-type>

    In my case, the command was:-

    aws sagemaker create-endpoint-config --endpoint-config-name distillerydummysagemakermodelconfig --production-variants VariantName=distillerydummyvariant,ModelName=distillerydummysagemakermodel,InstanceType=ml.t2.medium
    ```

    Replace `<your-instance-type>` with the desired SageMaker instance type. This is a crucial step and the main reason we are utilizing SageMaker, `Felipe`.

3. Create an endpoint:

    ```
    aws sagemaker create-endpoint --endpoint-name my-endpoint --endpoint-config-name my-endpoint-config

    In my case, the command was:

    aws sagemaker create-endpoint --endpoint-name distillerydummyendpoint --endpoint-config-name distillerydummyendpointconfig
    ```

4. Wait for the endpoint to be in the "InService" state:

    ```
    aws sagemaker wait endpoint-in-service --endpoint-name my-endpoint

    In my case, it was:

    aws sagemaker wait endpoint-in-service --endpoint-name distillerydummyendpoint
    ```

5. Once the endpoint is in service, you can test it by making predictions:

    ```
    aws sagemaker-runtime invoke-endpoint --endpoint-name my-endpoint --body "Hello, SageMaker!" --content-type "text/plain" output.txt

    In my case, it was:-
    
    aws sagemaker-runtime invoke-endpoint --endpoint-name distillerydummyendpoint --body "Hello, world!" --content-type "text/plain" output.txt
    ```
    

That's it! I've successfully created a simple `Docker image`, pushed it to `Amazon ECR`, and deployed it to `Amazon SageMaker` using the AWS CLI. `Felipe`, I kindly ask you to review the steps, and we can schedule a meeting in your morning to discuss further.

During a conversation with my brother, he mentioned that once we have the endpoint, we can utilize `AWS Lambda` to trigger the endpoint via the endpoint API.

Please go through it, `Felipe`; I believe this detailed documentation will be immensely beneficial for our project.
