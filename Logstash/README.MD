# Install and configure Logstash on AWS
Logstash is an open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to your favorite “stash.” (Ours is Elasticsearch, naturally.)

Logstash has three main functions
 - **INPUTS**: Ingest Data of All Shapes, Sizes, and Sources
 - **FILTERS**: Parse & Transform Your Data On the Fly
 - **OUTPUTS**: Choose Your Stash, Transport Your Data

 Logstash Features:
 - **PLUG & PLAY**: Accelerated Time to Insight with the Elastic Stack
 - **EXTENSIBILITY**: Create and Configure Your Pipeline, Your Way
 - **DURABILITY & SECURITY**: Trust in a Pipeline Built to Deliver
 - **MONITORING**: Have Full Visibility into Your Deployments
 - **MANAGEMENT & ORCHESTRATION**: Centrally Manage Deployments With a Single UI

Follow this article on **[Youtube](https://youtu.be/YasrCKykAKo)**
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/elk.png)
## Install Logstash

### Pre-Requisites
 - An ES Cluster accessible through its FQDN - [How to configure ES in AWS on EC2](https://github.com/miztiik/elk-stack/tree/master/ElasticSearch)
- Logstash requires Java 8. Java 9 is not supported. 

```sh
java -version
# Install Java v8 (if it is lesser than v8)
sudo yum -y install java-1.8.0-openjdk
sudo yum -y remove java-1.7.0-openjdk
java -version
echo $JAVA_HOME
# export PATH=$JAVA_HOME/bin:$PATH 
```

### Installing from the RPM repository
We are going to install Logstash in Amazon Linux 2, you should be able to adapt this procedure for say RHEL/CentOS/Fedora. 
- Accept the GPG key of the repo
- Create repo URL
- Install from RPM

#### Accept the GPG key
```sh
# Accept the ElasticSearch PGP Key
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

#### Create Repo
Create a file called `logstash.repo` in the `/etc/yum.repos.d/` directory. The below repo is for Logstash version 6.x
```sh
echo '[logstash-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
' | sudo tee /etc/yum.repos.d/elasticsearch.repo
```

#### Install from RPM
```sh
sudo yum -y install logstash

# For manual installations visit
# https://www.elastic.co/guide/en/logstash/6.3/installing-logstash.html
```

## Configure Logstash
The configuration above tells Logstash to collect all files with `.log` extention in `/var/log`, `/var/log/messages` and `/var/log/syslog`. And we will create a filter to prevent Elasticsearch to store logs in the message field and simplify the analysis.

```sh
# File Inputs & Outputs
echo 'input {
  file {
    path => [ "/var/log/*.log", "/var/log/messages", "/var/log/syslog" ]
    type => "syslog"
  }
}
output {
    elasticsearch {
        hosts => ["ec2-18-197-147-233.eu-central-1.compute.amazonaws.com:9200"]
    }
    #stdout { codec => rubydebug }
}' | sudo tee /etc/logstash/conf.d/logstash-syslog.conf


# Filter configuration
echo 'filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}' | sudo tee /etc/logstash/conf.d/logstash-syslog-filter.conf
```
#### Logstash Permissions
Logstash needs permissions to access the log files, Add `logstash` group to have read permissions on the log files that you like to be shipped
```sh
chgrp logstash /var/log/*.log /var/log/messages
chmod g+r /var/log/*.log /var/log/messages
```
If they aren't provided you will receive errors like this,
```pre
[2018-07-20T23:53:30,532][WARN ][logstash.inputs.file ] failed to open /var/log/messages: Permission denied - /var/log/messages
```

#### Configure Logstash to run as service
Use the `chkconfig` command to configure Logstash to start automatically when the system boots up
```sh
/usr/share/logstash/bin/system-install /etc/logstash/startup.options sysv
sudo chkconfig --add logstash
```
## Start / Stop Logstash
Logstash can be started and stopped using the `service` command
```sh
# To Start/Stop Logstash 
sudo -i service logstash start
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/ &

# To Stop Logstash 
sudo -i service logstash stop

# Checking Logstash logs
tail -f /var/log/logstash-stdout.log
```

### Searching Logs sent by Logstash in ElasticSearch
Check the list of documents under `Browser`
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/ELK-Indices-00.png)

#### Search for `event` or `string`
Here we are searching for `DCHP` events,
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/ELK-Search-00.png)

Here we are searching for `permissions` string,
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/ELK-Search-01.png)

## Next Steps
 - Visualizing Logs in Kibana - [Follow Here](https://github.com/miztiik/elk-stack/tree/master/Kibana) - [Youtube](https://youtu.be/TTwI8gPMIVc)