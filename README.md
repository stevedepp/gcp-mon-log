# gcp-mon-log
basic monitoring and logging for a gcp instance

please click on the video below for sound.

![demo](https://user-images.githubusercontent.com/38410965/111925375-08ea8900-8a7f-11eb-8d7e-32a52d4cc6c6.mp4)


#

Demo Video 9
GCP logging and monitoring an compute instance 
Steve Depp MSDS 434 section 55 

Setup local
- [x] cd into demo directory
- [x] clone my repo
- [x] cd into the repo

➜  gcp-mon-log git:(main) cd Documents/Personal/MSDS/434/week\ 09/
➜  gcp-mon-log git:(main) git clone https://github.com/stevedepp/gcp-mon-log.git
➜  gcp-mon-log git:(main) cd gcp-mon-log

<img width="682" alt="remote Total 14 (delta 5), reused 10 (delta 4), pack-reused 0" src="https://user-images.githubusercontent.com/38410965/113597987-79121680-960a-11eb-88d0-66c0aa6fbf74.png">

#

Setup GCP

- [x] GCP accounts
- [x] GCP projects
- [x] create project
- [x] make project the default project
- [x] confirm project is default
- [x] export project id
- [x] export account id
- [x] associated project with account for billing

➜  gcp-mon-log git:(main) gcloud alpha billing accounts list
➜  gcp-mon-log git:(main) gcloud projects list
➜  gcp-mon-log git:(main) gcloud projects create gcp-mon-log-depp 
➜  gcp-mon-log git:(main) gcloud config set project gcp-mon-log-depp
➜  gcp-mon-log git:(main) gcloud config get-value core/project 
➜  gcp-mon-log git:(main) export PROJECT=$(gcloud config get-value core/project)
➜  gcp-mon-log git:(main) export ACCOUNT=$(gcloud alpha billing accounts list | awk 'NR==2{print $1}')
➜  gcp-mon-log git:(main) gcloud beta billing projects link $PROJECT --billing-account $ACCOUNT 

<img width="682" alt="billinpEnabled true" src="https://user-images.githubusercontent.com/38410965/113598570-52a0ab00-960b-11eb-8329-85e1f363e86d.png">


#

Create a compute instance “larry”

- [x] look at this project’s APIs 
- [x] turn on compute API
- [x] look at available compute zones
- [x] look at available compute instances
- [x] create compute instance “larry”
- [x] make INSTANCE an environment variable for instance-id

➜  gcp-mon-log git:(main) gcloud services list --available | grep 'compute'
➜  gcp-mon-log git:(main) gcloud services enable compute.googleapis.com
➜  gcp-mon-log git:(main) gcloud compute zones list | grep 'us-centr'
➜  gcp-mon-log git:(main) gcloud compute machine-types list --zones us-central1-a | grep 'n1-standard'
➜  gcp-mon-log git:(main) gcloud compute instances create larry --boot-disk-device-name debian-cloud --zone us-central1-a --machine-type n1-standard-1 --tags http-server --scopes cloud-platform
➜  gcp-mon-log git:(main) export INSTANCE=$(gcloud compute instances describe larry --zone us-central1-a --format="value(id)")


#


Create compute firewall

Advantage:  If create a named firewall then can control which instances get which firewalls. If simple check firewall HTTP in console, cannot control future firewalls. 
- [x] create named firewall
- [x] make EXTERNAL_IP an environment variable
- [x] ssh into compute instance

➜  gcp-mon-log git:(main) gcloud compute --project $PROJECT firewall-rules create default-allow-http --target-tags=http-server --action=ALLOW --rules=tcp:80
➜  gcp-mon-log git:(main) export EXTERNAL_IP=$(gcloud compute instances list --project $PROJECT | awk 'NR==2{print $5}')
➜  gcp-mon-log git:(main) gcloud compute ssh larry --zone us-central1-a


#

Install Apache server & confirm traffic

- [x] install apache server 
- [x] confirm serving HTTP traffic

stevedepp@larry:~$ sudo apt-get update
stevedepp@larry:~$sudo apt-get install apache2 php7.0
y
stevedepp@larry:~$ sudo service apache2 restart
stevedepp@larry:~$ curl http://34.123.20.84



#

Install monitoring agents

- [x] curl cloud monitoring agent repo
- [x] add & update monitoring agent
- [x] install the monitoring agent
- [x] start the monitoring agent

stevedepp@larry:~$ curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
stevedepp@larry:~$ sudo bash add-monitoring-agent-repo.sh
stevedepp@larry:~$ sudo apt-get update
stevedepp@larry:~$ sudo apt-get install stackdriver-agent
stevedepp@larry:~$ sudo service stackdriver-agent start


#

Install the logging agent

- [x] curl cloud logging agent repo
- [x] add & update logging agent

stevedepp@larry:~$ curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh
stevedepp@larry:~$ sudo bash add-logging-agent-repo.sh
stevedepp@larry:~$ sudo apt-get update


#

Install the logging agent (cont)

- [x] install the logging agent
- [x] set to structured logs
- [x] start the logging agent

stevedepp@larry:~$ sudo apt-get install google-fluentd
stevedepp@larry:~$ sudo apt-get install google-fluentd-catch-all-config-structured
stevedepp@larry:~$ sudo service google-fluentd start


#

Create a Monitoring Workspace 

This can only be done from console.


#

Create a Monitoring Workspace 



#


Create a Monitoring Workspace 



#

Create a Monitoring Workspace 


#

Create an email notification

- [x] create an email notification
- [x] make EMAIL_NOTIFICATION an environment variable

➜  gcp-mon-log git:(main) gcloud beta monitoring channels create --channel-content-from-file email-notification.json
➜  gcp-mon-log git:(main) gcloud beta monitoring channels list
➜  gcp-mon-log git:(main) gcloud beta monitoring channels --format json list > notifs.json
➜  gcp-mon-log git:(main) export EMAIL_NOTIFICATION=( $(jq -r '.[].name' notifs.json) )
➜  gcp-mon-log git:(main) echo $EMAIL_NOTIFICATION



#

Create an uptime check

- [x] amend uptime check template for the new notification id
- [x] curl the API to create an uptime check

➜  gcp-mon-log git:(main) sed 's|variable|'$INSTANCE'|g' uptime_check_template.json > uptime_check.json
➜  gcp-mon-log git:(main) curl -X POST \
-H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
-H "Content-Type: application/json; charset=utf-8" \
https://monitoring.googleapis.com/v3/projects/$PROJECT/uptimeCheckConfigs -d @uptime_check.json 



#

Create 2 monitoring policies

- [x] amend uptime policy template for the new notification id
- [x] create uptime policy
- [x] amend network packets check template for the new notification id
- [x] create network packets policy
- [x] make POLICY_1 and POLICY_2 environment variables

➜  gcp-mon-log git:(main) sed 's|variable|'$EMAIL_NOTIFICATION'|g' uptime_policy_template.json > uptime_policy.json
➜  gcp-mon-log git:(main) gcloud alpha monitoring policies create --policy-from-file uptime_policy.json
➜  gcp-mon-log git:(main) sed 's|variable|'$EMAIL_NOTIFICATION'|g' rising_network_packets_policy_template.json > rising_network_packets_policy.json
➜  gcp-mon-log git:(main) gcloud alpha monitoring policies create --policy-from-file rising_network_packets_policy.json
➜  gcp-mon-log git:(main) export POLICY_1=$(gcloud alpha monitoring policies list --format='value(name)' | awk 'NR==1')
➜  gcp-mon-log git:(main) export POLICY_2=$(gcloud alpha monitoring policies list --format='value(name)' | awk 'NR==2')
➜  gcp-mon-log git:(main) gcloud alpha monitoring policies list



#

Test uptime check & email notification 

- [x] stop the instance “larry”
- [x] receive email notification

➜  gcp-mon-log git:(main) ✗ gcloud compute instances stop $INSTANCE --zone us-central1-a


#

Test curing uptime check & email notification 

- [x] start the instance “larry”
- [x] receive email notification 30 minutes later

➜  gcp-mon-log git:(main) ✗ gcloud compute instances start $INSTANCE --zone us-central1-a


#


Make a dashboard

- [x] create a dashboard
- [x] make DASH an environment variable

➜  gcp-mon-log git:(main) gcloud monitoring dashboards create --config-from-file dash.json
➜  gcp-mon-log git:(main) gcloud monitoring dashboards list
➜  gcp-mon-log git:(main) export DASH=$(gcloud monitoring dashboards list --format="value(name)")
➜  gcp-mon-log git:(main) echo $DASH



#


Look at dashboard


#

Tear it down
delete dashboard DASH
delete policy POLICY_1
delete policy POLICY_2
delete notification EMAIL_NOTIFICATION
delete instance INSTANCE
➜  gcp-mon-log git:(main) gcloud monitoring dashboards delete $DASH
➜  gcp-mon-log git:(main) gcloud alpha monitoring policies delete $POLICY_1
➜  gcp-mon-log git:(main) gcloud alpha monitoring policies delete $POLICY_2
➜  gcp-mon-log git:(main) gcloud beta monitoring channels delete $EMAIL_NOTIFICATION
➜  gcp-mon-log git:(main) gcloud compute instances delete $INSTANCE --zone us-central1-a
