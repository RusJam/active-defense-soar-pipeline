**Active Defense SOAR Pipeline: Automated Threat Detection, AI Enrichment, and Network Containment**

Project Overview
This project documents the design and implementation of a closed-loop incident response ecosystem. The pipeline bridges the gap between threat detection and automated remediation by handling security telemetry across a distributed network. When a high-fidelity threat is detected on an endpoint, the system automatically isolates the compromised host at the network layer, enriches the alert context using a local large language model, logs a structured ticket into an enterprise case management system, and dispatches a detailed notification to the security team. By eliminating human latency from the containment loop, the system reduces the mean time to remediate from minutes to less than seven seconds.

**System Architecture and Data Flow**
The following map illustrates how security telemetry cascades through the environment, shifting from initial exploitation to automated defense and analyst notification.

[ Attacker Machine ] 
       │
       ▼ (Active Directory Enumeration / Brute-Force)
[ Windows Target Endpoint ] ──► [ Local Firewall Rule ] ──► (Inbound Traffic Dropped)
       │
       ▼ (Security Event Logs)
[ Centralized Wazuh SIEM ] 
       │
       ▼ (Raw JSON Webhook Alert)
[ Dockerized n8n Orchestrator ]
       │
       ├─► [ Ollama / Llama 3 ] ──► (Contextual AI Threat Triage)
       │
       ├─► [ TheHive 5 Portal ] ──► (Structured Incident Investigation Case)
       │
       ▼
[ Discord SecOps Channel ] ────► (Polished Analyst Notification Broadcast)

**Threat Simulation**: Network enumeration and brute-force authentication cycles are executed against a Windows target host.

**Telemetry Collection**: The Windows Event Viewer captures the authentication failures and passes the raw logs to a centralized Wazuh SIEM manager.

**Automated Containment**: The SIEM engine recognizes the attack signature and immediately deploys an active response script on the target host. This script uses the native Windows firewall to drop all incoming traffic from the attacking IP address for a specified cooldown window.

**Orchestration and Routing**: Simultaneously, the SIEM manager dispatches a structured webhook payload to an n8n automation engine.

**Local AI Enrichment**: The automation engine strips out formatting limitations and passes the alert metadata to a local Ollama instance running Llama 3 to generate a targeted security summary without exposing sensitive infrastructure data to the public cloud.

**Case Generation**: The enriched data is written directly to a newly initialized incident case inside TheHive 5 dashboard.

**ChatOps Notification**: The final triaged output is pushed to a dedicated security operations communication channel.

Core Technology Stack
SIEM and Detection Architecture: Wazuh Indexer, Dashboard, and Manager using centralized endpoint monitoring agents.

Automation Workflow Engine: Containerized n8n instances deployed via Docker.

Local Artificial Intelligence: Ollama executing the Llama 3 large language model locally.

Incident Management Platform: TheHive 5 case repository backed by Cassandra storage and MinIO.

Target Environment: Windows Server operating system configured within a controlled virtual network.


**Operational Impact and Efficiency Metrics**
In a typical security operations center, an analyst must manually pivot across multiple monitoring interfaces, threat intelligence feeds, and endpoint management tools to validate an alert, isolate a host, and document the investigation. This architecture automates the entire triage and containment process, shifting the timeline from minutes to seconds.

Manual Detection and Containment: Generally requires 20 to 40 minutes depending on analyst availability, internal investigation processes, and system pivoting.

Automated Pipeline Execution: Completes detection, host isolation via native firewall rules, local AI triage, case creation, and team notification in 5 to 7 seconds.

This represents a significant reduction in the window of opportunity for an attacker, preventing lateral movement or domain pivoting before a human analyst even opens the ticket.

Technical Challenges and Engineering Solutions
Developing distributed security integrations requires working through complex environmental barriers. Below are the key engineering hurdles encountered during the implementation of this pipeline and how they were resolved.

1. The Defensive Feedback Loop Paradox
During automated load testing, secondary attack loops failed to trigger downstream automation events, resulting in empty data readouts within the orchestration dashboard. Investigation revealed that the active defense mechanisms were working exactly as intended. The initial automated firewall rule completely blocked the attacking IP address at the network boundary. Because the subsequent traffic never reached the Windows operating system, no new event logs were generated, preventing further alert escalations. This was solved by configuring a clear, temporary cooldown timer within the SIEM manager and implementing manual firewall flush procedures to allow for predictable testing cycles.

2. Structural Parsing Errors in Configuration Trees
Modifications to core SIEM active-response blocks initially caused the main analysis daemon to crash upon boot with generic configuration errors. This behavior occurs when structural markers or tag nests are broken within central configuration files. The issue was diagnosed by utilizing internal sandbox evaluation binaries to run offline syntax validations, which allowed for the isolation and removal of duplicate configuration parameters inherited from old baseline deployment templates.

3. Data Isolation and Scope Limits in Automation Nodes
Downstream ticketing steps and communication notifications were initially receiving empty text variables, forcing the local language model to hallucinate details using generic training data. In this specific orchestration environment, data evaluation default expressions look only at the output of the immediate preceding node. Because intermediate filtering and tagging nodes were stripping away the original event payload, the text fields arrived at the enrichment phase completely blank. This was resolved by refactoring the pipeline code to use absolute node referencing, forcing downstream nodes to draw data directly from the root webhook ingestion point regardless of intermediate steps.

System Validation
Active Host Isolation Policy
The automated containment shield can be validated directly from the target host by inspecting the active firewall rule policies established by the endpoint agent:
Get-NetFirewallRule | Where-Object { $_.DisplayName -like "*Wazuh*" } | Get-NetFirewallPortFilter

incident Escalation Sample
The following output demonstrates a verified alert payload routed cleanly through the local language model, utilizing absolute node references to eliminate placeholder information and map accurately to live environment hostnames:

Source SIEM: Wazuh Manager
Target System: Win-Target-01

Llama 3 Analysis: The log indicates a high-volume logon failure using an unknown username or bad password on Win-Target-01. This resembles a brute-force or credential-spraying attempt and poses an immediate security risk to the environment.


