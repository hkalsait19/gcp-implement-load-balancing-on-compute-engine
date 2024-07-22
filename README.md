# gcp-implement-load-balancing-on-compute-engine
GCP - Implement Load Balancing on Compute Engine

Refer [here](https://github.com/jjk-dev/qwiklabs-challenge-lab-gcp/blob/main/Challenge-Lab/GSP313_Create-and-Manage-Cloud-Resources-Challenge-Lab.md)

# Getting Started: Create and Manage Cloud Resources: [Challenge Lab](https://www.qwiklabs.com/focuses/10258?parent=catalog)

## Challenge scenario


### Task 1: Create a project jumphost instance
You will use this instance to perform maintenance for the project.

Requirements:
- Name the instance **nucleus-jumphost.**
- Use an **e2-micro** machine type.
- Use the default image type (Debian Linux).

Create VM instance.
```
gcloud compute instances create nucleus-jumphost-603 \
  --machine-type e2-micro \
  --zone europe-west1-b
```

### Task 2: Set up an HTTP load balancer
You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant. Create an HTTP load balancer with a managed instance group of **2 nginx web servers.** Use the following code to configure the web servers; the team will replace this with their own configuration later.

Run the configuration of the web servers.

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

Create multiple instance template.
```
gcloud compute instance-templates create nucleus-nginx-template \
    --machine-type e2-medium \
    --metadata-from-file startup-script=startup.sh
```

Create a target pool.
```
gcloud compute target-pools create nginx-pool 
```

Create a managed instance group.
```
gcloud compute instance-groups managed create nucleus-webserver-nginx-group \
  --base-instance-name nucleus-webserver \
  --size 2 \
  --template nucleus-nginx-template \
  --target-pool nginx-pool \
  --zone europe-west1-b \
  
gcloud compute instances list
```

Create a firewall rule to allow traffic.
```
gcloud compute firewall-rules create accept-tcp-rule-475 --allow tcp:80

gcloud compute forwarding-rules create nginx-lb \
  --region europe-west1 \
  --ports=80 \
  --target-pool nginx-pool
  
gcloud compute forwarding-rules list
```

Create a health check.
```
gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
  set-named-ports nucleus-webserver-nginx-group \
  --named-ports http:80
```

Create a backend service, and attach the managed instance group.
```
gcloud compute backend-services create nginx-backend \
  --protocol HTTP --http-health-checks http-basic-check \
  --global
  
gcloud compute backend-services add-backend nginx-backend \
  --instance-group nucleus-webserver-nginx-group \
  --instance-group-zone europe-west1-b \
  --global
```

Create a URL map, and target the HTTP proxy to route requests to your URL map.
```
gcloud compute url-maps create web-map \
  --default-service nginx-backend
  
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map web-map
```

Create a forwarding rule.
```
gcloud compute forwarding-rules create http-rule \
  --global \
  --target-http-proxy http-lb-proxy \
  --ports 80
  
gcloud compute forwarding-rules list
```


## Congratulations!
<img src="https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Create-and-Manage-Cloud-Resources.png" height="150" />
https://www.credly.com/badges/92e7177d-25d0-448f-96d2-c45148754964/public_url

### Finish your Quest
This self-paced lab is part of the [Create and Manage Cloud Resources](https://google.qwiklabs.com) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above to recognize your achievement. You can make your badge (or badges) public and link to them in your online resume or social media account. Enroll in a Quest and get immediate completion credit for taking this lab. [See other available Qwiklabs Quests.](https://google.qwiklabs.com/catalog)

This skill badge quest is part of Google’s [Associate Cloud Engineer](https://cloud.google.com/certification/cloud-engineer) and [Professional Cloud Architect](https://cloud.google.com/certification/cloud-architect) learning paths. Continue your learning journey by enrolling in the [Perform Foundational Infrastructure Tasks in Google Cloud](https://google.qwiklabs.com/quests/118) skill badge quest.
