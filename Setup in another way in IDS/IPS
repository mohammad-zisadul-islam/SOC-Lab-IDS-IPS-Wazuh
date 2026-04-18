Full IDS Documentation: Suricata + Wazuh (Step by Step)
What you will build

Suricata VM (IDS sensor): detects attacks, writes alerts to eve.json

Wazuh Agent (on Suricata VM): reads eve.json and sends to Wazuh Manager

Wazuh Server (Manager + Indexer + Dashboard): stores + displays alerts

Part A — Install and configure Suricata (on Suricata VM)
1) Install Suricata
sudo apt update
sudo apt install suricata -y


Check version:

suricata -V

2) Find your network interface (IMPORTANT)

Suricata must capture on the correct interface (not always eth0).

ip -br a


Example:

lo        UNKNOWN
ens33     UP     192.168.79.135/24


✅ Your interface is ens33 (the one with your IP).

3) Edit Suricata config
sudo nano /etc/suricata/suricata.yaml

3.1 Set the interface (fix “eth0 no such device”)

Find af-packet: and set your interface:

af-packet:
  - interface: ens33


(Replace ens33 with YOUR interface name.)

3.2 Enable eve.json output (Suricata JSON logs)

Find outputs: and ensure this exists:

outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: /var/log/suricata/eve.json
      types:
        - alert
        - dns
        - http
        - tls
        - flow

3.3 Ensure rule paths exist

Make sure you have something like:

default-rule-path: /var/lib/suricata/rules

rule-files:
  - suricata.rules
  - local.rules


Save and exit:

Ctrl+O → Enter

Ctrl+X

4) Create an ICMP detection rule (Ping alert rule)

Create local rules file:

sudo nano /var/lib/suricata/rules/local.rules


Paste exactly one line:

alert icmp any any -> any any (msg:"ICMP Ping Detected"; itype:8; sid:1000001; rev:1;)


Save and exit.

5) Test Suricata config
sudo suricata -T -c /etc/suricata/suricata.yaml


✅ Expected:

Configuration provided was successfully loaded


If you see rule errors, your local.rules syntax is wrong.

6) Start Suricata

If it failed earlier, reset first:

sudo systemctl reset-failed suricata


Restart:

sudo systemctl restart suricata
sudo systemctl status suricata --no-pager


✅ Must show: Active: active (running)

7) Confirm Suricata is writing logs
sudo ls -lah /var/log/suricata/
sudo tail -n 20 /var/log/suricata/eve.json


✅ You should see JSON lines.

Part B — Generate traffic to trigger IDS alerts
8) Test ICMP (ping) detection (IMPORTANT)

✅ Best method:

From another VM (Attacker/Test VM):
ping -c 3 <SURICATA_VM_IP>


Example:

ping -c 3 192.168.79.135

9) Confirm Suricata alert is generated

On Suricata VM:

sudo tail -n 50 /var/log/suricata/eve.json | grep -i icmp -n


Look for "ICMP Ping Detected".

Part C — Configure Wazuh Agent to send Suricata alerts
10) Add Suricata log to Wazuh Agent

Edit Wazuh agent config:

sudo nano /var/ossec/etc/ossec.conf


Add inside <ossec_config>:

<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>


Save and exit.

Restart agent:

sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent --no-pager

Part D — Verify Wazuh server is receiving Suricata events
11) Check Wazuh Manager archives (on Wazuh Server)

Run on Wazuh Manager:

sudo tail -n 200 /var/ossec/logs/archives/archives.json | grep -i suricata | tail


If you see output → ✅ Wazuh is receiving Suricata logs.

Part E — View ICMP alerts in Wazuh Dashboard
12) Wazuh Dashboard filters

Go to:
✅ Threat Hunting → Events (or Discover)

Search one of these:

Option A (signature text)
ICMP Ping Detected

Option B (protocol)
data.proto:ICMP

Option C (suricata group)
rule.groups:suricata


You should now see ICMP alerts.

Troubleshooting (Common Issues)
Suricata service fails

Check:

sudo journalctl -u suricata -n 50 --no-pager


Most common fix:

interface name wrong (eth0 doesn’t exist)

eve.json missing

Fix:

ensure filename: /var/log/suricata/eve.json

restart Suricata

No alerts even after ping

Make sure:

Suricata is running

local.rules is included in suricata.yaml

you are pinging Suricata VM IP from another VM


