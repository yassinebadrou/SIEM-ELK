## WELCOME TO THE INSTALLATION PROCESS OF ELK STACK ##

<h3>OS : ubuntu 20.04</h3>
<h3>TYPE : ALL-IN-ONE installation</h3>

1-  Update your packages in install some by tapping 
```
sudo apt update && sudo apt install -y net-tools unzip curl gnupg
```

2-  Now inport the PGP Key of elasticsearch repository by tapping 
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
3-  Secure your repos search by tapping 
```
apt-get install apt-transport-https
```
4-  Add the elasticsearch repository to your apt.source file by tapping (You will be installing ELK version 9):
```
echo "deb https://artifacts.elastic.co/packages/9.x/apt stable main" > /etc/apt/sources.list.d/elastic-9.x.list
```
5-  Update and install elasticsearch. for this type 
```
apt update
apt install elasticsearch -y
```
6-  Please save the authentification password of your elastic built-in superuser. You will see it at the end of the installation in the terminal

7-  Type these three commands:
```
systemctl daemon-reload
systemctl enable elasticsearch.service
```
8-  Test if your elasticsearch is working well by taping the command below, you will be asked to enter your username(elastic) and your password (superuser password).
```
curl http://localhost:9200
```
9-  Note that by default your elasticsearch node is only accessible from your localhost, if you want your node to be exposed to all your network, you need to put a non-loopback address 
    by uncommenting and modifying two lines in "/etc/elasticsearch/elasticsearch.yml" as follows:
	Go from This 
```
#network.host: localhost
#http.port: 9200
```
To this
```
network.host: your-vm-ip
http.port: 9200
```
10- In case you will send your logs directly to elasticsearch (if not PLEASE IGNORE THIS STEP), you need to genearte the elasticseatch fingerprint go to /etc/elasticsearch/certs/ then type:
```
openssl x509 -fingerprint -sha256 -in http_ca.crt
```
then remove the "two dots" between every two number (ex : AB:CD:7R => ABCD7R) \
11- Start you elasticsearch service now:
```
sudo systemctl start elasticsearch
```
12- Now let's proceed with the installation of Kibana. Type 
```
apt update
apt install kibana
```
13- As for elasticsearch, your kibana is only discoverable on your localhost, if you wanna expose it to the network, proceed as follow:
```
nano /etc/kibana/kibana.yml
```
Then Uncomment and modify 3 lines in this kibana configuration file. Go from :
```
#server.port: 5601
#server.host: "locahost"
#server.publicBaseUrl: "http://localhost:5601"
```
To this
```
server.port: 5601
server.host: "your-vm-ip"
server.publicBaseUrl: "http://your-vm-ip:5601"
```
14- Now you need to add some Encryption keys in the end of your kibana configuration file (/etc/kibana.kibana.yml).
```
cd /usr/share/kibana/bin
./kibana-encryption-keys generate
```
This will generate something like this, copy these 3 lines.
```
xpack.encryptedSavedObjects.encryptionKey: 42498487153da602c24473a44094ed27
xpack.reporting.encryptionKey: d44cbfd88a48def23f0707fd782d3245
xpack.security.encryptionKey: 28da2e750a88c60f2116355360edac61
```
Open your kibana config file by taping the command below then paste the 3 lines that you copied before at the end of /etc/kibana/kibana.yml configuration file, then exit.
```
nano /etc/kibana.kibana.yml
```
15- Start kibana service now
```
systemctl daemon-reload
systemctl enable kibana.service
systemctl start kibana.service
```
16- In order to ensure your kibana will join the elk stack, open your browser and type "http://your-vm:5601" . You'll see a text area
    inviting your to enter an enrollement token.Open the command line and type 
```
./usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```
You'll see a token, copy it and paste it in your the text area in the browser, click configure then you will be asked to enter a verification code. \

17- Type in your terminal 
```
./usr/share/kibana/bin/kibana-verification-code
```
You'll see a six number code, copy it and paste it in your browser again.\
You will see the authentication page enter "elastic" as a username then your built-in superuser that you have savec during elasticsearch installation as a password.\

