# Google-Admin-Audit-Log-with-Elastic-Search
#### Installing ELK Stack (Elasticsearch, Logstash, and Kibana) on Ubuntu 20.04 with Security enabled and Google Admin Audit Log Setup ####
##

## Prerequisites ##

* A Linux system Ubuntu 20.04 or 18.04
* Access to a terminal window/command line (Search > Terminal)
* A user account with sudo or root privileges
* Java version 8 or 11 (required for Logstash)

## Step 1: Java ##

1. Check Java Version
```
java -version
```
2. If you don’t have Java 8 installed, install it by opening a terminal window and entering the following:
```
sudo apt-get install openjdk-8-jdk

```
## Step 2: Install Nginx ##
```
sudo apt-get install nginx
```
## Elastic Repository ##
* Add Elastic Repository
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

* Install Transport-https
```
sudo apt-get install apt-transport-https
```
* Add the Elastic repository to your systems repository list:

```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee –a /etc/apt/sources.list.d/elastic-7.x.list

```
##  Step 3: Elasticsearch Install ##

* Prior to installing make sure your up to date
```
sudo apt-get update

```
* Install Elasticsearch
```
sudo apt-get install elasticsearch

```

## Configure Elasticsearch

```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
* Modify with the following:
```
#network.host: 192.168.0.1
#http.port: 9200

```
* with the following, make sure to remove the # in front.
```
network.host: localhost
http.port: 9200
#Discovery Section
discovery.type: single-node

```
## JVM Heap Size modification ##

* The default heap size is set at 1GB, lets bump those numbers some.

```
sudo nano /etc/elasticsearch/jvm.options

```
* Modify the -Xms512m & -Xmx512m

```
-Xms4G
-Xmx4G

```

## Elasticsearch
* Start Elasticsearch
```
sudo systemctl start elasticsearch.service
```

* Set Elasticsearch to start on boot
```
sudo systemctl enable elasticsearch.service
```
* Test Elasticsearch
```
curl -X GET "localhost:9200"

