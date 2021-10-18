<h1 align="center">Task 1: Create the production environment</h1>
<h3>Using the cloud console</h3>
Navigation menu -> Deployment Manager -> Verify that kraken-jumphost has been provisioned.

Navigation menu -> Compute Engine -> VM instances

Click SSH button against kraken-jumphost instance

In the SSH console, run

cd /work/dm

sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network --config=prod-network.yaml

<h3>Create a Kubernetes cluster in the new network</h3>
Navigation menu -> Kubernetes Engine -> Cluster -> Create cluster

* Name: kraken-prod
* Nodes: 2
* Network tab: kraken-prod-vpc

<h3>Create the frontend and backend deployments and services</h3>

Back in the SSH console, run

cd /work/k8s

kubectl create -f deployment-prod-backend.yaml

kubectl create -f deployment-prod-frontend.yaml

kubectl get pods

kubectl create -f service-prod-backend.yaml

kubectl create -f service-prod-frontend.yaml

kubectl get services

<h1 align="center">Task 2: Setup the Admin instance</h1>

Navigation menu -> Compute Engine -> VM instances -> Create instance

* Name: kraken-admin
* Zone: us-east1-b

In the networking tab, select network interfaces
* kraken-mgmt-subnet 
* kraken-prod-subnet 

Once the instance is created, go to Details and copy the instance ID.

<h3>Create a Monitoring workspace</h3>

Navigation menu -> Monitoring -> Alerting -> Create Policy -> Add condition

* Resource Type: GCE VM Instance
* Metric: CPU Utilization(compute.googleapis.com/instance/cpu/utilization)
* Filter: Insatnce ID (paste the copied insatnce ID of kraken-admin from the previous step)
* Threshold: 0.5 (for 1 minute)

Click Add.

<h1 align="center">Task 3: Verify the Spinnaker deployment</h1>

<h3>Connect the Spinnaker console</h3>
<h3>Using the cloud shell</h3>

gcloud config set compute/zone us-east1-b

gcloud container clusters get-credentials spinnaker-tutorial

DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" -o jsonpath="{.items[0].metadata.name}")

kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &

Go to WEB PREVIEW icon -> Preview on PORT 8080

Click applications -> Sample

Open the pipeline 

Manually run the pipeline(if it is not running)

Approve the deployment to production

Check the production frontend endpoint(use http)

<h3>Using the cloud shell</h3>

gcloud config set compute/zone us-east1-b

gcloud source repos clone sample-app

cd sample-app

touch a

git config --global user.email "$(gcloud config get-value core/account)"

git config --global user.name "$(gcloud config get-value core/account)"

git commit -a -m "making a change"

git tag v1.0.1

git push --tags