# Setup and Configure a Cloud Environment in Google Cloud

_Note: It is highly recommended that you __try__ to complete the objectives atleast twice or thrice before visiting the solutions._

## Task 1: Create development VPC manually

Make sure you create all resources in the us-east1 region and us-east1-b zone.

1. In the Google Cloud Console, navigate to VPC network > VPC networks
2.  Click on Create VPC network.
3.  Enter ```griffin-dev-vpc``` to the ```Name``` field.
    Select ```Custom``` for the ```Subnet creation mode```.
Add ```griffin-dev-wp``` subnet with the following parameters:
    |Field |	Value |
    |:------:|:----:|
    | Name | 	griffin-dev-wp |
    |Region | 	us-east1 |
    |IP address range | 	192.168.16.0/20 |

4. Click + Add subnet and add ```griffin-dev-mgmt``` subnet with the following parameters:

    |Field |	Value |
    |:------:|:----:|
    |Name| 	griffin-dev-mgmt|
    |Region| 	us-east1|
    |IP address range| 	192.168.32.0/20|
5. Click __Create__.



&nbsp;

## Task 2: Create production VPC using Deployment Manager



1. Copy the Deployment Manager configuration files to Cloud Shell using the following command:
```
gsutil cp -r gs://cloud-training/gsp321/dm ~/
```
2. Edit prod-network.yaml configuration file
```
cd dm
edit prod-network.yaml
```

3. Open editor in new window and search ```prod-network.yaml``` file. Replace ```SET_REGION``` with ```us-east1``` in the editor, and then save the change(```Ctrl+s```).

4. Close the tab and go back to the Cloud Shell, use the following command to create the production VPC network with the configuration files:
```
gcloud deployment-manager deployments create griffin-prod --config prod-network.yaml
```
5. Go back to the Cloud Console, navigate to Deployment Manager to confirm the deployment.

&nbsp;

## Task 3: Create bastion host

1. In the Cloud Console, navigate to Compute Engine > VM instances.Click Create.

2. Use the following parameters to create the bastion host:
    |Field |	Value |
    |:------:|:----:|
    |Name| 	```griffin-dev-db``` |
    |Region| 	```us-east1``` |
3. Expand the **Management, security, disks, networking, sole tenancy** section.
4. In the Networking tab, add ```bastion``` to the Network tags.
Click Add network interface, make sure that you set up two Network interfaces,
      -  ```griffin-dev-mgmt```
      -  ```griffin-prod-mgmt```
5. Click __Create__.
6. Navigate to VPC network > Firewall.
7. Click CREATE FIREWALL RULE. Configure the rule with the following parameters:
    |Field |	Value |
    |:------:|:----:|
    Name | 	allow-bastion-dev-ssh |
    Network|	griffin-dev-vpc |
    Targets|	bastion |
    Source IP ranges| 	192.168.32.0/20 |
    Protocols and ports| 	tcp: 22 |
8. Click __CREATE__.
9. Click CREATE FIREWALL RULE again. Configure another rule with the following parameters:
    |Field |	Value |
    |:------:|:----:|
    | Name |	allow-bastion-prod-ssh |
    | Network |	griffin-prod-vpc |
    | Targets | 	bastion |
    | Source IP ranges |	192.168.48.0/20 |
    | Protocols and ports |	tcp: 22 |
10. Click __CREATE__.

## Task 4: Create and configure Cloud SQL Instance


1. In the Cloud Console, navigate to SQL. Click __CREATE INSTANCE__. Choose MySQL.
2. Use the following parameters to create the instance:
    |Field |	Value |
    |:------:|:----:|
    |Name| 	griffin-dev-db |
    |Region| 	us-east1 |
    |Zone| 	us-east1-b |
    | Root password| 	e.g. 12345678 |

3.  Click Create.
4.  Click the ```griffin-dev-db``` in the SQL pane after it has been created. Under Connect to this instance, click on ```Connect using Cloud Shell```.

5. Go back to the Cloud Shell, run:
```
gcloud sql connect griffin-dev-db --user=root --quiet
```
6. Enter the Root password generated in Step 5. In the SQL console, run the following query to create the wordpress database:
```
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```
7. Type ```exit``` to quit the SQL shell.

&nbsp;

## Task 5: Create Kubernetes cluster
Create a 2 node cluster (n1-standard-4) called ```griffin-dev```, in the ```griffin-dev-wp``` subnet, and in the zone ```us-east1-b```.
1. In the Cloud Console, navigate to Kubernetes Engine > Clusters. Click __Create cluster__.

In the Cluster basics tab, configure:

 - Name: ```griffin-dev```
- Zone: ```us-east1-b```

2. In the left pane, click default-pool under **NODE POOLS** and set

- Number of nodes: ```2```

3. Click Nodes Under default-pool, and set

- Machine type: n1-standard-4

4. Go to the Network tab, set

- Network: ```griffin-dev-vpc```
- Node subnet: ```griffin-dev-wp```
5. Click __CREATE__.

## Task 6: Prepare the Kubernetes cluster

1. In the Cloud Shell, use the following command to copy the files for the Kubernetes:
```
gsutil cp -r gs://cloud-training/gsp321/wp-k8s ~/
```
2. Open wp-k8s/wp-env.yaml with the Cloud Shell Editor.
```
cd ~/wp-k8s
edit wp-env.yaml
```
3. Replace **username_goes_here** and p**assword_goes_here** to ```wp_user``` and ```stormwind_rules```, respectively.
4. Save the file change.
5. After the Kubernetes cluster has been created, click on the Connect button.
6.Run the following command to connect the cluster:
```
gcloud container clusters get-credentials griffin-dev --zone=us-east1
```
7. Deploy the configuration to the cluster using:
```
kubectl apply -f wp-env.yaml
```
8. Use the command below to create the key, and then add the key to the Kubernetes environment:
```
cloud iam service-accounts keys create key.json \
--iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
--from-file key.json
```


## Task 7: Create a WordPress deployment

1. Open wp-k8s/wp-deployment.yaml with the Cloud Shell Editor
```
cd ~/wp-k8s
edit wp-deployment.yaml
```
2. Replace **YOUR_SQL_INSTANCE** with griffin-dev-db’s Instance connection name (in our case replace with ```griffin-dev-db```).
3. Save the file changes.

4. Go back to the Cloud Shell, run the following commands:
```
kubectl create -f wp-deployment.yaml
kubectl create -f wp-service.yaml
```
5. Copy the External endpoints of the deployed wordpress service. To check External endpoints type:
```
kubectl get services wordpress
```
It will show you External IP. Save it for next steps.

## Task 8: Enable monitoring
1. Go back to the Cloud Console, and navigate to Monitoring.
2. In the Monitoring console, click Uptime checks in the left pane.
3. Click **CREATE UPTIME CHECK**.

Configure using the following parameters:
|Field |	Value |
|:------:|:----:|
|Title 	|```WordPress Uptime```|
|Check Type| 	HTTP|
|Resource Type| 	URL |
|Hostname| 	YOUR-WORDPRESS_ENDPOINT (```External IP [COPIED PREVIOUSLY]```) |
|Path |	/ |
4. Click on Next > Next > **Save**.

&nbsp;

## Task 9: Provide access for an additional engineer

1. In the Cloud Console, navigate to IAM & Admin > IAM.
2. Click +ADD.
3.  In the Add members to pane, copy and paste the ```Username2``` provided for the lab to the New members field.
4.  In the Role dropdown, select Project > Editor.
5.  Click **SAVE**.
