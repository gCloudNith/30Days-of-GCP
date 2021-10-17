<img src="Resources/images/Create-and-Manage-Cloud-Resources-Challenge-Lab.png" width="100%">

# Task 1: Create a project jumphost instance
## Using cloud shell

export INSTANCE_NAME="nucleus-jumphost"
export INSTANCE_TYPE="f1-micro"
export ZONE="us-east1-b"

gcloud compute instances create $INSTANCE_NAME \
--zone=$ZONE \
--type=$INSTANCE_TYPE

## You can also use Cloud console(GUI) to create an instance with the given specifications.

# Task 2: Create a Kubernetes service cluster
## Using cloud shell

gcloud config set compute/zone us-east1-b
gcloud container clusters create mycluster
gcloud container clusters get-credentials mycluster
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-app --type=LoadBalancer --port 8080
kubectl get service

# Task 3: Set up an HTTP load balancer
## Using the cloud shell

## Run the following script. It is to be used while creating an instance template.
## Script startup.sh

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

## Create an instance template

gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh

## Create a target pool

gcloud compute target-pools create nginx-pool

## Create a managed instance template

gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool 
gcloud compute instances list

## Create a firewall rule to allow tcp trafiic at port 80

gcloud compute firewall-rules create www-firewall --allow tcp:80 
gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool
gcloud compute forwarding-rules list

## Create a health check

gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80

## Create a backend service

gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global

## Add the managed instance group to the backend service

gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global

## Create a URL map to route the incoming requests to the backend service

gcloud compute url-maps create web-map \
--default-service nginx-backend 

## Create a target HTTP proxy to route requests to your URL map

gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map

## Create a forwarding rule

gcloud compute forwarding-rules create http-content-rule\
--global \
--target-http-proxy http-lb-proxy \
--ports 80 
gcloud compute forwarding-rules list
