This document shares the major challenges I faced during the implementation of the DevOps assignment, along with the root cause analysis and the solutions I applied.

1. Docker Image Not Found on EC2
   
Challenge : While running Docker on EC2: docker run simple-backend:latest , the image was not found.
Root Cause : Docker images built locally are not available on EC2, no centralized image registry was configured initially
Resolution : Introduced Docker Hub as the image registry

So updated pipeline:
Build image: using the dockerfile
Push image to Docker Hub: by docker login using docker username and access token which is stored in github secrets and pulled through environment variable
Pull image from Docker Hub on EC2

2. GitHub Actions → EC2 SSH Connectivity Failure
Challenge: GitHub Actions pipeline failed during deployment with --> dial tcp ***:22: i/o timeout
Root Cause : EC2 Security Group did not allow inbound SSH (port 22) from GitHub runners
Resolution : Updated EC2 Security Group to allow SSH access , verified public IP and correct SSH key usage and ensured EC2 instance was in a public subnet with an Internet Gateway

3. Docker Daemon Permission Denied in CI/CD Pipeline
Challenge: Pipeline failed with --> permission denied while trying to connect to the Docker daemon socket
Root Cause: Docker was installed but ec2-user was not added to the docker group ,docker daemon runs as root by default
Resolution: Added user to docker group  "sudo usermod -aG docker ec2-user"
Re-logged into the EC2 instance and verified Docker access without sudo

4. Terraform State Locking Failure
Challenge: Terraform initialization failed with --> Error acquiring the state lock , ResourceNotFoundException: Requested resource not found
Root Cause: DynamoDB lock table for Terraform state did not exist ,backend was partially configured
Resolution: Manually created --> S3 bucket for Terraform state , DynamoDB table for state locking , then Re-initialized Terraform backend successfully and finally achieved safe, centralized state management

5. Invalid RDS PostgreSQL Engine Version
Challenge : Terraform failed while creating RDS --> InvalidParameterCombination: Cannot find version 15.3 for postgres
Root Cause: Specified PostgreSQL engine version was not supported in the selected AWS region
Resolution: Updated Terraform configuration to use a supported PostgreSQL version , Re-applied Terraform successfully

6. CloudWatch Logs Visibility Confusion
Challenge: After enabling EC2 monitoring no logs were visible in CloudWatch Log Groups where got confused between metrics vs logs
Root Cause: EC2 monitoring only enables metrics, not logs ,CloudWatch Logs require explicit log group and agent configuration
Resolution: Created CloudWatch Log Groups manually, understood separation i.e
Metrics → CPU, ALB RequestCount
Logs → Application / system logs

and then built a CloudWatch dashboard with EC2 and ALB widgets

7. CI/CD Pipeline Complexity & Debugging
Challenge: Pipeline failed multiple times due to: Missing Docker service ,incorrect execution order ,Dependency on instance state
Resolution: Incrementally fixed pipeline step-by-step ,validated each stage independently: Build, Push ,SSH ,Deploy
Improved understanding of CI/CD failure isolation

8.Best Part was terraform infra , it worked smoothly , very easily created infrastruture using terraform
