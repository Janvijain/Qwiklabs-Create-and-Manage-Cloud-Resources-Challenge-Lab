# Qwiklabs-Create-and-Manage-Cloud-Resources-Challenge-Lab
Step by Step guide to solve this challenge.
Can refer my YouTube video for the same : https://youtu.be/VKyxcFlpZ3s

# Task 1
Navigation menu->Compute Engine->VM instances->create->Name=nucleus-jumphost 
Machine configuration->series->N1->machine type->f1-micro
Boot disk->Debian Linux

# Task 2
Open Cloud Shell

-> gcloud auth list

-> gcloud config list project

-> gcloud config set compute/zone us-east1-b

-> gcloud container clusters create nucleus-jumphost-webserver1

-> gcloud container clusters get-credentials nuleus-jumphost-webserver1

-> kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0

-> kubectl expose deployment hello-app --type=LoadBalancer --port 8080

-> kubectl get service

# Task 3
Type in the Cloud Shell

cat << EOF > startup.sh      
#! /bin/bash             
apt-get update               
apt-get install -y nginx           
service nginx start             
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html              
EOF          

Create an instance template.

-> gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh

Create a target pool.

-> gcloud compute target-pools create nginx-pool

n/20->us-east1                    \\check for us-east1 zone on your screen and select number accordingly

Create a managed instance group.

-> gcloud compute instance-groups managed create nginx-group \     
--base-instance-name nginx \          
--size 2 \          
--template nginx-template \          
--target-pool nginx-pool            

-> gcloud compute instances list

Create a firewall rule to allow traffic (80/tcp).

-> gcloud compute firewall-rules create www-firewall --allow tcp:80

-> gcloud compute forwarding-rules create nginx-lb \         
--region us-east1 \       
--ports=80 \      
--target-pool nginx-pool      

-> gcloud compute forwarding-rules list

Create a health check.

-> gcloud compute http-health-checks create http-basic-check

Create a backend service, and attach the managed instance group.

-> gcloud compute instance-groups managed set-named-ports nginx-group --named-ports http:80

-> gcloud compute backend-services create nginx-backend \             
--protocol HTTP --http-health-checks http-basic-check --global

-> gcloud compute backend-services add-backend nginx-backend \                    
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global

Create a URL map, and target the HTTP proxy to route requests to your URL map.

-> gcloud compute url-maps create web-map --default-service nginx-backend

-> gcloud compute target-http-proxies create http-lb-proxy --url-map web-map

Create a forwarding rule.

-> gcloud compute forwarding-rules create http-content-rule \              
--global \                  
--target-http-proxy http-lb-proxy \                 
--ports 80           

-> gcloud compute forwarding-rules list
