# SOC-Lab-IDS-IPS-Wazuh


 The proposed project will provide a fully working example of IDS/IPS implementation using Ubuntu server.
 It includes configuration of Wazuh SIEM to collect logs from various sources, monitor events, and detect potential #security threats.
This project showcases a full-fledged hands-on implementation of the IDS and IPS system on Ubuntu Server, with Wazuh SIEM used for centralizing logging and monitoring systems.
 This lab is aimed at beginners and those interested in becoming SOC Analysts to learn more about practical security monitoring and threat detection.

This lab comprises the steps for configuring and integrating tools to perform a hands-on approach for detecting attacks in a simulated lab environment.
# 🔒 Key Focus Points: 

## Security Monitoring 
## Threat Detection 
## Logging and Analysis 
## IDS & IPS Integration 
## Deployment of Wazuh SIEM

# ⚙️ Features for Your Project (Mandatory)
1. Ubuntu Server Setup
2. IDS Setup (using Snort/Suricata)
3. IDS/IPS Mode Activation
4. Wazuh SIEM Integration
5. Collection and Analysis of Logs
6. Monitoring of Alerts through Dashboard
7. Attack Simulation and Testing
# 🧰 Tools Needed
1. Wazuh (SIEM tool)
2. Ubuntu Server
3. Snort / Suricata (IDS/IPS tool)
4. Elastic Stack (if needed to be used)
#  Use Case
1. Practice Lab for SOC Analyst
2. Training Blue Teams
3. Threat Detection Training
4. Monitoring of Logs
5. Beginners Lab for Cybersecurity
# 💡 Additional Tips (to Make It Github Ready)
1. Include a screenshot (Wazuh dashboard) in your README
2. Include attack test results
3. Include step-by-step guide
4. Include a diagram for professional look



# Use Case in IDS/IPS

   # IDS
# Installation
It's assumed that you run a recent Ubuntu release as the official PPA can then be used for the installation. To install the latest stable Suricata version, follow the steps:

# Install Suricata 

<pre> sudo apt update </pre>
<pre> sudo apt install suricata -y </pre>
<pre> sudo add-apt-repository ppa:oisf/suricata-stable </pre>
<pre> sudo apt update </pre>
<pre> sudo apt install suricata -y </pre>
<pre> sudo apt-get install software-properties-common </pre>
<pre> sudo add-apt-repository ppa:oisf/suricata-stable </pre>
<pre> sudo apt update </pre>
<pre> sudo apt install suricata jq </pre>

# Chack Suricata install 

<pre> sudo tail -f /var/log/suricata/stats.log </pre>
<pre> sudo suricata -T -c /etc/suricata/suricata.yaml -v </pre>
<pre>  sudo systemctl status suricata </pre>


# Service Enable/Start
<pre> sudo systemctl enable suricata </pre>
<pre> sudo systemctl start suricata </pre>
<pre> sudo systemctl status suricata </pre>


*Use that information to configure Suricata*
<pre> sudo nano /etc/suricata/suricata.yaml </pre>

There are many possible configuration options, we focus on the setup of the HOME_NET variable and the network interface configuration. The HOME_NET variable should include, in most scenarios, the IP address of the monitored interface and all the local networks in use. The default already includes the RFC 1918 networks. In this example 10.0.0.23 is already included within 10.0.0.0/8. If no other networks are used the other predefined values can be removed.  

In this example the interface name is enp1s0 so the interface name in the af-packet section needs to match. An example interface config might look like this:

Capture settings:

af-packet:  
    - interface: ens33  
      cluster-id: 99  
      cluster-type: cluster_flow  
      defrag: yes  
      tpacket-v3: yes  




# Suricata status and configuration file : 
<pre> sudo systemctl status suricata </pre>

# Configuration file : 
<pre> ip a </pre>
<pre> sudo nano /etc/suricata/suricata.yaml </pre>

*Ip change your ip*
HOME_NET: "[192.168.33.137/24"  
EXTERNAL_NET: "any"  
af-packet change  - interface: ens33 (your interface)  
sudo systemctl restart suricata  

