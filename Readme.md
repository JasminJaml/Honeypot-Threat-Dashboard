# Interactive Honeypot & Threat Analysis Dashboard

This project is a 4-container, Docker-based threat intelligence system designed to capture, parse, and visualize real-time cyberattack data from a live Cowrie honeypot.

The system is deployed on an AWS EC2 instance and uses the ELK Stack (Elasticsearch, Kibana) with Filebeat to create a dashboard of attacker Tactics, Techniques, and Procedures (TTPs).

## System Architecture

The system evolved from a 3-container to a 4-container architecture to resolve critical data-flow challenges:

### Container Components
- **Cowrie (Honeypot)**: Internet-exposed sensor capturing SSH/Telnet attacks, writing JSON logs to shared volume
- **Filebeat (Log Shipper)**: Monitors and parses honeypot logs, ships structured data to Elasticsearch
- **Elasticsearch (Database Engine)**: Indexes and stores attack data for fast search and analysis
- **Kibana (Visualization)**: Real-time dashboard displaying threat intelligence through interactive charts

#### Data Pipeline
Attacker → Cowrie → shared_logs/cowrie.json → Filebeat → Elasticsearch → Kibana Dashboard

## Technical Features & Configuration

- **Orchestration**: The entire 4-container stack is deployed and managed with a single docker-compose.yml file.

- **Data Processing**: Filebeat with custom configuration using `json.keys_under_root: true` to properly extract and parse JSON fields from honeypot logs.

- **Persistence**: A shared "bind mount" folder (~/shared_logs) is used for log persistence and inter-container communication, which proved more reliable than named volumes for this use case.

- **Security**: The AWS Security Group is configured to be restrictive, only exposing the necessary ports (SSH, Kibana) to the admin's IP and the honeypot port to the public.

## Core Troubleshooting Challenge: Elasticsearch Mapping Conflict

A major part of this project was diagnosing and solving a "silent failure" where all containers were healthy, but no parsed data appeared in Kibana.

**Problem**: Early, broken data from a misconfigured Filebeat created a "corrupted schema" (Index Template) in Elasticsearch. This "corrupted memory" caused Elasticsearch to reject all new, correctly parsed fields.

**Solution**: The system was fixed by following a precise, real-world troubleshooting plan:
1. Stopped the filebeat container
2. Manually deleted the corrupted index (`curl -X DELETE "localhost:9200/filebeat-*"`)
3. Manually deleted the corrupted "blueprint" (`curl -X DELETE "localhost:9200/_index_template/filebeat-7.17.15"`)
4. Relaunched the system, which forced Elasticsearch to create a new, correct schema from the good data

## Final Result: The Dashboard

The final Kibana dashboard visualizes key threat intelligence in real-time, including:
- Top 10 most common usernames
- Top 10 most common passwords

<img width="960" height="510" alt="33" src="https://github.com/user-attachments/assets/6d51a86c-c9d4-4f70-89ff-50da1e087d3b" />









