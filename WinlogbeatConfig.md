
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

