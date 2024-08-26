# React_App_demo

this repo is for creating a demo react app for learning purpose .

######################

To create a Google Cloud Platform (GCP) CodeBuild pipeline to deploy a React.js application on an NGINX server, you'll typically follow these steps:

##################
CICD CODE 

GITHUB: https://github.com/AttulKumar/React_App_demo/tree/main
######################
Prerequisites
GCP Project: Ensure you have a GCP project set up.
GCP CLI: Install and authenticate the Google Cloud SDK (gcloud).
Source Code Repository: Your React.js application should be stored in a Git repository (e.g., GitHub, GitLab, Bitbucket).
Google Cloud Build: This is a fully managed CI/CD platform on GCP, which you will use to build and deploy your application.
Compute Engine Instance: This will run the NGINX server.
Step 1: Set Up a Compute Engine Instance with NGINX
Create a Compute Engine VM:
1.
Go to the GCP Console.
Navigate to Compute Engine > VM Instances and create a new instance.
Choose a machine type, configure networking, and other settings as needed.

2.
Install NGINX:

SSH into the instance using the GCP console or gcloud CLI.
3.
Run the following commands to install NGINX:

bash
Copy code
sudo apt update
sudo apt install nginx
Configure NGINX for React App:

4.
Modify the NGINX configuration to serve your React app. Typically, you'll edit the default server block in /etc/nginx/sites-available/default:

nginx
Copy code
server {
    listen 80;
    server_name your_domain.com;  # Replace with your domain or public IP

    location / {
        root /var/www/html;  # Directory where your React app will be deployed
        index index.html;
        try_files $uri $uri/ /index.html;  # Ensure React routes work
    }
}
Restart NGINX:

After editing the config file, restart NGINX:

bash
Copy code
sudo systemctl restart nginx
Step 2: Set Up Google Cloud Build
Create a Cloud Build Configuration File (cloudbuild.yaml):

In your React.js project repository, create a cloudbuild.yaml file to define the build steps. This file tells Cloud Build how to build your React app and deploy it to the NGINX server.
Example cloudbuild.yaml:

yaml
Copy code
steps:
- name: 'node:14'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    npm install
    npm run build
- name: 'gcr.io/cloud-builders/gsutil'
  args: ['rsync', '-r', 'build/', 'gs://YOUR_BUCKET_NAME']

- name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    gcloud compute ssh YOUR_INSTANCE_NAME --zone YOUR_ZONE --command "sudo rm -rf /var/www/html/*"
    gcloud compute scp --recurse gs://YOUR_BUCKET_NAME/* YOUR_INSTANCE_NAME:/var/www/html --zone YOUR_ZONE
Replace YOUR_BUCKET_NAME, YOUR_INSTANCE_NAME, and YOUR_ZONE with your actual bucket name, instance name, and zone.

Create a GCS Bucket (if not already created):

This is used as a temporary storage location to transfer the build files to the VM.
bash
Copy code
gsutil mb gs://YOUR_BUCKET_NAME
Enable Required APIs:

Enable the Cloud Build, Compute Engine, and Cloud Storage APIs:
bash
Copy code
gcloud services enable cloudbuild.googleapis.com compute.googleapis.com storage.googleapis.com
Step 3: Set Up Triggers for Automated Deployments
Create a Cloud Build Trigger:

In the GCP Console, navigate to Cloud Build > Triggers.
Click Create Trigger.
Choose your repository and branch to monitor (e.g., main).
Use the cloudbuild.yaml file for the build configuration.
Push Code to the Repository:

Push your React.js code to the configured branch in your repository. This will trigger the Cloud Build pipeline.
Step 4: Testing and Monitoring
Verify Deployment:

After the build completes, visit the IP or domain name of your Compute Engine instance to verify the React app is served correctly via NGINX.
Monitor Builds:

Monitor the builds in the Cloud Build dashboard for any errors or logs.

#################
ERRORS


The error Permission denied (publickey) indicates that the Google Cloud Build service account doesn't have the necessary permissions or access to SSH into the Compute Engine instance. This can happen for several reasons, such as missing SSH keys, incorrect IAM roles, or issues with OS Login.

