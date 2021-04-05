## GCP logging and monitoring an compute instance 

please click on the video below for sound.

![demo](https://user-images.githubusercontent.com/38410965/111925375-08ea8900-8a7f-11eb-8d7e-32a52d4cc6c6.mp4)


#

> Hello everyone and thank you for watching my video of setting up monitoring and logging for a GCP instance.  First step is to set up locally.  

**Demo Video 9**

### GCP logging and monitoring an compute instance 

**Steve Depp MSDS 434 section 55** 

### Setup local**
- [x] cd into demo directory
- [x] clone my repo
- [x] cd into the repo

➜  gcp-mon-log git:(main) cd Documents/Personal/MSDS/434/week\ 09/
➜  gcp-mon-log git:(main) git clone https://github.com/stevedepp/gcp-mon-log.git
➜  gcp-mon-log git:(main) cd gcp-mon-log

<img width="682" alt="remote Total 14 (delta 5), reused 10 (delta 4), pack-reused 0" src="https://user-images.githubusercontent.com/38410965/113597987-79121680-960a-11eb-88d0-66c0aa6fbf74.png">

#

> Then, set up a GCP project and link it to your billing account.  I will be setting up environment variables as we go to facilitate tear down later on.

### Setup GCP

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

> We then create a compute instance, named 'larry' in this case. 

### Create a compute instance “larry”

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

<img width="682" alt="list --available grep" src="https://user-images.githubusercontent.com/38410965/113598780-a612f900-960b-11eb-8e6d-b3e839e5e11d.png">

#

> Then, create a firewall to allow HTTP access.   It is worth nothing that naming a firewall has distinct advantages.  GCP has some odd features when it comes to ticking the box for HTTPS or HTTP on the command line. 

### Create compute firewall

Advantage:  If create a named firewall then can control which instances get which firewalls. If simple check firewall HTTP in console, cannot control future firewalls. 
- [x] create named firewall
- [x] make EXTERNAL_IP an environment variable
- [x] ssh into compute instance

➜  gcp-mon-log git:(main) gcloud compute --project $PROJECT firewall-rules create default-allow-http --target-tags=http-server --action=ALLOW --rules=tcp:80
➜  gcp-mon-log git:(main) export EXTERNAL_IP=$(gcloud compute instances list --project $PROJECT | awk 'NR==2{print $5}')
➜  gcp-mon-log git:(main) gcloud compute ssh larry --zone us-central1-a

<img width="682" alt="waiting for SSH key to propagate" src="https://user-images.githubusercontent.com/38410965/113598811-b4f9ab80-960b-11eb-8308-060409db6b02.png">

#

> Next, we Install and confirm traffic on the server, ...

### Install Apache server & confirm traffic

- [x] install apache server 
- [x] confirm serving HTTP traffic

stevedepp@larry:~$ sudo apt-get update
stevedepp@larry:~$sudo apt-get install apache2 php7.0
y
stevedepp@larry:~$ sudo service apache2 restart
stevedepp@larry:~$ curl http://34.123.20.84

<img width="682" alt="the exact distribution terms for each program are described in the" src="https://user-images.githubusercontent.com/38410965/113598868-c642b800-960b-11eb-8bf4-22dec37c87a7.png">

<img width="952" alt="Apache2 Debian Default Page" src="https://user-images.githubusercontent.com/38410965/113598877-c8a51200-960b-11eb-8478-8c07c8897c9b.png">

#

> ... instal monitoring agents, ...

### Install monitoring agents

- [x] curl cloud monitoring agent repo
- [x] add & update monitoring agent
- [x] install the monitoring agent
- [x] start the monitoring agent

stevedepp@larry:~$ curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
stevedepp@larry:~$ sudo bash add-monitoring-agent-repo.sh
stevedepp@larry:~$ sudo apt-get update
stevedepp@larry:~$ sudo apt-get install stackdriver-agent
stevedepp@larry:~$ sudo service stackdriver-agent start

<img width="682" alt="Hit3 httpdeb debian orgdebian buster InRelease" src="https://user-images.githubusercontent.com/38410965/113598918-d9558800-960b-11eb-92a6-7ecc9a18b7ae.png">

#

> ... instal a logging agent, ...

### Install the logging agent

- [x] curl cloud logging agent repo
- [x] add & update logging agent

stevedepp@larry:~$ curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh
stevedepp@larry:~$ sudo bash add-logging-agent-repo.sh
stevedepp@larry:~$ sudo apt-get update

<img width="682" alt="-cloud-packages-archive-keyring" src="https://user-images.githubusercontent.com/38410965/113598954-e4a8b380-960b-11eb-8720-3010bfa680be.png">

#

### Install the logging agent (cont)

- [x] install the logging agent
- [x] set to structured logs
- [x] start the logging agent

stevedepp@larry:~$ sudo apt-get install google-fluentd
stevedepp@larry:~$ sudo apt-get install google-fluentd-catch-all-config-structured
stevedepp@larry:~$ sudo service google-fluentd start

<img width="682" alt="gcp-mon-log" src="https://user-images.githubusercontent.com/38410965/113599159-394c2e80-960c-11eb-99d8-1e2ce6bfa018.png">

#

> Create a monitoring workspace.  This is the only step that cannot be done at the command line.

### Create a Monitoring Workspace 

This can only be done from console.

![Google Cloud](https://user-images.githubusercontent.com/38410965/113599195-4701b400-960c-11eb-9269-567ffcbaece2.png)

<img width="952" alt="© Google Cloud Platform sta" src="https://user-images.githubusercontent.com/38410965/113599211-4bc66800-960c-11eb-8019-6b226ab126be.png">

#

> A work space sits along side a project to perfrom alerts notifcations etc.

### Create a Monitoring Workspace 

![Google Cloud](https://user-images.githubusercontent.com/38410965/113599672-eb83f600-960c-11eb-81a2-57bcfe7cfd90.png)

<img width="952" alt="Building your Workspace" src="https://user-images.githubusercontent.com/38410965/113599685-f0e14080-960c-11eb-97b3-c35189cfc7a3.png">

<img width="783" alt="Note You must use the Google Cloud Console to create a Workspace" src="https://user-images.githubusercontent.com/38410965/113599695-f5a5f480-960c-11eb-8635-611b4befb1b3.png">


#

### Create a Monitoring Workspace 

![Google Cloud](https://user-images.githubusercontent.com/38410965/113599711-fdfe2f80-960c-11eb-952d-70128eff02a7.png)

<img width="952" alt="Finishing Workspace creation" src="https://user-images.githubusercontent.com/38410965/113599725-02c2e380-960d-11eb-8ddf-ff435914a599.png">

<img width="783" alt="Note You must use the Google Cloud Console to create a Workspace" src="https://user-images.githubusercontent.com/38410965/113599740-08b8c480-960d-11eb-874e-d8cf7282cde8.png">

#

### Create a Monitoring Workspace 

<img width="952" alt="Welcome to Monitoring!" src="https://user-images.githubusercontent.com/38410965/113599758-0fdfd280-960d-11eb-8946-be9a3d579feb.png">

#

> Then, configure an email notifciton service.

### Create an email notification

- [x] create an email notification
- [x] make EMAIL_NOTIFICATION an environment variable

➜  gcp-mon-log git:(main) gcloud beta monitoring channels create --channel-content-from-file email-notification.json
➜  gcp-mon-log git:(main) gcloud beta monitoring channels list
➜  gcp-mon-log git:(main) gcloud beta monitoring channels --format json list > notifs.json
➜  gcp-mon-log git:(main) export EMAIL_NOTIFICATION=( $(jq -r '.[].name' notifs.json) )
➜  gcp-mon-log git:(main) echo $EMAIL_NOTIFICATION

<img width="682" alt="gcp-mon-log" src="https://user-images.githubusercontent.com/38410965/113599776-166e4a00-960d-11eb-8d5a-ab033151cbd0.png">

#

> An uptime check can only be configured via the API or console, ie not via gcloud CLT.  

### Create an uptime check

- [x] amend uptime check template for the new notification id
- [x] curl the API to create an uptime check

➜  gcp-mon-log git:(main) sed 's|variable|'$INSTANCE'|g' uptime_check_template.json > uptime_check.json
➜  gcp-mon-log git:(main) curl -X POST \
-H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
-H "Content-Type: application/json; charset=utf-8" \
https://monitoring.googleapis.com/v3/projects/$PROJECT/uptimeCheckConfigs -d @uptime_check.json 

<img width="682" alt="curl - POST" src="https://user-images.githubusercontent.com/38410965/113599786-1cfcc180-960d-11eb-9d69-11e1733ce994.png">

#

> Next, I make 2 monitoring policies: one that checks on network packets and the other checks on whether the instanced is running or not.

### Create 2 monitoring policies

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

<img width="682" alt="Created alert" src="https://user-images.githubusercontent.com/38410965/113599813-26862980-960d-11eb-9413-dd0060c2b4ec.png">

<img width="682" alt="gcp-mon-log" src="https://user-images.githubusercontent.com/38410965/113599834-2d14a100-960d-11eb-81ce-46fd5ff5a6ab.png">

#

> The latter is an uptime check. And the policy is configured so that if the instance is not running for 60 seconds, I get an email.  So, here I shut the instance down at the command line, and receive an email about 2 minutes later

### Test uptime check & email notification 

- [x] stop the instance “larry”
- [x] receive email notification

➜  gcp-mon-log git:(main) ✗ gcloud compute instances stop $INSTANCE --zone us-central1-a

<img width="682" alt="gcp-mon-log" src="https://user-images.githubusercontent.com/38410965/113599861-3998f980-960d-11eb-8e27-f0f21cb18370.png">

<img width="842" alt="Uptime not for 60 seconds" src="https://user-images.githubusercontent.com/38410965/113599879-3e5dad80-960d-11eb-9fcc-f4cf2f158870.png">

#

> And here, I start it back up again, and get an email about 30 minutes later.

### Test curing uptime check & email notification 

- [x] start the instance “larry”
- [x] receive email notification 30 minutes later

➜  gcp-mon-log git:(main) ✗ gcloud compute instances start $INSTANCE --zone us-central1-a

<img width="682" alt="concomputeviprojectsgcp-mon-leg-deppzon" src="https://user-images.githubusercontent.com/38410965/113599910-49b0d900-960d-11eb-856c-ad54cabeb9b2.png">

<img width="842" alt="Uptime not for 60 seconds" src="https://user-images.githubusercontent.com/38410965/113599920-4d446000-960d-11eb-80a5-20b582bb2c1e.png">

#

> We can make a dashboard with a line of code.  The rest here is just an echo of the configuration.

### Make a dashboard

- [x] create a dashboard
- [x] make DASH an environment variable

➜  gcp-mon-log git:(main) gcloud monitoring dashboards create --config-from-file dash.json
➜  gcp-mon-log git:(main) gcloud monitoring dashboards list
➜  gcp-mon-log git:(main) export DASH=$(gcloud monitoring dashboards list --format="value(name)")
➜  gcp-mon-log git:(main) echo $DASH

<img width="682" alt="80x  gcp-mon-log - stevedepp@Steves-MBP-2 -  9gcp-mon-log - -zsh -" src="https://user-images.githubusercontent.com/38410965/113599945-546b6e00-960d-11eb-9b77-489836869290.png">

#

> So, you could pass someone that yaml or json file containing the config if they like yours.

### Look at dashboard

<img width="952" alt="E Google Cloud Platform" src="https://user-images.githubusercontent.com/38410965/113599993-6220f380-960d-11eb-9b41-25a2cfb89c2e.png">

<img width="952" alt="E Google Cloud Platform" src="https://user-images.githubusercontent.com/38410965/113600013-69e09800-960d-11eb-8fe9-bb6c55e99eba.png">

#

> Finally, tear it all down.  Thank you for watching. 

### Tear it down
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

<img width="682" alt="attached disks configured" src="https://user-images.githubusercontent.com/38410965/113600063-7a910e00-960d-11eb-8ab9-bc0dcc37d9f7.png">
