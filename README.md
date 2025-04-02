## WELCOME TO THE INSTALLATION PROCESS OF ELK STACK ##

<h3>OS : ubuntu 20.04</h3>
<h3>TYPE : ALL-IN-ONE installation</h3>


1-  Update your packages b y tapping 
```
apt update
apt install gnupg
```

2-  Now inport the PGP Key of elasticsearch repository by tapping 
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
3-  Secure your repos search by tapping 
```
apt-get install apt-transport-https
```
4-  Add the elasticsearch repository to your apt.source file by tapping:
```
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" > /etc/apt/sources.list.d/elastic-8.x.list
```
5-  Update and install elasticsearch. for this tape 
```
apt update
apt install elasticsearch -y
```
6-  Please take note of the authentification password of your elastic built-in superuser. You will see it at the end of the installation 
7-  Tape these three commands:
```
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```
8-  Test if your elasticsearch is working well by taping 
```
curl http://localhost:9200
```
9-  Note that by default your elasticsearch node is only accessible from your localhost, if you want your node to be exposed to all your network, you need to put a non-loopback address 
    by uncommenting and modifying this line 
```    
"#network.host: localhost" in the "/etc/elasticsearch/elasticsearch.yml" file.
```
10- to genearte the elasticseatch fingerprint go to /etc/elasticsearch/certs/ then type 
```
openssl x509 -fingerprint -sha256 -in http_ca.crt
```
then remove the "two dots" between every two number (ex : AB:CD:7R => ABCD7R) \

11- Now let's proceed with the installation of Kibana. Tape 
```
apt update
apt install kibana
```
12- Start kibana service 
```
"systemctl start kibana.service
```
13- In order to ensure your kibana will join the elk stack, open your browser and tape "http://localhost:5601" . You'll see a text area
    inviting your to enter an enrollement token\
    
14- Tape 
```
./usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```
You'll see a token, copy it and past it in your browser. you will be asked to enter a verification code. 

15- Tape in your terminal 
```
./usr/share/kibana/bin/kibana-verification-code
```
You'll see a six number code, copy it and past it in your browser again.\

16- Please Note that like elasticsearch, by default your kibana node is only accessible from your localhost, if you want your node to be
    exposed to all your network, you need to put a non-loopback address by uncommenting and modifying this line 
```
"#server.host: "locahost"" in the "/etc/kibana/kibana.yml" file.

```
17- Your kibana is ready, use the password of the elastic built-in superuser and connect.\
18- Now let's proceed with the installation of Logstash. Tape
```
apt update
apt install logstash
```
19- Start your logstash service
```
systemctl start logstash.service
```
20- proceed with the configuration file in /etc/logstash/conf.d/* - this is where you put your input|filter|output files. \
21- Here is an example in case you are working with one configuration file.
```
# #################################################################################################################################
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
    beats {
        port => 5044
    }
}

output {
    stdout { codec => rubydebug }
    elasticsearch {
        hosts => ["https://192.168.100.2:9200"]
        index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
        user => "elastic"
        # SSL enabled
        ssl => true
        ssl_certificate_verification => true
        # Path to your Cluster Certificate .pem downloaded earlier
        cacert => "/home/user/http_ca.crt"
        password => "Gbrx2WKwN7A8NFOzuYe+"
        action => "create"
    }
}
# #################################################################################################################################
```

22- Note that you should copy the elasticsearch certificate located in /etc/elasticsearch/certs and named http_ca.crt and paste it in a path and give it access permissions, that way your logstash can communicate with your elasticsearch.
```
cp /etc/elasticsearch/certs/http_ca.crt /home/user/.
```
you will put this path in the cacert field of the previous step (config file of logstash) \

23- to test that your logstash is recieving logs from the beats type 
```
tcpdump -Xni eth0 port 5044
```
the interface eth0 may be different in each case. \

23- Make sure the important part in winlogbeat.yml or your beat looks like this :
```
# #################################################################################################################################
# ------------------------------ Logstash Output -------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["192.168.100.2:5044"]

setup.kibana:
  host: "http://192.168.100.2:5601"
  username: "elastic"
  password: "Gbrx2WKwN7A8NFOzuYe+"
  pipeline: "winlogbeat-%{[agent.version]}-routing"
# #################################################################################################################################
```

this is how you check for elasticsearch indexes
```
curl -k -u "elastic:password" -X GET "https://192.168.159.130:9200/_cat/indices?v"
```
24- At this stage your need to setup your configuration files for logstash, especialy for filtering.

SYSMON Packet:
open powershell as an administrator then type "Sysmon64.exe -accepteula -i -n"

```
 Credentials:
    -username : elastic
    -password : Gbrx2WKwN7A8NFOzuYe+
	-fingerprint : 0527283E2226E57946667418532187A56DD0F41F9B93C7EC59924B440F515EA8
```