Steps to Resolve the Issue
1. Ensure SSH Key Propagation
Check if SSH keys are properly set up: Ensure that the SSH keys are correctly propagated to the VM. This might require a few minutes when first set up.

Use OS Login (Recommended): Using OS Login simplifies managing SSH access by linking SSH keys with IAM permissions.

Enable OS Login:

At the project level:
bash
Copy code
gcloud compute project-info add-metadata --metadata enable-oslogin=TRUE
At the instance level:
bash
Copy code
gcloud compute instances add-metadata YOUR_INSTANCE_NAME --metadata enable-oslogin=TRUE --zone YOUR_ZONE
Grant IAM Roles:

Grant the roles/compute.osAdminLogin role to the Cloud Build service account to allow SSH access:
bash
Copy code
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member=serviceAccount:YOUR_CLOUD_BUILD_SERVICE_ACCOUNT \
    --role=roles/compute.osAdminLogin



###################

The error Permission denied (publickey) indicates that the Google Cloud Build service account doesn't have the necessary permissions or access to SSH into the Compute Engine instance. This can happen for several reasons, such as missing SSH keys, incorrect IAM roles, or issues with OS Login.

Steps to Resolve the Issue
1. Ensure SSH Key Propagation
Check if SSH keys are properly set up: Ensure that the SSH keys are correctly propagated to the VM. This might require a few minutes when first set up.

Use OS Login (Recommended): Using OS Login simplifies managing SSH access by linking SSH keys with IAM permissions.

Enable OS Login:

At the project level:
bash
Copy code
gcloud compute project-info add-metadata --metadata enable-oslogin=TRUE
At the instance level:
bash
Copy code
gcloud compute instances add-metadata YOUR_INSTANCE_NAME --metadata enable-oslogin=TRUE --zone YOUR_ZONE
Grant IAM Roles:

Grant the roles/compute.osAdminLogin role to the Cloud Build service account to allow SSH access:
bash
Copy code
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member=serviceAccount:YOUR_CLOUD_BUILD_SERVICE_ACCOUNT \
    --role=roles/compute.osAdminLogin
2. Manually Add SSH Keys to the Instance Metadata
If you're not using OS Login, you might need to manually add the SSH key for the Cloud Build service account to the instance's metadata.

Generate an SSH Key Pair:

Run the following command on your local machine (or Cloud Shell) to generate an SSH key pair if you don’t already have one:
bash
Copy code
ssh-keygen -t rsa -f ~/.ssh/cloudbuild_key -C cloudbuild -N ""
Add the Public Key to the Instance:

Add the public key (~/.ssh/cloudbuild_key.pub) to the instance’s metadata:
bash
Copy code
gcloud compute instances add-metadata YOUR_INSTANCE_NAME \
  --metadata-from-file ssh-keys=~/.ssh/cloudbuild_key.pub --zone YOUR_ZONE
Update cloudbuild.yaml:

Modify your Cloud Build steps to use the corresponding private key for SSH:
yaml
Copy code
- name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    gcloud compute ssh YOUR_INSTANCE_NAME \
      --zone YOUR_ZONE \
      --ssh-key-file ~/.ssh/cloudbuild_key \
      --command "your-command-here"
3. Check IAM Roles for the Cloud Build Service Account
Ensure that the Cloud Build service account has the necessary IAM roles to access the Compute Engine instance.

Assign the Required Roles:

roles/compute.instanceAdmin.v1
roles/iam.serviceAccountUser
roles/compute.osLogin (if using OS Login)
You can assign these roles with the following commands:

bash
Copy code
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member=serviceAccount:YOUR_CLOUD_BUILD_SERVICE_ACCOUNT \
    --role=roles/compute.instanceAdmin.v1

gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member=serviceAccount:YOUR_CLOUD_BUILD_SERVICE_ACCOUNT \
    --role=roles/iam.serviceAccountUser
4. Check VM Firewall Rules
Ensure that the firewall rules allow SSH traffic on port 22.
Create a Firewall Rule (if necessary):

bash
Copy code
gcloud compute firewall-rules create allow-ssh \
    --allow tcp:22 \
    --target-tags=YOUR_INSTANCE_TAG \
    --direction=INGRESS \
    --priority=1000 \
    --network=default
