## WELCOME TO THE INSTALLATION PROCESS OF WINLOGBEAT AND SYSMON ##

<h3>OS Windows 7/10/11 </h3>

1- Download winlogbeat via official web site:\
<a href="https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-9.1.2-windows-x86_64.zip"> Click Here to download </a>\
2- Rename the file to "winlogbeat"\
3- Move the winlogbeat file to 'C:\Program Files\'.\
4- inside the folder you will find a configuration file named "winlogbeat.yml".\
5- Open it, clear it, Then paste the content bellow in it:
```
###################### Winlogbeat Configuration Example ########################

# This file is an example configuration file highlighting only the most common
# options. The winlogbeat.reference.yml file from the same directory contains
# all the supported options with more comments. You can use it as a reference.
#
# You can find the full configuration reference here:
# https://www.elastic.co/guide/en/beats/winlogbeat/index.html

# ======================== Winlogbeat specific options =========================

# event_logs specifies a list of event logs to monitor as well as any
# accompanying options. The YAML data type of event_logs is a list of
# dictionaries.
#
# The supported keys are name, id, xml_query, tags, fields, fields_under_root,
# forwarded, ignore_older, level, event_id, provider, and include_xml.
# The xml_query key requires an id and must not be used with the name,
# ignore_older, level, event_id, or provider keys. Please visit the
# documentation for the complete details of each option.
# https://go.es.io/WinlogbeatConfig

winlogbeat.event_logs:
  - name: Application
    ignore_older: 72h

  - name: System

  - name: Security

  - name: Microsoft-Windows-Sysmon/Operational
    event_id: 1, 2, 3, 5, 6, 7, 8, 9, 10, 11, 12, 15, 16, 17, 20, 21, 22
  - name: Windows PowerShell
    event_id: 400, 403, 600, 800

  - name: Microsoft-Windows-PowerShell/Operational
    event_id: 4103, 4104, 4105, 4106

  - name: ForwardedEvents
    tags: [forwarded]

# ====================== Elasticsearch template settings =======================

setup.template.settings:
  index.number_of_shards: 1
  #index.codec: best_compression
  #_source.enabled: false


# ================================== General ===================================

# The name of the shipper that publishes the network data. It can be used to group
# all the transactions sent by a single shipper in the web interface.
#name:

# The tags of the shipper are included in their field with each
# transaction published.
#tags: ["service-X", "web-tier"]

# Optional fields that you can specify to add additional information to the
# output.
#fields:
#  env: staging

# ================================= Dashboards =================================
# These settings control loading the sample dashboards to the Kibana index. Loading
# the dashboards is disabled by default and can be enabled either by setting the
# options here or by using the `setup` command.
####setup.dashboards.enabled: true

# The URL from where to download the dashboard archive. By default, this URL
# has a value that is computed based on the Beat name and version. For released
# versions, this URL points to the dashboard archive on the artifacts.elastic.co
# website.
#setup.dashboards.url:

# =================================== Kibana ===================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
#setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  #host: "localhost:5601"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:

# =============================== Elastic Cloud ================================

# These settings simplify using Winlogbeat with the Elastic Cloud (https://cloud.elastic.co/).

# The cloud.id setting overwrites the `output.elasticsearch.hosts` and
# `setup.kibana.host` options.
# You can find the `cloud.id` in the Elastic Cloud web UI.
#cloud.id:

# The cloud.auth setting overwrites the `output.elasticsearch.username` and
# `output.elasticsearch.password` settings. The format is `<user>:<pass>`.
#cloud.auth:

# ================================== Outputs ===================================

# Configure what output to use when sending the data collected by the beat.

# ---------------------------- Elasticsearch Output ----------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
#  hosts: ["localhost:9200"]

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  #username: "elastic"
  #password: "changeme"

  # Pipeline to route events to security, sysmon, or powershell pipelines.

# ------------------------------ Logstash Output -------------------------------

output.logstash:
  # The Logstash hosts
  hosts: ["10.10.10.2:5044"]
####setup.kibana:
  ####host: "http://192.168.159.130:5601"
  ####username: "elastic"
  ####password: "CD27P-N=8BW+LLEnjiV7"
  ####pipeline: "winlogbeat-%{[agent.version]}-routing"

# ================================= Processors =================================
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - drop_event:
      when:
        or:
          - regexp:
              winlog.event_data.SourceIp: "^[a-fA-F0-9:]+$"
          - regexp:
              winlog.event_data.DestinationIp: "^[a-fA-F0-9:]+$"

# ================================== Logging ===================================

# Sets log level. The default log level is info.
# Available log levels are: error, warning, info, debug
#logging.level: debug

# At debug level, you can selectively enable logging only for some components.
# To enable all selectors, use ["*"]. Examples of other selectors are "beat",
# "publisher", "service".
#logging.selectors: ["*"]

# ============================= X-Pack Monitoring ==============================
# Winlogbeat can export internal metrics to a central Elasticsearch monitoring
# cluster.  This requires xpack monitoring to be enabled in Elasticsearch.  The
# reporting is disabled by default.

# Set to true to enable the monitoring reporter.
#monitoring.enabled: false

# Sets the UUID of the Elasticsearch cluster under which monitoring data for this
# Winlogbeat instance will appear in the Stack Monitoring UI. If output.elasticsearch
# is enabled, the UUID is derived from the Elasticsearch cluster referenced by output.elasticsearch.
#monitoring.cluster_uuid:

# Uncomment to send the metrics to Elasticsearch. Most settings from the
# Elasticsearch outputs are accepted here as well.
# Note that the settings should point to your Elasticsearch *monitoring* cluster.
# Any setting that is not set is automatically inherited from the Elasticsearch
# output configuration, so if you have the Elasticsearch output configured such
# that it is pointing to your Elasticsearch monitoring cluster, you can simply
# uncomment the following line.
#monitoring.elasticsearch:

# ============================== Instrumentation ===============================

# Instrumentation support for the winlogbeat.
#instrumentation:
    # Set to true to enable instrumentation of winlogbeat.
    #enabled: false

    # Environment in which winlogbeat is running on (eg: staging, production, etc.)
    #environment: ""

    # APM Server hosts to report instrumentation results to.
    #hosts:
    #  - http://localhost:8200

    # API Key for the APM Server(s).
    # If api_key is set then secret_token will be ignored.
    #api_key:

    # Secret token for the APM Server(s).
    #secret_token:


# ================================= Migration ==================================

# This allows to enable 6.7 migration aliases
#migration.6_to_7.enabled: true

```
6- You will find an OUTPUT SECTION that looks like this
```
# ------------------------------ Logstash Output -------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["your-elk-vm:5044"]
```
You should replace the "your-elk-vm" by your actual ELK SIEM address ip in that configuration file.\
7- Open powershell as an administrator.Then type:
```
cd 'C:\Program Files\winlogbeat'
.\Powershell.exe -ep bypass
.\install-service-winlogbeat.ps1
Start-Service winlogbeat
```
8- Your winlogbeat is working now and sending logs to your logstash.\
9- Let's proceed with the installation of Sysmon\
Download winlogbeat via official web site: <a href="https://download.sysinternals.com/files/Sysmon.zip"> Click Here to download </a>\
10- Rename the Downloaded File to Sysmon\
11- create a new xml file and name it Sysmon.xml, then fill it wih the content bellow:
```
<Sysmon schemaversion="4.90">
  <EventFiltering>

    <!-- Process Events -->
    <ProcessCreate onmatch="include"/>
    <ProcessTerminate onmatch="include"/>

    <!-- File Events -->
    <FileCreate onmatch="include"/>
    <FileCreateTime onmatch="include"/>
    <FileDelete onmatch="include"/>

    <!-- Registry Events -->
    <RegistryEvent onmatch="include"/>

    <!-- Network Connections -->
    <NetworkConnect onmatch="include">
      <!-- Exclude noisy system processes -->
      <Image condition="is not">C:\Windows\System32\svchost.exe</Image>
      <Image condition="is not">C:\Windows\System32\lsass.exe</Image>
      <Image condition="is not">C:\Windows\System32\services.exe</Image>
      <Image condition="is not">C:\Windows\System32\wininit.exe</Image>
    </NetworkConnect>

    <!-- Driver Loads -->
    <DriverLoad onmatch="include"/>

    <!-- Image Loads -->
    <ImageLoad onmatch="include">
      <ImageLoaded condition="is not">C:\Windows\System32\svchost.exe</ImageLoaded>
    </ImageLoad>

    <!-- WMI, Pipes, Clipboard -->
    <WmiEvent onmatch="include"/>
    <PipeEvent onmatch="include"/>
    <ClipboardChange onmatch="include"/>

    <!-- DNS Queries -->
    <DnsQuery onmatch="include"/>

  </EventFiltering>
</Sysmon>

```
12- Save the file, then place it inside the Sysmon folder.\
13- Move the sysmon folder into 'C:\Program Files\'.\
14- Open a Powershell as administrator, the type:
```
cd 'C:\Program Files\Sysmon\'
.\Powershell.exe -ep bypass
.\Sysmon64.exe -i Sysmon.xml -n -accepteula

```
15- You have now winlogbeat and sysmon sending logs perfectly to you SIEM.
