# Tugas Final Projek 
> Setup + konfigurasi ELK Server sampai selesai

**by : mahendteam**

## ELK Server Setup
 * [Dependensi ELK](#persiapan)
 * [ELK Installation](#ELK-Setup)
   * [Elasticsearch](#Install-Elasticsearch)
   * [Logstash](#logstash)
   * [Kibana](#Kibana)
 * [Setup Agent ELK](#Agent-ELK)


## Persiapan
Firewall rule untuk mengijinkan koneksi pipeline logstash
```
firewall-cmd --permanent --add-port=5044/tcp
firewall-cmd --permanent --add-port=5601/tcp
firewall-cmd --reload
```

Install JDK8
```
yum -y install java-1.8.0 wget
```

## ELK Setup
### Install Elasticsearch

Menambahkan sourceslist repo elk
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
```
nano /etc/yum.repos.d/elk.repo
```
```
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
Install elasticsearch
```
yum install -y elasticsearch
```

Konfigurasi Elasticsearch
```
nano /etc/elasticsearch/elasticsearch.yml
network.host: localhost
```

Service elasticsearch
```
systemctl enable elasticsearch
systemctl start elasticsearch
```

### logstash
Install logstash
```
yum -y install logstash
```


Opsi tambahan : Create SSL Certificate
```
nano /etc/pki/tls/openssl.cnf
subjectAltName = IP:[ip elk server]
```

```
cd /etc/pki/tls/
openssl req -x509 -days 365 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt
```

Konfigurasi logstash [Disini kami akan mengirimkan log apache dari agent ELK Server]
```
nano /etc/logstash/conf.d/logstash.conf
```

Konfigurasi logstash untuk mem-parsing log apache sudah kami siapkan di sini : 
**https://github.com/mahend01/ELKFinalProject/blob/master/Logstashapache2parsing.txt**



Logstash service 
```
systemctl start logstash
systemctl enable logstash
```

### Kibana
Install kibana
```
yum -y install kibana
```

Konfigurasi kibana
```
nano /etc/kibana/kibana.yml

server.host: "localhost"
elasticsearch.url: "http://localhost:9200"
```

Service kibana 
```
systemctl start kibana
systemctl enable kibana
```

## Agent-ELK
Agent ELK adalah server yang mengirimkan log ke ELK Server (bisa disebut agnet/client). Pada skenario projek kali ini kami mengirimkan log **apache2**
Dalam pengiriman log, agent menggunakan **filebeat**

Install apache (untuk skenario pengiriman access.log dan error.log)
```
apt update && apt install apache2 -y && service apache2 start
```

Firewall rule : mengijinkan koneksi pipeline logstash
```
firewall-cmd --permanent --add-port=5044/tcp
firewall-cmd --permanent --add-port=5601/tcp
firewall-cmd --reload
```

Install filebeat
```
//Deb
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.1.2-amd64.deb
dpkg -i filebeat-5.1.2-amd64.deb

//RPM
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.1.2-x86_64.rpm
sudo rpm -vi filebeat-5.1.2-x86_64.rpm
```

Service filebeat
```
systemctl enable filebeat
systemctl start filebeat
```

Enable module apache2 
```
filebeat modules enable apache2
```

Konfigurasi filbeat
```
nano /etc/filebeat/filebeat.yml

filebeat.inputs:
- type: log
 
      enabled: true
      paths:
        - /var/log/apache2/access.log
        - /var/log/apache2/error.log

output.logstash:
  hosts: ["[IP ELK Server]:5044"]
```

Melihat Log filebeat
```
cat /var/log/filebeat/filebeat
```
