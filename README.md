Project OverviewUnlike traditional beginner cybersecurity projects that focus mainly on external network attacks, this project simulates a real enterprise Security Operations Center (SOC) environment focused on detecting suspicious internal user behavior.  
By deploying an isolated lab environment, this project emphasizes:
-Insider Threat Detection & Behavioral Monitoring   
-Endpoint Telemetry Analysis via Sysmon   
-PowerShell Abuse Detection (encoded commands)   
-SIEM Alert Investigation & Log Analysis   
-MITRE ATT&CK Mapping   
The lab demonstrates how SOC analysts monitor endpoints, analyze logs, investigate alerts, and identify potentially malicious user activities inside an organization.  

Architecture & Data Flow
<img width="1122" height="1402" alt="ChatGPT Image May 16, 2026, 07_00_54 PM" src="https://github.com/user-attachments/assets/ea18d473-b891-427f-a740-f3454682aaef" />

Technologies & Tools Used
<img width="628" height="235" alt="Screenshot 2026-05-16 183754" src="https://github.com/user-attachments/assets/d1ff6bd8-fdbc-4a6f-956f-64ca2ef6278c" />
Deployment & Setup Steps
Phase 1: Virtual Environment & Networking
1.Deployed four dedicated virtual machines on VMware Workstation: Kali Linux, Windows Server 2022, and Ubuntu Server.  
2.Configured internal networking and resolved connectivity hurdles:  
    -Verified local interface status using ip a.  
    -Fixed underlying DNS resolution and adapter communication issues to ensure internet routing where needed.  
    -Validated routing using ping 8.8.8.8.  
Phase 2: Wazuh SIEM Installation
1.Ran the automated script to deploy Wazuh Manager, Indexer, Dashboard, and Filebeat on the Ubuntu host: 
                        curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
                        sudo bash ./wazuh-install.sh -a
2.Accessed the web interface via https://<UBUNTU_IP> to confirm dashboard stability and administrative access.
3.Phase 3: Endpoint Telemetry Integration (Wazuh + Sysmon)
Installed the Wazuh Agent on Windows Server 2022 via PowerShell:
                        Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.5-1.msi -OutFile $env:temp\wazuh-agent.msi
                        msiexec.exe /i $env:temp\wazuh-agent.msi /q WAZUH_MANAGER='YOUR_UBUNTU_IP' WAZUH_AGENT_NAME='WindowsServer2022'
                        Restart-Service WazuhSvc
2.Downloaded Sysmon alongside the community-standard SwiftOnSecurity Configuration file for optimized, high-visibility telemetry filtering.  
3.Initialized Sysmon via Command Prompt/PowerShell: 
                        Sysmon64.exe -accepteula -i "C:\Users\Administrator\Downloads\sysmon-config-master\sysmon-config-master\sysmonconfig-export.xml"
4.Directed the Wazuh Agent to collect Sysmon telemetry by modifying C:\Program Files (x86)\ossec-agent\ossec.conf and appending the event channel location:
                        <localfile>
                          <location>Microsoft-Windows-Sysmon/Operational</location>
                          <log_format>eventchannel</log_format>
                        </localfile>
5.Restarted the agent service to apply changes: Restart-Service WazuhSvc.

Threat Simulation & SOC Analysis
The Attack Simulation
To simulate an insider threat performing defensive evasion, a base64-encoded command string was triggered through PowerShell on the monitored endpoint:
                        powershell -enc ZQBjAGgAbwAgAHQAZQBzAHQA
Critical Event Telemetry TrackedBy leveraging Sysmon, the SIEM monitored crucial core activities based on specific Event IDs:  
    -Event ID 1: Process Creation (e.g., tracking powershell.exe)   
    -Event ID 3: Network Connections   
    -Event ID 11: File Creation (detecting script drops in real time)   
    -Event ID 13: Registry Modifications   
    -Event ID 22: DNS Queries   
Investigation Breakdown: Sysmon Event ID 11
An alert triggered on the dashboard when a PowerShell execution dropped a script inside a temporary directory. 
<img width="377" height="179" alt="Screenshot 2026-05-16 185215" src="https://github.com/user-attachments/assets/9164cbc8-98f3-49c5-ac3d-91d4c4fa3a2f" />

Why was this flagged? 
Attackers frequently abuse PowerShell to bypass execution policies, and malicious tools or payloads are heavily dropped directly into volatile Temp directories to evade simple disk audits.

MITRE ATT&CK Framework Mapping
The triggered alerts correlated directly with the following adversary tactics and techniques:  
-T1059 (Execution): Command and Scripting Interpreter   
-T1059.001 (Execution): PowerShell usage   
-T1105 (Command and Control): Ingress Tool Transfer

Lab Visuals & Evidence
VMware Virtual Infrastructure
<img width="1915" height="1027" alt="Screenshot 2026-05-09 100644" src="https://github.com/user-attachments/assets/914e3c9c-2818-443e-b19a-b2a5f0b74ace" />
Wazuh Dashboard Active Home View
<img width="1919" height="917" alt="Screenshot 2026-05-09 100706" src="https://github.com/user-attachments/assets/ee78eba2-7d76-4fe5-a56a-401417fbc145" />
Endpoint Agent Connectivity Status
<img width="1919" height="909" alt="Screenshot 2026-05-09 100717" src="https://github.com/user-attachments/assets/2fc42694-9ff9-42b8-a89e-32ab21179bc7" />
Logs 
<img width="1916" height="918" alt="Screenshot 2026-05-09 140719" src="https://github.com/user-attachments/assets/5f0eea49-1834-4fd7-8667-e06545e92d8a" />
Sysmon operational logs 
<img width="1916" height="918" alt="Screenshot 2026-05-09 140719" src="https://github.com/user-attachments/assets/ffedb8fd-20d3-4171-b4c5-4593a3347fed" />
