## Detecting DoS Attacks with Prometheus: A Lab Guideline

This guideline outlines a lab to detect Denial-of-Service (DoS) attacks using Prometheus, a powerful open-source monitoring and alerting toolkit.

## What is Prometheus?

Prometheus is a time-series database and monitoring system that collects metrics from targets (e.g., servers, applications) at regular intervals. It uses a pull-based model, where Prometheus scrapes metrics from exposed HTTP endpoints on the targets. Prometheus provides a powerful query language (PromQL) for analyzing collected data and generating alerts.

## What are we detecting?

This lab will focus on detecting common DoS attack patterns by analyzing network traffic metrics. We will primarily look for:

*   **Increased Network Traffic Volume:** A sudden spike in network traffic (packets/second, bytes/second) can indicate a DoS attack.
*   **High Number of Connections:** An unusually high number of concurrent connections to a service can also be a sign of a DoS attack.
*   **Increased TCP SYN Packets:** A large number of SYN packets without corresponding ACKs suggests a SYN flood attack.

## Lab Objective

In this lab, you will learn how to use Prometheus to detect DoS attacks by:

*   Setting up Prometheus to collect network metrics.
*   Writing PromQL queries to identify DoS attack patterns.
*   Configuring alerting rules to trigger notifications when a DoS attack is detected.

## Lab Setup

1.  **Install Prometheus:** You can install Prometheus using various methods (e.g., Docker, binary download). Refer to the official Prometheus documentation for installation instructions: [https://prometheus.io/docs/prometheus/latest/installation/](https://prometheus.io/docs/prometheus/latest/installation/)

2.  **Install Node Exporter (for system metrics):** Node Exporter is a Prometheus exporter that exposes a wide range of system metrics, including network statistics. Install it on the machine you want to monitor (the target of the simulated DoS attack). First download the node-exporter extension: https://github.com/prometheus/node_exporter/releases/


    ```bash
    tar xvf node_exporter-*.linux-amd64.tar.gz
    cd node_exporter-*
    sudo cp node_exporter /usr/local/bin
    #This is optional to run as a service. You can just run it locally
    sudo useradd -rs /bin/false node_exporter
    sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
    sudo nano /etc/systemd/system/node_exporter.service
    ```

    Paste the following into the service file:

    ```
    [Unit]
    Description=Node Exporter
    After=network.target

    [Service]
    User=node_exporter
    ExecStart=/usr/local/bin/node_exporter

    [Install]
    WantedBy=multi-user.target
    ```

    Optinally, you can run it as a deamon:

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable node_exporter
    sudo systemctl start node_exporter
    ```

3.  **Configure Prometheus to scrape Node Exporter:** Edit the Prometheus configuration file (`prometheus.yml`) to add a scrape configuration for Node Exporter.

    ```yaml
    scrape_configs:
      - job_name: 'node'
        static_configs:
          - targets: ['<target_ip>:9100'] # Replace <target_ip> with the IP of the machine running Node Exporter. This could be your same IP if you are using 1 machine
    ```

4.  **Simulate a DoS attack:** Use a tool like the DNS scanner or ACK flooding in Project 1 to generate DoS traffic against the target machine.

## Lab Procedure

1.  **Start Prometheus and Node Exporter:** Ensure both services are running. Send some traffic to your host, of any kind (what if you install an Apache server?)

2.  **Access Prometheus Web UI:** Open your web browser and navigate to `http://<prometheus_ip>:9090` (replace `<prometheus_ip>` with the IP address of your Prometheus server. This could be your same IP if you are using 1 machine).

3.  **Explore Network Metrics:** Use PromQL queries to explore the network metrics exposed by Node Exporter. Some useful metrics include:

    *   `rate(node_network_receive_bytes_total[1m])`: Incoming network traffic rate in bytes per second.
    *   `rate(node_network_transmit_bytes_total[1m])`: Outgoing network traffic rate in bytes per second.
    *   `rate(node_network_receive_packets_total[1m])`: Incoming network traffic rate in packets per second.
    *   `rate(node_network_transmit_packets_total[1m])`: Outgoing network traffic rate in packets per second.
    *   `node_tcp_current_established`: Current number of established TCP connections. (find the equivalent command for UDP)
    *   `rate(node_network_tcp_in_segs_total[1m])`: Rate of incoming TCP segments. (find the equivalent command for UDP)
    *   `rate(node_network_tcp_out_segs_total[1m])`: Rate of outgoing TCP segments. (find the equivalent command for UDP)

4.  **Write PromQL Queries for DoS Detection:** Write PromQL queries to detect DoS attack patterns. For example:

    *   **High Traffic Volume:** `rate(node_network_receive_bytes_total[1m]) > 1000000` (detects incoming traffic exceeding 1MB/s).
    *   **High Number of Connections:** `node_tcp_current_established > 1000` (detects more than 1000 established TCP connections). (find the equivalent command for UDP)
    *   **High SYN Packet Rate (Requires more advanced setup with tcpdump metric collection):** This requires capturing TCP SYN packets with tcpdump and exporting the metrics to Prometheus. This is more complex but provides valuable insight.

5.  **Create Alerting Rules:** Configure Prometheus alerting rules to trigger alerts when the DoS detection queries evaluate to true. Edit the Prometheus configuration file (`prometheus.yml`) to add alerting rules.

    ```yaml
    rule_files:
      - rules.yml # Define rules in a separate file

    #Example rules.yml
    groups:
    - name: DoS_Detection
      rules:
      - alert: HighNetworkTraffic
        expr: rate(node_network_receive_bytes_total[1m]) > 1000000
        for: 1m # Alert if the condition is true for 1 minute
        labels:
          severity: critical
        annotations:
          summary: "High Network Traffic Detected"
          description: "Incoming network traffic exceeds 1MB/s on {{ $labels.instance }}"
    ```

6.  **Test Alerting:** Simulate a DoS attack and observe if the alerts are triggered in the Prometheus web UI and any configured alert receivers (e.g., email, Slack).

## Lab Tasks

1.  Experiment with different PromQL queries to detect various DoS attack patterns.
    - Check CPU usage. Is the CPU usage > 90% in 5 seconds?
    - Check HTTP requests per second
    - Check HTTP requests per 5 minutes
    - Check the amount of HTTP requests with error code 5.. in 5 minutes
    - Check for possible brute force attacks by checking the amount of ```authentication_failures_total``` in 5 minutes
2.  Adjust the thresholds in the queries and alerting rules to fine-tune the detection sensitivity. Give 5 different configurations and support why do you think those new configurations are useful
3.  Explore different alerting configurations (e.g., using different alert receivers, adding more annotations).
    - Give at least 2 examples
4.  Research other metrics that could be useful for detecting DoS attacks.
    - Give at least 3 different metrics and the queries you could use. Some examples are: Network Latency,  Disk IO Operations, number of TCP active connections.


## Discussion and Conclusion

Discuss your findings from the lab tasks. How effective is Prometheus for detecting DoS attacks? What are the advantages and limitations of this approach? What other tools or techniques could be used in conjunction with Prometheus for enhanced DoS detection and mitigation? (I want this in the video!)

## Further Research

*   Explore the use of other Prometheus exporters for collecting more specialized metrics (e.g., application-specific metrics).
*   Investigate how Prometheus can be integrated with other security tools and platforms.
*   Research advanced DoS attack techniques and how they can be detected using monitoring and analysis tools.