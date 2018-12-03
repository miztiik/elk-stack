# Install and configure Elasticsearch on AWS
Elasticsearch makes it easy to deploy, secure, operate, and scale Elasticsearch for log analytics, full text search, application monitoring, and more. 

ElasticSearch supports following use cases, 
 - Log Analytics
 - Full Text Search
 - Distributed Document Store
 - Real-time Application Monitoring
 - Clickstream Analytics

 This document deals with IaaS based implementation of ElasticSearch, If you are looking for Managed ElasticSearch Service - [AWS ElasticSearch Service FAQ](https://aws.amazon.com/elasticsearch-service/faqs/)

Follow this article on **[Youtube](https://youtu.be/7WE8AAdGSlM)**

![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/elk.png)

### Pre-Requisites
 - An EC2 Instance- [How to Create EC2 in AWS](https://www.youtube.com/watch?v=N_mP4mIqK8A&list=PLxzKY3wu0_FLaF9Xzpyd9p4zRCikkD9lE&index=11&t=0s)
 - An Security Group port to be open - [Attach SG to running EC2](https://www.youtube.com/watch?v=GlPTgGZR-j8﻿)
    - Port 9200: For REST API ( _for learning open to the internet_, usually private subnets in your VPC)
    - Port 9300: For ES Nodes to communicate ( _for learning open to the internet_, usually private subnets in your VPC) 

## Step 1: Install Elasticsearch

### Pre-Requisites
The minimum required Java version is 8.
```sh
java -version
# Install Java v8 (if it is lesser than v8)
sudo yum -y install java-1.8.0-openjdk
sudo yum -y remove java-1.7.0-openjdk
java -version
echo $JAVA_HOME
# export PATH=$JAVA_HOME/bin:$PATH 
```

We are going to install ES in Amazon Linux 2, you should be able to adapt this procedure for say RHEL/CentOS/Fedora.
Elasticsearch requires at least Java 8.
```sh
# Accept the ElasticSearch GPG Key
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
### Installing from the RPM repository
Create a file called `elasticsearch.repo` in the `/etc/yum.repos.d/` directory. The below repo is for ES version 6.x
```sh
cat > /etc/yum.repos.d/elasticsearch.repo << "EOF"
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF

# Install from RPM Repo
sudo yum -y install elasticsearch

# For manual installations visit
# https://www.elastic.co/downloads/elasticsearch
```

## Running Elasticsearch
Use the `chkconfig` command to configure Elasticsearch to start automatically when the system boots up
```sh
sudo chkconfig --add elasticsearch
```
Elasticsearch can be started and stopped using the `service` command
```sh
# To Start/Stop Elasticsearch 
sudo -i service elasticsearch start
sudo -i service elasticsearch stop
```

```sh
[root@ip-10-80-4-74 ~]# sudo -i service elasticsearch start
Starting elasticsearch:                                    [  OK  ]
```

#### Configuring Access to ElasticSearch through Public IP
Typically you add your corporate IP/private IP and allow the same in the firewall/security group to allow access to ElasticSearch. In our case, just to give a dirty hack to get going we will open to the internet.
DO THIS ONLY FOR LEARNING PURPOSES - NOT FOR BUSINESS WORKLOADS CONFIGURATIONS
```sh
echo "network.host: 0.0.0.0" >> /etc/elasticsearch/elasticsearch.yml
curl localhost:9200
```
We should see something like this:
```sh

[root@ip-10-80-4-74 elasticsearch]# curl localhost:9200
{
  "name" : "Gr7qXpl",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "XouOh_loQ62c3nHcDxbDOQ",
  "version" : {
    "number" : "6.3.1",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "eb782d0",
    "build_date" : "2018-06-29T21:59:26.107521Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
It is my strong recommendation that you put your servers in a load balacer. Check out how to setup [ELB here](https://www.youtube.com/watch?v=QyjDktNxdQg)


## Accessing the ES through GUI
Install this chrome plugin
```sh
https://github.com/mobz/elasticsearch-head
```
From your browser, poing to the public IP of your ES Cluster at port 9200
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/ELK-health-00.png)

## Next Steps
 - Pushing logs from Logstash to ElastiSearch - [Follow Here](https://github.com/miztiik/elk-stack/tree/master/Logstash) - [Youtube](https://youtu.be/YasrCKykAKo)