*Default rules file parth* 
<pre> /var/lib/suricata/rules </pre>


# Configuration file 

<pre> sudo nano /var/ossec/etc/ossec.conf  </pre>

Change :  
 - eve-log:  
      enabled: yes  
      filetype: regular #regular|syslog|unix_dgram|unix_stream|redis  
      filename: eve.json  
      types: - alert
            - dns  
            - http
            - tls
            - flow

# Local rules setup :
<pre> sudo nano /var/lib/suricata/rules/local.rules </pre>
*Rules*

<pre> Alert icmp any any -> any any (msg:”ICMP Ping Detected”: itype:8; sid:1000001; rev:1;) </pre>
<pre> alert icmp any any -> $HOME_NET any (msg:"ALERT: ICMP Flood detected - 10+ packets in 10s"; threshold:type limit, track by_src, count 10, seconds 10; sid:1000001; rev:1;) </pre>
<pre> alert icmp any any -> $HOME_NET any (msg:"CRITICAL: Aggressive ICMP Flood - 20+ packets in 4s"; threshold:type limit, track by_src, count 20, seconds 4; sid:1000003; rev:1;)</pre>


*Add*

<pre> /var/ossec/etc/ossec.conf </pre>

<ossec_config>  
  <localfile>  
    <log_format>json</log_format>  
    <location>/var/log/suricata/eve.json</location>  
  </localfile>  
</ossec_config>  



# Permission Fix (socket directory issue)
<pre> sudo mkdir -p /var/run/suricata </pre>
<pre> sudo chown suricata:suricata /var/run/suricata </pre>
<pre> sudo systemctl restart suricata </pre>
<pre> sudo nano /etc/apt/sources.list.d/wazuh.list </pre>

# Log chack 
<pre> sudo tail -f /var/log/suricata/fast.log </pre>


# IPS


*Configuration file parth* 

<pre> sudo nano /var/ossec/etc/ossec.conf </pre>
*af-packet*

# Surocata rules file read
<pre> sudo i </pre>
<pre> cd /var/lib/suricata/rules/ </pre>
<pre> nano suricata.rules </pre>


# Manually suricata log show  

<pre> cd /var/log/suricata/</pre> 
<pre> nano fast.log </pre>
<pre> cd /etc/suricata/rules</pre>
<pre> nano local.rules </pre>

*Rules*
<pre> drop tcp any any -> any 80 (msg:"Test"; sid:1000001;) </pre>


# Suricata mane file rules gula ki ki ase egula e dekar jonno  
Agent e  

<pre> /etc/suricata/rules </pre>

*Rouls*
<pre> cat emerging-icmp_info.rules </pre>
<pre> cd nano /etc/suricata/suricata.yaml</pre>
*nfq:*

<pre> /var/lib/suricata/rules/ nano local.rules </pre>

<pre> drop icmp any any -> $HOME_NET any (msg:"CRITICAL: ICMP FLOOD Blocked"; threshold:type limit, track by_src, count 20, seconds 4; sid:1000003; rev:1;) </pre>
<pre> drop icmp any any -> $HOME_NET any (msg:"ICMP Flood Detected: 5 pings exceeded, blocking source"; threshold:type limit, track by_src, count 5, seconds 10; sid:1000005; rev:1;) </pre>
<pre> drop icmp any any -> $HOME_NET any (msg:"DROP the ICMP PACKET"; sid:1000002; rev:1;) </pre>


*Log show*
<pre> sudo suricata -T -c /etc/suricata/suricata.yaml -S /var/lib/suricata/rules/local.rules </pre>





# Manumaly Log Show in Machin

1. Main log in suricata
<pre> sudo tail -f /var/log/suricata/suricata.log</pre>
<pre> sudo tail -f /var/log/suricata/stats.log </pre>

2. Real-time log follow করতে
<pre> sudo tail -f /var/log/suricata/suricata.log </pre>

4. JSON log দেখতে (SIEM বা Wazuh integration-এর জন্য)
<pre> sudo tail -f /var/log/suricata/eve.json </pre>









