<img src="/Resources/images/Create-and-Manage-Cloud-Resources-Challenge-Lab.png" width="100%">

<h1 align="center">Task 1: Create a project jumphost instance</h1>

<h3>Using cloud shell</h3>

```
gcloud compute instances create nucleus-jumphost \
          --network nucleus-vpc \
          --zone us-east1-b  \
          --machine-type f1-micro  \
          --image-family debian-9  \
          --image-project debian-cloud
```

<h3> You can also use Cloud console(GUI) to create an instance with the given specifications.</h3>

<br>
<h1 align="center"> Task 2: Create a Kubernetes service cluster</h1>
<h3> Using cloud shell</h3>

```
gcloud container clusters create nucleus-backend \
          --num-nodes 1 \
          --network nucleus-vpc \
          --region us-east1
```
```
gcloud container clusters get-credentials nucleus-backend \
          --region us-east1

```
```
kubectl create deployment hello-server \
          --image=gcr.io/google-samples/hello-app:2.0

```
```
kubectl expose deployment hello-server \
          --type=LoadBalancer \
          --port 8080
```

<br>
<h1 align="center"> Task 3: Set up an HTTP load balancer</h1>
<h3> Using the cloud shell </h3>

<h5> Run the following script. It is to be used while creating an instance template.</h5>

<h4 align="center">Script startup.sh</h4>

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

<h3> Create an instance template</h3>

```
gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh
```

<h3> Create a target pool</h3>

```
gcloud compute target-pools create nginx-pool
```

<h3> Create a managed instance template</h3>

```
gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool 
gcloud compute instances list
```

<h3> Create a firewall rule to allow tcp trafiic at port 80</h3>

```
gcloud compute firewall-rules create www-firewall --allow tcp:80 
gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool
gcloud compute forwarding-rules list
```

<h3> Create a health check </h3>

```
gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80
```

<h3> Create a backend service</h3>

```
gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global
```

<h3> Add the managed instance group to the backend service</h3>

```
gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global
```

<h3> Create a URL map to route the incoming requests to the backend service</h3>

```
gcloud compute url-maps create web-map \
--default-service nginx-backend 
```

<h3> Create a target HTTP proxy to route requests to your URL map</h3>

```
gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map
```

<h3> Create a forwarding rule </h3>

```
gcloud compute forwarding-rules create http-content-rule\
--global \
--target-http-proxy http-lb-proxy \
--ports 80 
gcloud compute forwarding-rules list
```
