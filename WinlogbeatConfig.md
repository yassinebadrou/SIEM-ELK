## WELCOME TO THE INSTALLATION PROCESS OF WINLOGBEAT AND SYSMON ##

<h3>OS Windows 7/10/11 </h3>

1- Download winlogbeat via official web site:
<a href="https://www.elastic.co/fr/downloads/beats/winlogbeat" Click Here to download </a>



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

1- to test that your logstash is recieving logs from the beats type 
```
tcpdump -Xni eth0 port 5044
```
