# AWS CloudWatch Logs Insights - VPC Flow Logs Incident Response

This directory contains practical AWS CloudWatch Logs Insights queries for investigating network incidents, performance anomalies, and security events using VPC Flow Logs.

---

## Scenario 1: Outbound Traffic Analysis
**Ticket:** Inspection of outbound HTTP/HTTPS connections originating from a private application instance (`10.0.3.45`).
Scenario: The security team has reported that instances within the Private Subnet are attempting to download external files over HTTP (80) and HTTPS (443) ports. You need to generate a quick report of the last 20 allowed access attempts originating from the private application IP 10.0.3.45, showing the timestamp, destination IP, destination port, and protocol, sorted from most recent to oldest.

fields @timestamp, dstAddr, dstPort, protocol
| filter srcAddr = '10.0.3.45' and action = 'ACCEPT' and (dstPort = 80 or dstPort = 443)
| sort @timestamp desc
| limit 20

---

## Scenario 2: Internal Port Scan Detection
**Ticket:** Internal host (`10.0.1.99`) suspected of port scanning or brute-force attempts across internal subnets.
Incident: The SecOps team suspects that a server within the VPC (IP 10.0.1.99) has been compromised and is performing a port scan or attempting a brute-force attack against multiple internal servers on the 10.0.0.0/16 network.

Your mission:
Create a query using `stats` that analyzes only the blocks (REJECT) caused by the attacking IP 10.0.1.99.


filter action = 'REJECT' and srcAddr = '10.0.1.99'
| stats count(*) as total_bloqueios by dstAddr, dstPort
| sort total_bloqueios desc

---

## Scenario 3: Database Connectivity Failure
**Ticket #8492:** Checkout microservice (`10.0.5.22`) experiencing connection drops when communicating with the database cluster subnet (`10.0.10.0/24`).
Ticket #8492 — PagerDuty
Service: Checkout / Payments Microservice
Severity: High (P1)

Description: The Checkout microservice, hosted on the instance with IP 10.0.5.22, is experiencing unexplained latency spikes and packet loss when attempting to persist data to the database cluster (IP range 10.0.10.0/24).

The DevOps team suspects the application is attempting to re-establish connections on incorrect ports or is encountering intermittent network blocks when writing data.

We need an analysis of the network logs to understand which database servers are affected, which ports are involved in this failed traffic, and the volume of these rejections.

filter srcAddr = '10.0.5.22' and action = 'REJECT' and dstAddr like /10\.0\.10\./
| stats count(*) as total_bloqueios by dstAddr, dstPort
| sort total_bloqueios desc

---

## Scenario 4: Command & Control Investigation (Timeline Analysis)
**Ticket #9102:** Suspicious outbound activity from `10.0.2.100` targeting the management subnet `172.16.50.0/24`.
Ticket #9102 — Security Team (SOC) Alert
Severity: Medium (P2)

Description: Our edge sensors detected a suspicious communication attempt originating from an internal server on our private subnet (IP 10.0.2.100). The server is suspected of attempting to contact a range of external command-and-control servers or performing a port scan on the management subnet (172.16.50.0/24).

The incident response team needs to identify the destination ports being targeted or probed, the volume of blocked attempts (REJECT) per port, and the timestamps of the server's first and last attempts in order to reconstruct the attack timeline.

filter srcAddr = '10.0.2.100' and action = 'REJECT' and dstAddr like /172\.16\.50\./
| stats earliest(@timestamp) as inicio, latest(@timestamp) as ultimo, count(*) as total_bloqueios by dstAddr, dstPort
| sort total_bloqueios desc

---

## Scenario 5: Data Exfiltration & FinOps Analysis
**Ticket #1042:** Auth service (`10.0.1.50`) transferring anomalous data volume over accepted outbound connections.
Ticket #1042 — Production Incident
Service: Authentication / OAuth Microservice
Severity: P1 (Critical)

Description: The security team (SecOps) received a SIEM alert indicating that our Authentication server instance (IP 10.0.1.50) may have been compromised and is attempting data exfiltration (massive data transfer out of the network) via successfully established connections.

The incident response team requires an urgent observability report to understand where this data is going, the total volume of data transferred to each destination, and the number of connections made, with results sorted to immediately highlight the top data recipient at the top of the list.

filter srcAddr = '10.0.1.50' and action = 'ACCEPT'
| stats sum(bytes) as total_transferido, count(*) as total_acessos_aceitos by dstAddr, dstPort
| sort total_transferido desc