```
* This is what you should get
```
root@neovm1:/home/neo# curl -X GET "localhost:9200"
{
  "name" : "neovm1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "iugauTIGS3WEGbQ3uUJTfw",
  "version" : {
    "number" : "7.15.0",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "79d65f6e357953a5b3cbcc5e2c7c21073d89aa29",
    "build_date" : "2021-09-16T03:05:29.143308416Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```
## Step 4: Install Kibana ##

```
sudo apt-get install kibana
```

* Configure Kibana
```
sudo nano /etc/kibana/kibana.yml
```
* Delete the # at the beginning of these lines
```
#server.port: 5601
#server.host: "your-hostname"
#elasticsearch.hosts: ["http://localhost:9200"]
```
* How it should look after
```
server.port: 5601
server.host: "You Servers IP"
elasticsearch.hosts: ["http://localhost:9200"]
```
## Start and Enable Kibana

* Start the Kibana service:
```
sudo systemctl start kibana
```
* Then make sure you enable so it runs on boot
```
sudo systemctl enable kibana

```
## Allow Traffic ##
* Allow Traffic on Port 5601
```
sudo ufw allow 5601/tcp
```
* Test Kibana in your browser
```
http://localhost:5601

```
* Kibana Works.
![image](https://user-images.githubusercontent.com/33500545/148628629-97f91591-1445-4dcd-900b-7593b8ec4a53.png)


## Step 5: Install Logstash ##

* Install Logstash:
```
sudo apt-get install logstash

```
* Start the Logstash Service:
```
sudo systemctl start logstash

```
* Enable Logstash on boot:
```
sudo systemctl enable logstash

```
* Check the status of the Service
```
sudo systemctl status logstash

```
* Results
```
logstash.service - logstash
    Loaded: loaded (/etc/systemd/system/logstash.service; enabled; vendor preset: enabled)
    Active: active (running) since Thu 2021-09-30 16:37:33 UTC; 14s ago
  Main PID: 11219 (java)
     Tasks: 16 (limit: 4583)
    Memory: 508.5M
    CGroup: /system.slice/logstash.service
            └─11219 /usr/share/logstash/jdk/bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75
```

## Configure Logstash ##

* Logstash is a highly customizable part of the ELK stack. Once installed, configure its INPUT, FILTERS, and OUTPUT pipelines according to your own individual use case.

* All custom Logstash configuration files are stored in /etc/logstash/conf.d/.

## Step 6: Install Filebeat ##

* Install Filebeat
```
sudo apt-get install filebeat

```
* Edit the filebeat.yml configure
```
sudo nano /etc/filebeat/filebeat.yml

```
* Under the Elasticsearch out put comment out the section: Ex:
```
# output.elasticsearch:
   # Array of hosts to connect to.
   # hosts: ["localhost:9200"]

```
* Uncomment the Logstash section:
* Before
```
# output.logstash
     # hosts: ["localhost:5044"]
```
* After
```
 output.logstash
    hosts: ["localhost:5044"]
```

* Now we are going to enable the filebeat but instead of using system as the default we are going to just do the Google Workspace

```
sudo filebeat modules enable google_workspace

```
* Time to setup the index template:
```
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'

```
## Step 6 Detour - Google Workspace Setup before you start Filebeat

* Link to Filebeat setup document-
https://www.elastic.co/guide/en/beats/filebeat/7.15/filebeat-module-google_workspace.html

* Modifications after you get it setup.

```
sudo nano /etc/filebeat/modules.d/google_workspace.yml

```
* In the directions you will have modified the true/false, credentials, and deligated_account.
* Something the directions don't show is you need to add "filebeat.modules:" to the top of the file.
* Before google_workspace.yml
```
# Module: google_workspace
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-google_workspace>

- module: google_workspace
  saml:
    enabled: true

```
* Add filebeat.modules: (Don't know why its missing but you will get errors)
```
filebeat.modules:
# Module: google_workspace
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-google_workspace>

- module: google_workspace
  saml:
    enabled: true

```

* Start Filebeat
```
sudo systemctl start filebeat

```
* Enable filebeat
```
sudo systemctl enable filebeat

```
* Verify the service is working
```
curl -XGET http://localhost:9200/_cat/indices?v

```
* Results should be simlar
```
Synchronizing state of filebeat.service with SysV service script with /lib/syste                                md/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable filebeat
Created symlink /etc/systemd/system/multi-user.target.wants/filebeat.service → /                                lib/systemd/system/filebeat.service.
root@neovm1:/etc/filebeat/modules.d# curl -XGET http://localhost:9200/_cat/indic                                es?v
health status index                             uuid                   pri rep d                                ocs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases                  jGx_PuCPQsqcL0juRBbuDQ   1   0                                         43            0     41.1mb         41.1mb
green  open   .kibana_7.15.0_001                PkHr6Ls0RkWp1EBG-i8nYg   1   0                                         21            1      2.3mb          2.3mb
green  open   .apm-custom-link                  qZM7kJG6T3mecC2nnSOTdg   1   0                                          0            0       208b           208b
green  open   .kibana-event-log-7.15.0-000001   cvAgfOzJRVKD8MLKFKckCw   1   0                                          1            0        6kb            6kb
green  open   .apm-agent-configuration          ctbqhdnWQGe0SuquEyVRVQ   1   0                                          0            0       208b           208b
yellow open   filebeat-7.15.0-2021.09.30-000001 GqiA6YI4RyyPjAVkT_aszQ   1   1                                          0            0       208b           208b
green  open   .kibana_task_manager_7.15.0_001   nAo234n0QGyuQ8ofvWsxtA   1   0                                            229.4kb        229.4kb


```
## Step 7 : Lets add some security before trying to use the Elasticsearch Integration module
* Follow this guide: https://www.elastic.co/guide/en/elasticsearch/reference/current/security-minimal-setup.html
* Note the paths from the guide are not the same as on the system.
```
Paths are /urs/share/elasticsearch/bin
```

## Step 8 : Elasticsearch-Agent Installation
* Directions - https://www.elastic.co/guide/en/fleet/current/run-elastic-agent-standalone.html

## Step 9 : Elastic Search Integration
* After you login to elasticsearch go to Add Elastic Agent Integration
* Search for google_workspace
* Click Add Google google Workspace
* Integration Settings
* 1. Click on Advanced Settings and change the Namespace to GoogleAudit
* 2. JWT File: Point to the location of your credentials.json
* 3. You can turn off the ones you don't want.
* 4. After your doing click on data streams and watch for it to increase


