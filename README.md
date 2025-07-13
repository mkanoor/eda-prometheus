# eda-prometheus

## Configuring event streams in alertmanager

### EDA Setup
Login to EDA

Create a Basic EventStream Credential 
 - Name: demo
 - Organization: Default
 - Credential type: select Basic Event Stream
 - Username: fred
 - Password: secret

Create a Basic Event Stream (name: es1)
 - Name: es1
 - Organization: Default
 - Event stream type:select Basic Event Stream
 - Credential: select demo

Copy the Event Stream URL which is generated for you.

If using podman containers you might have to change the localhost
to host.containers.internal so that alertmanager can reach AAP

### Alertmanager Setup
Edit the alertmanager/alertmanager.yml

Set the webhook_configs url to be EventStream URL
Set the basic_auth username and password from above
If using self signed certificate for AAP set
tls_config:
    insecure_skip_verify: true


### Prometheus Setup
Edit the prometheus/prometheus.yml

In the job_name: 'blackbox' set the static_configs targets to be
the servers you want to monitor
e.g.
   - targets:
     - https://www.example.com
     - https://prometheus.io

Start the containers using

docker-compose up


# To check the containers

Prometheus: http://localhost:9090
Blackbox Exporter: http://localhost:9115/
Alertmanager: http://localhost:9030/

# To test Blackbox
To test Blackbox exporter and probe_success
curl 'localhost:9115/probe?target=https://www.yahoo.com&module=http_2xx&debug=true

# Sample Prometheus Payload with multiple alerts
```
alerts:
- annotations:
    description: https://prometheus.io is reachable for more than 1 minute.
    summary: Host https://prometheus.io is up
  endsAt: '0001-01-01T00:00:00Z'
  fingerprint: 226ceaf4d7c8760e
  generatorURL: http://a847ec21bcc3:9090/graph?g0.expr=probe_success%7Bjob%3D%22blackbox%22%7D+%3D%3D+1&g0.tab=1
  labels:
    alertname: HostUp
    instance: https://prometheus.io
    job: blackbox
    severity: critical
  startsAt: '2025-07-13T20:36:16.646Z'
  status: firing
- annotations:
    description: https://www.example.com is reachable for more than 1 minute.
    summary: Host https://www.example.com is up
  endsAt: '0001-01-01T00:00:00Z'
  fingerprint: dbf8a773a01078c8
  generatorURL: http://a847ec21bcc3:9090/graph?g0.expr=probe_success%7Bjob%3D%22blackbox%22%7D+%3D%3D+1&g0.tab=1
  labels:
    alertname: HostUp
    instance: https://www.example.com
    job: blackbox
    severity: critical
  startsAt: '2025-07-13T20:36:16.646Z'
  status: firing
commonAnnotations: {}
commonLabels:
  alertname: HostUp
  job: blackbox
  severity: critical
externalURL: http://localhost:9093
groupKey: '{}:{alertname="HostUp"}'
groupLabels:
  alertname: HostUp
receiver: my-eda-eventstream
status: firing
truncatedAlerts: 0
version: '4'
```