📘 Suricata IPS (NFQUEUE) + Wazuh Integration Documentation

Goal:
Enable Suricata IPS to block ICMP traffic and display blocked events in the Wazuh dashboard.

🧱 Architecture

Suricata VM

Runs Suricata 8.x in IPS (inline) mode

Blocks traffic using NFQUEUE

Writes alerts to eve.json

Wazuh Agent (on Suricata VM)

Reads Suricata eve.json

Sends events to Wazuh Manager

Wazuh Manager

Decodes Suricata events

Applies custom rules

Displays alerts in dashboard

1️⃣ Prerequisites

Linux kernel supports NFQUEUE

Suricata ≥ 8.x installed

Wazuh agent installed on Suricata VM

Wazuh manager reachable

2️⃣ Enable NFQUEUE kernel modules
sudo modprobe nfnetlink
sudo modprobe nfnetlink_queue
sudo modprobe nf_conntrack


Verify:

lsmod | grep nfqueue

3️⃣ Configure iptables (send ICMP to NFQUEUE)
sudo iptables -I INPUT  -p icmp -j NFQUEUE --queue-num 0
sudo iptables -I OUTPUT -p icmp -j NFQUEUE --queue-num 0


Verify:

sudo iptables -S | grep NFQUEUE

4️⃣ Suricata configuration (IMPORTANT)
❌ Do NOT use:

--nfqueue (not supported in Suricata 8)

AF-PACKET copy-mode: ips (unless you have 2 NICs)

✅ Use NFQUEUE via -q
5️⃣ Configure systemd for Suricata IPS

Edit override:

sudo systemctl edit suricata


Add only this:

[Service]
ExecStart=
ExecStart=/usr/bin/suricata -c /etc/suricata/suricata.yaml -q 0 --pidfile /run/suricata.pid --user suricata --group suricata


Reload and restart:

sudo systemctl daemon-reload
sudo systemctl reset-failed suricata
sudo systemctl restart suricata


Check:

sudo systemctl status suricata
ps aux | grep suricata | grep -v grep


✅ You must see -q 0

6️⃣ Create ICMP IPS rule (BLOCK)

Edit:

sudo nano /var/lib/suricata/rules/local.rules


Add:

drop icmp any any -> any any (msg:"IPS ICMP blocked"; itype:8; sid:1000001; rev:1;)


Test config:

sudo suricata -T -c /etc/suricata/suricata.yaml


Expected:

Configuration provided was successfully loaded


Restart Suricata:

sudo systemctl restart suricata

7️⃣ Verify traffic is blocked

From another VM:

ping <SURICATA_VM_IP>


Expected:

❌ Ping fails / times out

8️⃣ Verify Suricata logs the block
sudo grep -i "IPS ICMP blocked" /var/log/suricata/eve.json


You should see:

"event_type":"alert"
"action":"blocked"
"signature":"IPS ICMP blocked"

9️⃣ Configure Wazuh agent (Suricata logs)

Edit:

sudo nano /var/ossec/etc/ossec.conf


Add:

<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>


Restart agent:

sudo systemctl restart wazuh-agent


Verify:

sudo grep eve.json /var/ossec/logs/ossec.log


Expected:

Analyzing file: '/var/log/suricata/eve.json'

🔟 Wazuh Manager custom rule (IPS alert)

Edit on Wazuh Manager:

sudo nano /var/ossec/etc/rules/local_rules.xml


Add:

<group name="suricata,ips,">
  <rule id="100500" level="10">
    <if_sid>suricata</if_sid>
    <field name="data.alert.action">blocked</field>
    <description>Suricata IPS blocked traffic</description>
  </rule>
</group>


Restart manager:

sudo systemctl restart wazuh-manager

🔍 11️⃣ Verify alert on Wazuh Manager
sudo grep -i "Suricata IPS blocked" /var/ossec/logs/alerts/alerts.json

📊 12️⃣ Wazuh Dashboard

In Security Events / Discover, filter:

rule.description:"Suricata IPS blocked traffic"


You should see ICMP block events 🎉

⚠️ Common Mistakes (Important)

❌ Using --nfqueue
❌ AF-PACKET IPS with one interface
❌ Missing sid in Suricata rule
❌ Expecting alerts when Suricata is not running
❌ Checking alerts on agent instead of manager

✅ Final Result

ICMP packets blocked by Suricata IPS

Block events logged in eve.json

Wazuh agent collects them

Wazuh dashboard displays IPS alerts
