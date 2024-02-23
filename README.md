## Monitoring Apache metrics with Prometheus, Fluentd, and Grafana

## Introduction
This project guides you through the process of setting up an Apache web server, integrating Prometheus and Fluentd for monitoring, and visualizing the metrics on Grafana. Alerts for 4XX and 5XX errors are configured in Prometheus, and the final step involves creating a Grafana dashboard for comprehensive visualization.

## Table of Contents
1. Setting up Apache Web Server
2. Creating a Dummy HTML Page
3. Installing Prometheus
4. Feeding Apache Server Logs to Prometheus
5. Creating Dashboards and Alerts in Prometheus
6. Integrating Grafana for Visualization
7. Testing and Validation

## 1. Setting up Apache Web Server
Install Apache using the operating system's package manager.
Ensure Apache is running by accessing the default landing page in a web browser.

`$ brew install httpd`

## 2. Creating a Dummy HTML Page
Create a simple HTML page using a text editor.
Save the HTML page in the default Apache web server directory (e.g., /var/www/html/ on Linux systems).
```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>My Apache Web Page</title>
</head>
<body>
<header>
<h1>Welcome to My Apache Web Page</h1>
</header>
<section>
<p>This is a simple HTML page served by Apache.</p>
</section>
<footer>
<p>&copy; 2024 My Name</p>
</footer>
</body>
</html>
```
## 3. Installing Prometheus
Download and install Prometheus from the official website.
Ensure Prometheus is running by accessing its web interface at http://localhost:9090.
```
$ prometheus --config.file=/Path/to/prometheus.yml
```
## 4. Feeding Apache Server Logs to Prometheus
Check for a direct way to feed Apache logs to Prometheus.
Install Fluentd as an intermediary tool to tail Apache access and error logs.
Using Docker: 

`$ docker build -t my-fluentd-image .`

`$ docker run -p 24231:24231 my-fluentd-image`

```
Configure Fluentd to export log data in a format Prometheus can scrape.
fluent.conf:
<source>
@type forward
@id forward_input
bind 127.0.0.1
port 24224
</source>

<source>
@type http
@id http_input
bind 127.0.0.1
port 8080
</source>

<source>
@type tail
path /opt/homebrew/var/log/httpd/access_log
tag apache.access
pos_file /path/to/access_log.pos
<parse>
@type apache
</parse>
</source>

<source>
@type tail
path /opt/homebrew/var/log/httpd/error_log
tag apache.error
pos_file /path/to/error_log.pos
<parse>
@type apache
</parse>
</source>

<match apache.**>
@type prometheus
</match>

<match company.*>
@type copy
<store>
@type prometheus
<metric>
name fluentd_output_status_num_records_total
type counter
desc The total number of outgoing records
<labels>
tag ${tag}
hostname ${hostname}
</labels>
</metric>
</store>
</match>

<match apache.access>
@type prometheus
<metric>
name apache_http_requests_total
type counter
desc Total number of requests
</metric>
</match>

# Add a debug output to troubleshoot
<match apache.access>
@type stdout
@id stdout_output_apache_access
</match>

# Expose metrics in Prometheus format
<source>
@type prometheus
bind 0.0.0.0
port 24231
metrics_path /metrics
</source>

<source>
@type prometheus_output_monitor
interval 10
<labels>
hostname ${hostname}
</labels>
</source>
```
<img width="1423" alt="Screenshot 2024-02-20 at 5 11 26 PM" src="https://github.com/pveerrotwal/titanwork/assets/108364147/46044708-397a-4541-97db-0c8bc89a17a3">

## 5. Creating Dashboards and Alerts in Prometheus
Define Prometheus scraping targets for Fluentd-exported Apache log metrics.
```
scrape_configs:
- job_name: 'fluentd'
static_configs:
- targets: ['localhost:24231']
```
Create Prometheus queries to monitor occurrences of 4XX and 5XX errors in Apache logs.
groups:
```
- name: apache_alerts
rules:
 - alert: Apache4XXErrors
expr: sum(apache_http_requests_total{status=~"4.."}) > 0
for: 1m
annotations:
summary: "High 4XX Errors on Apache"
description: "There are 4XX errors happening on Apache. Investigate further."
 - alert: Apache5XXErrors
expr: sum(apache_http_requests_total{status=~"5.."}) > 0
for: 1m
annotations:
summary: "High 5XX Errors on Apache"
description: "There are 5XX errors happening on Apache. Investigate further."
```
Configure alerting rules in Prometheus to trigger alerts for exceeded error thresholds.
Define notification channels (e.g., email, Slack) for alert delivery to the developer group.
```
route:
group_by: ['alertname', 'cluster', 'service']
group_wait: 30s
group_interval: 5m
repeat_interval: 3h
receiver: 'default'
receivers:
- name: 'default'
email_configs:
- to: 'xyz@gmail.com'
send_resolved: true
```
## 6. Integrating Grafana for Visualization
Install Grafana using the official installation guide.

`$ brew install grafana`

Configure Prometheus as a data source in Grafana.
Create a Grafana dashboard, visualizing key Apache metrics and error rates.

<img width="1440" alt="Screenshot 2024-02-20 at 5 28 12 PM" src="https://github.com/pveerrotwal/titanwork/assets/108364147/03334e25-b161-4395-b059-41e74fe86f0a">

Add panels for 4XX and 5XX error occurrences.

## 7. Testing and Validation
Test the entire setup by deliberately triggering 4XX and 5XX errors on the Apache server.
Verify that Prometheus detected errors, triggered alerts, and sent notifications.
Explore the Grafana dashboard for visual representation of Apache metrics.
Iterate on configurations, alerts, and dashboard designs based on feedback and observed behavior.

## Conclusion

This project provides a comprehensive guide for monitoring Apache web server performance and errors using Prometheus, Fluentd, and Grafana. The combination of these tools offers robust monitoring, alerting, and visualization capabilities for effective system management. Feel free to customize and expand upon the project based on your specific needs.

