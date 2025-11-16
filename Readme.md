Interactive Honeypot & Threat Analysis Dashboard

This project is a 4-container, Docker-based threat intelligence system designed to capture, parse, and visualize real-time cyberattack data from a live Cowrie honeypot.

The system is deployed on an AWS EC2 instance and uses the ELK Stack (Elasticsearch, Kibana) with Filebeat to create a dashboard of attacker Tactics, Techniques, and Procedures (TTPs).

Final System Architecture

The project was re-architected from a simple 3-container model to a more robust 4-container system to solve data-flow challenges. The final architecture is:

Cowrie (The Honeypot): The sensor, exposed to the internet. It catches SSH/Telnet attacks and writes them to a cowrie.json log file in a shared volume.

Filebeat (The "Delivery Truck"): The log shipper. It monitors the shared log file, parses the NDJSON (Newline Delimited JSON) data, and ships the parsed fields to Elasticsearch.

Elasticsearch (The Database): The "engine." It receives the parsed JSON from Filebeat and indexes it, making it searchable.

Kibana (The Dashboard): The "monitor." It reads from Elasticsearch and displays the data in real-time visualizations (pie charts, bar charts, etc.).

Data Flow:
Attacker → Cowrie (Container) → shared_logs/cowrie.json → Filebeat (Container) → Elasticsearch (Container) → Kibana (Dashboard)

Technical Features & Configuration

Orchestration: The entire 4-container stack is deployed and managed with a single docker-compose.yml file.

Data Pipeline: Uses Filebeat with a custom filebeat.yml to correctly parse ndjson log files using the ndjson parser.

Persistence: A shared "bind mount" folder (~/shared_logs) is used for log persistence and inter-container communication, which proved more reliable than named volumes for this use case.

Security: The AWS Security Group is configured to be restrictive, only exposing the necessary ports (SSH, Kibana) to the admin's IP and the honeypot port to the public.

Core Troubleshooting Challenge: Elasticsearch Mapping Conflict

A major part of this project was diagnosing and solving a "silent failure" where all containers were healthy, but no parsed data appeared in Kibana.

Problem: Early, broken data from a misconfigured Filebeat created a "corrupted schema" (Index Template) in Elasticsearch. This "corrupted memory" caused Elasticsearch to reject all new, correctly parsed fields.

Solution (Option A): The system was fixed by following a precise, real-world troubleshooting plan:

Stopped the filebeat container.

Manually deleted the corrupted index (curl -X DELETE "localhost:9200/filebeat-*")

Manually deleted the corrupted "blueprint" (curl -X DELETE "localhost:9200/_index_template/filebeat-7.17.15").

Relaunched the system, which forced Elasticsearch to create a new, correct schema from the good data.

Final Result: The Dashboard

The final Kibana dashboard visualizes key threat intelligence in real-time, including:

Top 10 most common usernames.

Top 10 most common passwords.

<img width="960" height="540" alt="33" src="https://github.com/user-attachments/assets/dcfbd2d8-23e3-4e7d-9583-2fc32b10cef1" />



