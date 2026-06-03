# Setup Guide

This guide shows the main installation and configuration steps used to build the Elastic Stack machine monitoring setup.

---

## 1. Get the VM IP Address

Record the IP address of each VM because the addresses are used in the configuration files.

For Linux:

```bash
ip a
```

For Windows:

```powershell
ipconfig /all
```

---

## 2. Install Java on Linux VMs

Install Java if it is not already available.

```bash
java -version
```

```bash
sudo apt update
sudo apt install openjdk-11-jdk
```

---

## 3. Elasticsearch Setup

Download and install Elasticsearch.

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.6.2-amd64.deb
sudo dpkg -i elasticsearch-8.6.2-amd64.deb
```

Edit the Elasticsearch configuration file.

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Use the example file in `configs/elasticsearch.yml.example`

Start Elasticsearch.

```bash
sudo systemctl start elasticsearch
```

Check that Elasticsearch is running.

```bash
sudo systemctl status elasticsearch
curl -X GET http://<ELASTICSEARCH_SERVER_IP>:9200/?pretty
```

Access Elasticsearch in the browser at `http://<ELASTICSEARCH_SERVER_IP>:9200`

---

## 4. Kibana Setup

Download and install Kibana.

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.6.2-amd64.deb
sudo dpkg -i kibana-8.6.2-amd64.deb
```

Edit the Kibana configuration file.

```bash
sudo nano /etc/kibana/kibana.yml
```

Use the example file in `configs/kibana.yml.example`

Start Kibana.

```bash
sudo systemctl start kibana
```

Check that Kibana is running.

```bash
sudo systemctl status kibana
```

Access Kibana in the browser at `http://<KIBANA_SERVER_IP>:5601`

---

## 5. Logstash Setup

Download and install Logstash.

```bash
wget https://artifacts.elastic.co/downloads/logstash/logstash-8.6.2-amd64.deb
sudo dpkg -i logstash-8.6.2-amd64.deb
```

Create or edit the Logstash pipeline configuration file.

```bash
sudo nano /etc/logstash/conf.d/logstash.conf
```

Use the example file in `configs/logstash.conf.example`

Test the Logstash configuration.

```bash
sudo /usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/logstash.conf
```

Start Logstash.

```bash
sudo systemctl start logstash
```

Check that Logstash is running.

```bash
sudo systemctl status logstash
```

---

## 6. Metricbeat Setup

Download and install Metricbeat.

```bash
wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-8.6.2-amd64.deb
sudo dpkg -i metricbeat-8.6.2-amd64.deb
```

Edit the Metricbeat configuration file.

```bash
sudo nano /etc/metricbeat/metricbeat.yml
```

Use the example file in `configs/metricbeat.yml.example`

Enable the system module.

```bash
sudo metricbeat modules enable system
```

Test the Metricbeat configuration.

```bash
sudo metricbeat test config
```

Test the Metricbeat output connection.

```bash
sudo metricbeat test output
```

Start Metricbeat.

```bash
sudo systemctl start metricbeat
```

Check that Metricbeat is running.

```bash
sudo systemctl status metricbeat
```

---

## 7. Winlogbeat Setup

Download and extract Winlogbeat on the Windows VM.

```powershell
Start-BitsTransfer -Source "https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-8.6.2-windows-x86_64.zip" -Destination "C:\winlogbeat.zip"
Expand-Archive -Path "C:\winlogbeat.zip" -DestinationPath "C:\winlogbeat" -Force
Get-ChildItem -Path "C:\winlogbeat"
cd C:\winlogbeat\winlogbeat-8.6.2-windows-x86_64
```

Install the Winlogbeat service.

```powershell
.\install-service-winlogbeat.ps1
```

Edit the Winlogbeat configuration file.

```powershell
notepad.exe winlogbeat.yml
```

Use the example file in `configs/winlogbeat.yml.example`

Test the Winlogbeat configuration.

```powershell
.\winlogbeat.exe test config -c winlogbeat.yml -e
```

Test the Logstash output connection.

```powershell
.\winlogbeat.exe test output
```

Start Winlogbeat.

```powershell
Start-Service winlogbeat
```

Check that Winlogbeat is running.

```powershell
Get-Service winlogbeat
```

---

## 8. Filebeat Setup

Download and install Filebeat.

```bash
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.2-amd64.deb
sudo dpkg -i filebeat-8.6.2-amd64.deb
```

Edit the Filebeat configuration file.

```bash
sudo nano /etc/filebeat/filebeat.yml
```

Use the example file in `configs/filebeat.yml.example`

Enable the required Filebeat modules.

```bash
sudo filebeat modules enable system
sudo filebeat modules enable auditd
```

Or manually edit the module files using the example files in `configs/filebeat-modules/`

```bash
sudo nano /etc/filebeat/modules.d/system.yml
sudo nano /etc/filebeat/modules.d/auditd.yml
```

Verify that the modules are enabled.

```bash
sudo filebeat modules list | grep system
sudo filebeat modules list | grep auditd
```

Test the Filebeat configuration.

```bash
sudo filebeat test config
```

Test the connection to Logstash.

```bash
sudo /usr/share/filebeat/bin/filebeat --path.home /usr/share/filebeat -e -c /etc/filebeat/filebeat.yml test output
```

Start Filebeat.

```bash
sudo systemctl start filebeat
```

Check that Filebeat is running.

```bash
sudo systemctl status filebeat
```

---