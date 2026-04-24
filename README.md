# SOC-Lab-IDS-IPS-Wazuh

`Creat by mohammad zisadul islam`
`Update : 21/02/2026`
`IDS&IPS Setup and integratiom`


**The proposed project will provide a fully working example of IDS/IPS implementation using Ubuntu server.
It includes configuration of Wazuh SIEM to collect logs from various sources, monitor events, and detect potential #security threats.
This project showcases a full-fledged hands-on implementation of the IDS and IPS system on Ubuntu Server, with Wazuh SIEM used for centralizing logging and monitoring systems.
This lab is aimed at beginners and those interested in becoming SOC Analysts to learn more about practical security monitoring and threat detection.**

`This lab comprises the steps for configuring and integrating tools to perform a hands-on approach for detecting attacks in a simulated lab environment.`

# 🔒 Key Focus Points: 

### Security Monitoring 
### Threat Detection 
### Logging and Analysis 
### IDS & IPS Integration 
### Deployment of Wazuh SIEM

# ⚙️ Features for Your Project (Mandatory)
1. Ubuntu Server Setup
2. IDS Setup (using Snort/Suricata)
3. IDS/IPS Mode Activation
4. Wazuh SIEM Integration
5. Collection and Analysis of Logs
6. Monitoring of Alerts through Dashboard
7. Attack Simulation and Testing
# 🧰 Tools Needed
- Wazuh (SIEM tool)
- Ubuntu Server
- Snort / Suricata (IDS/IPS tool)
- Elastic Stack (if needed to be used)


#  Use Case
- Practice Lab for SOC Analyst
- Training Blue Teams
- Threat Detection Training
- Monitoring of Logs
- Beginners Lab for Cybersecurity

# 💡 Additional Tips (to Make It Github Ready)
- Include a screenshot (Wazuh dashboard) in your README
- Include attack test results
- Include step-by-step guide
- Include a diagram for professional look



# Use Case in IDS/IPS

   ## IDS
# Installation
`It's assumed that you run a recent Ubuntu release as the official PPA can then be used for the installation. To install the latest stable Suricata version, follow the steps:`

# Install Suricata 

```bash
sudo apt update </pre>
```
```bash
sudo apt install suricata -y </pre>
```
```bash
sudo add-apt-repository ppa:oisf/suricata-stable
 sudo apt update 
sudo apt install suricata -y
sudo apt-get install software-properties-common
 sudo add-apt-repository ppa:oisf/suricata-stable
 sudo apt update 
 sudo apt install suricata jq
```

# Chack Suricata install 
```
 sudo tail -f /var/log/suricata/stats.log 
 sudo suricata -T -c /etc/suricata/suricata.yaml -v 
 sudo systemctl status suricata
```

# Service Enable/Start
```
 sudo systemctl enable suricata 
 sudo systemctl start suricata 
 sudo systemctl status suricata
```

*Use that information to configure Suricata*
``` sudo nano /etc/suricata/suricata.yaml```

<p>There are many possible configuration options, we focus on the setup of the HOME_NET variable and the network interface configuration. The HOME_NET variable should include, in most scenarios, the IP address of the monitored interface and all the local networks in use. The default already includes the RFC 1918 networks. In this example 10.0.0.23 is already included within 10.0.0.0/8. If no other networks are used the other predefined values can be removed.</p>  

`In this example the interface name is enp1s0 so the interface name in the af-packet section needs to match. An example interface config might look like this:`

### Capture settings:

```
af-packet:  
    - interface: ens33  
      cluster-id: 99  
      cluster-type: cluster_flow  
      defrag: yes  
      tpacket-v3: yes  
```



## Suricata status and configuration file 
```sudo systemctl status suricata```

## Configuration file  
```
ip a
sudo nano /etc/suricata/suricata.yaml
```

*Ip change your ip*

```
HOME_NET: "[192.168.33.137/24"  
EXTERNAL_NET: "any"  
af-packet change  - interface: ens33 (your interface)
```
``` 
sudo systemctl restart suricata  
```

*Default rules file parth* 
``` /var/lib/suricata/rules
```


## Configuration file 

```sudo nano /var/ossec/etc/ossec.conf
```

```
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
```

## Local rules setup
```
sudo nano /var/lib/suricata/rules/local.rules </pre>
```
*Rules*

```
Alert icmp any any -> any any (msg:”ICMP Ping Detected”: itype:8; sid:1000001; rev:1;)
```
```
alert icmp any any -> $HOME_NET any (msg:"ALERT: ICMP Flood detected - 10+ packets in 10s"; threshold:type limit, track by_src, count 10, seconds 10; sid:1000001; rev:1;)
```
```
alert icmp any any -> $HOME_NET any (msg:"CRITICAL: Aggressive ICMP Flood - 20+ packets in 4s"; threshold:type limit, track by_src, count 20, seconds 4; sid:1000003; rev:1;)
```

*Add*

```
/var/ossec/etc/ossec.conf
```
```
<ossec_config>  
  <localfile>  
    <log_format>json</log_format>  
    <location>/var/log/suricata/eve.json</location>  
  </localfile>  
</ossec_config>  
```


## Permission Fix (socket directory issue)

```
sudo mkdir -p /var/run/suricata
```
```
sudo chown suricata:suricata /var/run/suricata
```
```
sudo systemctl restart suricata
```
```
sudo nano /etc/apt/sources.list.d/wazuh.list </pre>
```
## Log chack 
```
sudo tail -f /var/log/suricata/fast.log
```


## IPS


### Configuration file parth 

```sudo nano /var/ossec/etc/ossec.conf
```
*af-packet*

## Surocata rules file read
```
sudo i
```
```
cd /var/lib/suricata/rules/
```
```
nano suricata.rules
```


## Manually suricata log show  
```
cd /var/log/suricata/
```
```
nano fast.log
```
```
cd /etc/suricata/rules
 ```
```
nano local.rules
```
*Rules*
```
drop tcp any any -> any 80 (msg:"Test"; sid:1000001;)
```


## Suricata mane file rules gula ki ki ase egula e dekar jonno  
`Agent e`

```
/etc/suricata/rules
```

*Rouls*
```
cat emerging-icmp_info.rules
```
```
cd nano /etc/suricata/suricata.yaml
```
*nfq:*

```
/var/lib/suricata/rules/ nano local.rules
```

```
drop icmp any any -> $HOME_NET any (msg:"CRITICAL: ICMP FLOOD Blocked"; threshold:type limit, track by_src, count 20, seconds 4; sid:1000003; rev:1;)
```
```
drop icmp any any -> $HOME_NET any (msg:"ICMP Flood Detected: 5 pings exceeded, blocking source"; threshold:type limit, track by_src, count 5, seconds 10; sid:1000005; rev:1;)
```
```
drop icmp any any -> $HOME_NET any (msg:"DROP the ICMP PACKET"; sid:1000002; rev:1;)
 ```


*Log show*
```
sudo suricata -T -c /etc/suricata/suricata.yaml -S /var/lib/suricata/rules/local.rules
```





## Manumaly Log Show in Machin

`1. Main log in suricata`
```
sudo tail -f /var/log/suricata/suricata.log
```
```
sudo tail -f /var/log/suricata/stats.log
```

`2. Real-time log follow করতে`
```
sudo tail -f /var/log/suricata/suricata.log
```

`4. JSON log দেখতে (SIEM বা Wazuh integration-এর জন্য)`
```
sudo tail -f /var/log/suricata/eve.json
 ```