18- Now let's proceed with the installation of Logstash. Type
```
apt update
apt install logstash
```
19- You will need to specify the path of your logstash pipelines configuration settings.
Open a terminal then type:
```
nano /etc/logstash/logstash.yml
```
You will see a section called "Pipeline Configuration Settings", under it replace this line 
```
#path.config : 
```
with this 
```
path.config: /etc/logstash/conf.d/*.conf
```
This will ensure that you logs will be going through every configuration file located in "/etc/logstash/conf.d" file.\
\
20- Until now, your elasticsearch communication very well with your kibana, but we should ensure that our logstash communicate perfectly with our elasticsearch\
this is the Overvue of the Stack
```
AGENT >> LOGSTASH >> ELASTICSEARCH >> KIBANA 
```
20.1- You need to identify you home user, check it in "/home/" directory. let's go for the example of "michael"\
Your home directory will be "/home/michael". we need to copy elasticsearch certificate and make it accessible by logstash in order to ensure the communication betweek logstash and elasticsearch. To do so, Type:
```
cp /etc/elasticsearch/certs/http_ca.crt /home/michael/
chmod 777 /home/michael/http_ca.crt
```
20.2- We will create the configuration files corresponding to the 3 steps of logstash pipeline
```
cd /etc/logstash/conf.d/
touch 001-input.conf
touch 101-filter.conf
touch 201-output.conf
```
20.3- we will fill these 3 files as follow:\
open your 001-input.conf
```
nano 001-input.conf
```
Then paste this:
```
# #################################################################################################################################
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
    beats {
        port => 5044
    }
}
```
open your 101-filter.conf
```
nano 101-filter.conf
```
Then paste this: 
```
filter {

  ##########################################################################
  # 0) Basic guards
  ##########################################################################
  if ![winlog] { drop { } }

  ##########################################################################
  # 1) Ensure event.code exists (Sysmon/Windows) and is an integer
  ##########################################################################
  if [winlog][event_id] {
    mutate {
      copy    => { "[winlog][event_id]" => "[event][code]" }
      convert => { "[event][code]" => "integer" }
    }
  }

  # Also normalize winlog version/opcode if present
  mutate {
    convert => {
      "[winlog][version]" => "integer"
      "[winlog][opcode]"  => "integer"
    }
  }

  ##########################################################################
  # 2) ECS host / user / process basics (non-destructive copies)
  ##########################################################################
  if [winlog][computer_name] and ![host][hostname] {
    mutate { add_field => { "[host][hostname]" => "%{[winlog][computer_name]}" } }
  }

  if [winlog][user][name] and ![user][name] {
    mutate { add_field => { "[user][name]" => "%{[winlog][user][name]}" } }
  }

  # Sysmon often provides ProcessId / ParentProcessId under event_data
  if [winlog][event_data][ProcessId] {
    mutate {
      add_field => { "[process][pid]" => "%{[winlog][event_data][ProcessId]}" }
      convert   => { "[process][pid]" => "integer" }
    }
  }
  if [winlog][event_data][ParentProcessId] {
    mutate {
      add_field => { "[process][parent][pid]" => "%{[winlog][event_data][ParentProcessId]}" }
      convert   => { "[process][parent][pid]" => "integer" }
    }
  }

  # Executable / CommandLine
  if [winlog][event_data][Image] and ![process][executable] {
    mutate { add_field => { "[process][executable]" => "%{[winlog][event_data][Image]}" } }
  }
  if [winlog][event_data][CommandLine] and ![process][command_line] {
    mutate { add_field => { "[process][command_line]" => "%{[winlog][event_data][CommandLine]}" } }
  }

  ##########################################################################
  # 3) Network ECS mappings + type fixes
  ##########################################################################
  # Copy/rename common Sysmon/Windows fields
  if [winlog][event_data] {
    mutate {
      rename => {
        "[winlog][event_data][SourceIp]"              => "[source][ip]"
        "[winlog][event_data][DestinationIp]"         => "[destination][ip]"
        "[winlog][event_data][SourcePort]"            => "[source][port]"
        "[winlog][event_data][DestinationPort]"       => "[destination][port]"
        "[winlog][event_data][Protocol]"              => "[network][transport]"
        "[winlog][event_data][DestinationPortName]"   => "[service][name]"
        "[winlog][event_data][QueryName]"             => "[dns][question][name]"
        "[winlog][event_data][QueryResults]"          => "[dns][answers]"
        "[winlog][event_data][LogonType]"             => "[winlog][logon_type]"
      }
    }

    mutate {
      convert => {
        "[source][port]"         => "integer"
        "[destination][port]"    => "integer"
        "[winlog][logon_type]"   => "integer"
      }
    }

    # Bytes/length if present
    mutate {
      convert => {
        "[network][bytes]"       => "integer"
        "[source][bytes]"        => "integer"
        "[destination][bytes]"   => "integer"
      }
    }
  }

  ##########################################################################
  # 4) DNS enrichment (reverse lookup) with safe cache + fallback
  ##########################################################################
  if [destination][ip] {
    # Seed domain with IP before reverse lookup (dns filter replaces it)
    mutate {
      add_field => { "[destination][domain]" => "%{[destination][ip]}" }
    }

    dns {
      reverse          => [ "[destination][domain]" ]
      action           => "replace"
      nameserver       => [ "8.8.8.8", "1.1.1.1" ]
      timeout          => 5
      hit_cache_size   => 5000
      hit_cache_ttl    => 900
      failed_cache_size=> 1000
      failed_cache_ttl => 300
    }
  }

  # If event is DNS (Microsoft-Windows-DNS-Client/Operational, ID 22), prefer QueryName
  if [event][code] == 22 and [dns][question][name] and [destination][domain] {
    if [destination][domain] =~ /\.(in-addr|ip6|googleusercontent|cloudapp|amazonaws)\.com$/ {
      mutate { replace => { "[destination][domain]" => "%{[dns][question][name]}" } }
    }
  }

  ##########################################################################
  # 5) Reasonable drops (your original behavior preserved)
  ##########################################################################
  if [destination][ip] in ["8.8.8.8", "1.1.1.1"] {
    mutate { add_tag => ["dropped_dns_query"] }
    drop { }
  }

  ##########################################################################
  # 6) Extra Sysmon niceties: timestamps & booleans
  ##########################################################################
  # UtcTime -> event.created (leave @timestamp from winlogbeat)
  if [winlog][event_data][UtcTime] {
    date {
      match  => [ "[winlog][event_data][UtcTime]", "ISO8601", "yyyy-MM-dd HH:mm:ss.SSSZ", "yyyy-MM-dd HH:mm:ssZ" ]
      target => "[event][created]"
    }
  }

  # Common boolean-ish strings that show up in Sysmon (normalize)
  mutate {
    gsub => [
      # Trim spaces that sometimes creep in
      "[dns][question][name]", "^\s+|\s+$", ""
    ]
  }

  # Example: if event_data has "Allowed", "Yes", "No" etc. (add any you use)
  if [winlog][event_data][Allowed] {
    mutate { lowercase => [ "[winlog][event_data][Allowed]" ] }
    ruby {
      code => '
        val = event.get("[winlog][event_data][Allowed]")
        if ["true","yes","1"].include?(val)
          event.set("[rule][allowed]", true)
        elsif ["false","no","0"].include?(val)
          event.set("[rule][allowed]", false)
        end
      '
    }
  }

  ##########################################################################
  # 7) Housekeeping: remove stray custom fields you no longer want
  ##########################################################################
  mutate {
    remove_field => [ "[Image]" ]  # superseded by [process][executable]
  }

  ##########################################################################
  # 8) (Optional) Tag event.category/type from channel/provider
  ##########################################################################
  if [winlog][channel] and ![event][category] {
    # crude mapping; refine if you like
    if [winlog][channel] =~ /Security/ {
      mutate { add_field => { "[event][category]" => "authentication" } }
    } else if [winlog][channel] =~ /Sysmon/ {
      mutate { add_field => { "[event][category]" => "process" } }
    } else if [winlog][channel] =~ /DNS/ {
      mutate { add_field => { "[event][category]" => "network" } }
    }
  }
}
```
Open your 201-output.conf
```
nano 201-output.conf
```
Then paste this:
```
output {
    stdout { codec => rubydebug }
    elasticsearch {
        hosts => ["https://your-vm-ip:9200"]
        index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
        user => "elastic"
        password => "your-superuser-password"
        ssl_enabled => true
        ssl_verification_mode => full
        ssl_certificate_authorities => ["/home/michael/http_ca.crt"]
        action => "index"
    }
}
```
Please note that in this line " ssl_certificate_authorities => ["/home/michael/http_ca.crt"]" , We have mentionned for logstash the path of elasticsearch certificate to ensure a perfect communication.\
Replace your ELK VM ip address as well as your password.\
21- Now, we may start your logstash service
```
systemctl daemon-reload
systemctl start logstash.service
```
22- One last step to do, Elasticsearch create two shards (primary and replica) for every index for redundancy and high availability. But since we are only using an all-in-one installation, which mean we have elasticsearch, kibana and logstash in the same VM, we will NO longer need to have a replica of every shard.\
To fix this issue, open the browser:
```
http://your-vm-ip:5601
```
Then Go to the top left you'll find a 3 small horizontal lines, click on them\
Scroll down until you see "Management", under it click "Dev Tools"\
You will see and API terminal, paste the content bellow then run it
```
PUT _index_template/winlogbeat-template
{
  "index_patterns": ["winlogbeat-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```
23- If you need assistance with Logstash pipeline configuration files please refer to this folder <a href="">Logstash Configuration files<\a>
