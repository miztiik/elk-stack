# Install and configure Kibana on AWS
Kibana lets you visualize your Elasticsearch data and navigate the Elastic Stack, so you can do anything from learning why you're getting paged at 2:00 a.m. to understanding the impact rain might have on your quarterly numbers. Kibana gives you the freedom to select the way you give shape to your data. And you don’t always have to know what you're looking for. With its interactive visualizations, start with one question and see where it leads you.

Kibana Features:
 - **Visualize & Explore**: A Picture's Worth a Thousand Log Lines
 - **Time Series**
 - **Analyze Relationships with Graph**
 - **Explore Anomalies with Machine Learning**


Follow this article on **[Youtube](https://youtu.be/TTwI8gPMIVc)**
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/Kibana-Dashboards.gif)

### Pre-Requisites
 - An ES Cluster accessible through its FQDN - [How to configure ES in AWS on EC2](https://github.com/miztiik/elk-stack/tree/master/ElasticSearch) - [Youtube](https://youtu.be/7WE8AAdGSlM)
 - An instance sending data to ES using Logstash - [How to configure Logstash on EC2](https://github.com/miztiik/elk-stack/tree/master/Logstash) - [Youtube](https://youtu.be/YasrCKykAKo)

## Install Kibana
We are going to install Kibana in Amazon Linux 2, you should be able to adapt this procedure for say RHEL/CentOS/Fedora.
- Accept the GPG key of the repo
- Create repo URL
- Install from RPM

#### Accept the GPG key
```sh
# Accept the ElasticSearch GPG Key
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

#### Create Repo
Create a file called `kibana.repo` in the `/etc/yum.repos.d/` directory. The below repo is for `Kibana` version `6.x`
```sh
echo '[kibana-6.x]
name=Kibana repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
' | sudo tee /etc/yum.repos.d/kibana.repo
```

#### Install from RPM
```sh
# Install from RPM Repo
sudo yum -y install kibana

# For manual installations visit
# https://www.elastic.co/guide/en/kibana/master/rpm.html
```

## Configuring Kibana via `config file`
Kibana loads its configuration from the `/etc/kibana/kibana.yml` file by default. The main config item is to update the URL of Elasticsearch Domain & make it accessible from different IP(including internet!).

Find the line `elasticsearch.url` and append your `ES Domain URL`:
```sh
# DIRTY WAY TO update it is, (NON PRODUCTION CONFIGURATION, SECURE THEM REALLY!!!)

echo '
elasticsearch.url: "http://ec2-18-197-147-233.eu-central-1.compute.amazonaws.com:9200/" 
server.host: "0.0.0.0"
' | sudo tee /etc/kibana/kibana.yml
```

#### Configure Kibana to run as service
Use the `chkconfig` command to configure Kibana to start automatically when the system boots up
```sh
sudo chkconfig --add kibana
```
## Start Kibana
Kibana can be started and stopped using the `service` command.

**IMPORTANT Note:** Make sure that port 5601 is open in your security group!
```sh
# To Start/Stop Kibana 
sudo -i service kibana start

# To Stop Kibana 
sudo -i service kibana stop

# Checking Kibana logs
tail -f /var/log/kibana/kibana.stdout
```



## Access Kibana
Check the list of documents under `Browser` with this url `http://KIBANA_SERVER_URL_OR_IP:5601/`
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/Kibana-Dashboards-00.png)

### Visualizing documents
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/Kibana-Dashboards-01.png)