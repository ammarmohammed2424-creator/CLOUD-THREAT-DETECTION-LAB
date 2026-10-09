#cloud-threat-detection-lab

A hybrid SOC detection lab utilizing a local Wazuh SIEM to monitor an AWS EC2 instance over an encrypted WireGuard VPN, detecting and alerting on Nmap port scanning activity (MITRE ATT&amp;CK T1046).

<img width="860" height="686" alt="Screenshot 2026-10-08 102935" src="https://github.com/user-attachments/assets/2eaf8235-c0a1-4858-a8ac-d3d53ec0bc88" />


Step 1: Connecting to AWS EC2 via SSH

Established an SSH connection from the local management host to the remote AWS EC2 Ubuntu instance (ubuntu@<EC2_PUBLIC_IP>) using an SSH private key. Verified system initialization and base IPv4 interface configuration (<EC2_PRIVATE_IP>)

<img width="1688" height="900" alt="Screenshot 2026-10-07 142210" src="https://github.com/user-attachments/assets/94cbc476-36a9-4a6b-983c-362520bb6c03" />


Step 2: Preparing Packages and Repositories on EC2

Updated local package repositories (sudo apt update) and attempted installation of required networking utilities (wireguard-tools) on the EC2 instance to prepare for VPN tunnel setup.

<img width="1050" height="732" alt="Screenshot 2026-10-08 110758" src="https://github.com/user-attachments/assets/f6a9eb8f-5c5b-41d2-a137-53228dfcea3d" />

Step 3: Installing WireGuard Tools on EC2

Resolved package manager locks and successfully installed wireguard-tools on the AWS EC2 instance to enable kernel-level encrypted tunneling.

<img width="688" height="594" alt="Screenshot 2026-10-08 110250" src="https://github.com/user-attachments/assets/e4783d7c-8485-45b5-ab44-d5ca75876194" />


Step 4: Verifying Local Wazuh Manager Host

Checked system status and local interface bindings on the local Wazuh SIEM Virtual Machine (<WAZUH_VM_IP> NAT interface) prior to initiating the WireGuard peer connection.

<img width="874" height="343" alt="Screenshot 2026-10-08 122009" src="https://github.com/user-attachments/assets/d74ff589-695f-4700-a480-754979a75c44" />



<img width="909" height="327" alt="Screenshot 2026-10-08 122004" src="https://github.com/user-attachments/assets/b2c8b5a9-3ae7-436c-87f7-f6826164f31f" />


Step 5: Configuring WireGuard VPN Peer Files (wg0.conf)

Configured /etc/wireguard/wg0.conf on both nodes:EC2 Interface (<VPN_EC2_IP>/24): Set up as the WireGuard server endpoint listening on UDP port 51820.   Wazuh VM Interface (<VPN_WAZUH_IP>/24): Set up as the client peer pointing to the EC2 endpoint (<EC2_PUBLIC_IP>:51820) with PersistentKeepalive = 25 to maintain active NAT traversal. 

<img width="753" height="301" alt="Screenshot 2026-10-08 121948" src="https://github.com/user-attachments/assets/9f8abc48-087e-4ca1-8349-4b04311d4886" />


Step 6: Activating WireGuard Service and Verifying Tunnel

Started the WireGuard service (sudo systemctl enable --now wg-quick@wg0) and verified point-to-point tunnel reachability by executing an internal ping test from the Wazuh VM (<VPN_WAZUH_IP>) to the EC2 private VPN IP (<VPN_EC2_IP>). Result: 4 packets transmitted, 0% packet loss.

<img width="686" height="234" alt="Screenshot 2026-10-08 144432" src="https://github.com/user-attachments/assets/56b30def-62b5-40b6-90b1-da9256267d22" />



Step 7: Simulating Port Scan Reconnaissance from Kali Linux

Simulated an external attack by executing an Nmap TCP SYN stealth port scan (sudo nmap -sS -Pn) from the Kali Linux VM against the EC2 public IP (<EC2_PUBLIC_IP>). The scan revealed open SSH service (22/tcp) and dropped/filtered closed service ports.

<img width="1851" height="685" alt="Screenshot 2026-10-08 145521" src="https://github.com/user-attachments/assets/7725f071-0be6-4732-a617-28c5a7c0fa29" />


Step 8: Analyzing Security Events on Wazuh SIEM Dashboard

Evaluated parsed security alerts in real time on the Wazuh Dashboard (https://<WAZUH_DASHBOARD_IP>). The Wazuh Agent running on EC2 (agent.id: 006, agent.ip: <VPN_EC2_IP>) captured firewall block logs and transmitted them across the WireGuard VPN tunnel to trigger rule alerts mapped to attacker telemetry (data.srcip: <ATTACKER_IP>).

<img width="1828" height="641" alt="Screenshot 2026-10-08 162635" src="https://github.com/user-attachments/assets/ff8508fd-afe7-44ab-96ab-07eb5124b927" />


Step 9: Real-Time Security Alerts Summary Table

Monitored live threat telemetry on the Wazuh SIEM Security Alerts dashboard. The table captures real-time SSH credential access attempts on the EC2 instance, automatically categorizing unauthorized username probes (Rule 5710) under MITRE ATT&CK techniques T1110.001 (Password Guessing) and T1021.004 (SSH Remote Services), alongside automated Rootcheck host anomaly detection logs (Rule 510).
