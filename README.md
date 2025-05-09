# Assignment-10-

This section provides a brief overview of the setup process, explains the configuration and logic behind our monitoring and alerting system, details the alert rationale and testing procedures, and describes the SMTP setup for email notifications.
1. Setup Process
   
The monitoring and alerting system was set up using Docker Compose to orchestrate the following services:
  Microservices: The application's core services, including the WebSocket-based Notification Service (as updated from Lab 8's MQTT version). Each service is configured to expose metrics.
  
  Node Exporter: Deployed to collect system-level metrics from the host (CPU, memory, disk usage) and exposes them on port 9100.
  
  Prometheus: Configured to scrape metrics from Node Exporter, the Notification Service, and all other microservices. It runs on port 9090 and uses prometheus.yml for its configuration, including scrape jobs and alert rules defined in alert-rules.yml.
  
  Grafana: Connected to Prometheus as a data source. It runs on port 3000 and is used to visualize the collected metrics through custom dashboards.
  
  Alertmanager: (For Bonus Email Alerts) Configured to receive alerts from Prometheus and send email notifications based on the rules defined in alertmanager.yml. It runs on port 9093.
The entire setup is managed using the docker-compose.yml file, which defines the services, their dependencies, network configurations, and volume mounts for configuration files.

2 Explanation of Setup, Alert Logic, and Testing

  Setup: The system was set up by defining each service in the docker-compose.yml file, specifying the Docker image to use, port mappings, and any necessary volume mounts for configuration files (e.g., prometheus.yml, alert-rules.yml, alertmanager.yml). A shared Docker network ensures that the services can communicate with each other using their service names as hostnames.
  
  Alert Logic: Alert rules are defined in the alert-rules.yml file. These rules are based on PromQL expressions that evaluate the collected metrics. When a defined condition is met for a specified duration, an alert is triggered and sent to Alertmanager.
  
    High Notification Service Error Rate: An alert is configured to fire if the rate of errors in the Notification Service exceeds a certain threshold within a defined time window. This helps identify potential issues with message delivery.
    
    Excessive CPU/Memory Usage: Alerts are set up to trigger if the CPU or memory usage on the host (as reported by Node Exporter) goes above a critical level for a sustained period. This indicates potential resource exhaustion.
    
  Testing: To verify the alerting system, we simulated conditions that should trigger the defined alerts:
  
    High Notification Service Error Rate: We induced errors in the Notification Service (e.g., by sending malformed requests or simulating internal failures) to observe if the error rate increased and an alert was triggered in Prometheus and (if configured) forwarded to Alertmanager.
    
    Excessive CPU/Memory Usage: We simulated high CPU or memory load on the system (e.g., by running CPU-intensive tasks) to check if Node Exporter reported the increased usage and if Prometheus triggered the corresponding alerts.
    
3. Alert Rationale and Testing Steps
   
  Alert Rationale: The chosen alerts are critical for maintaining the health and reliability of the platform:
  
    High Notification Error Rate: Timely detection of high error rates in the notification service is crucial to ensure users receive important communications. A rapid response to such alerts can prevent widespread notification failures.
    
    Excessive CPU/Memory Usage: High resource utilization can lead to performance degradation and potential service instability across all microservices running on the same host. Proactive alerting allows for investigation and mitigation before outages occur.
  Testing Steps:
    High Notification Error Rate:
      Action: Introduced intentional errors within the Notification Service logic.
      
      Expected Outcome: The notification_errors_total metric should increase, and Prometheus should trigger the HighNotificationErrorRate alert based on the defined rule in alert-rules.yml. The alert should be visible in the Prometheus UI. If configured, Alertmanager should then send an email notification.
      
    Excessive CPU/Memory Usage:
      Action: Utilized system tools to generate a high CPU load (e.g., stress command) or consume a significant amount of memory on the machine running the Docker containers.
      
      Expected Outcome: Node Exporter should report the increased CPU and/or memory usage. Prometheus should then trigger the ExcessiveCPUUsage or ExcessiveMemoryUsage alert based on the rules in alert-rules.yml. The alert should be visible in the Prometheus UI. If configured, Alertmanager should send an email notification.
   
5. Brief Explanation of Your SMTP Setup (e.g., Gmail/Outlook/local server)
For the bonus email alert functionality, we configured Alertmanager to use [Specify your SMTP setup here, e.g., Gmail's SMTP server]. The alertmanager.yml file was configured with the following details:

SMTP Server: smtp.YOUR_SMTP_SERVER:YOUR_SMTP_PORT (e.g., smtp.gmail.com:587 for Gmail).

  Authentication: The smtp_auth_username was set to our Gmail address, and smtp_auth_password was configured with the corresponding application-specific password (as Gmail requires for external applications).
  
  Sender Address: The smtp_from address was set to the address used for sending alerts.
  TLS: smtp_require_tls was set to true to ensure a secure connection to the SMTP server.
This setup allows Alertmanager to send email notifications to the specified recipient email address whenever critical alerts are triggered by Prometheus
