# Cloud-Native SOC & Global Honeypot  
## "Transforming Raw Logs into Real-Time Threat Intelligence"

---

# 🎯 The Vision

Modern cybersecurity is about visibility. This project demonstrates the deployment of a full-stack Security Operations Center (SOC) in Microsoft Azure. By intentionally exposing a vulnerable Windows 10 Virtual Machine (the "Honeypot") to the open internet, I captured, analyzed, and visualized live cyber attacks from global threat actors.

<img width="719" height="405" alt="image" src="https://github.com/user-attachments/assets/239811df-188e-49be-9fa7-06ca260b1a78" />

---

# 🏗️ Phase 1: The Sacrificial Lamb (Virtual Machine)

I deployed a Windows 10 VM in East US 2, configured as a "Honeypot."

<img width="681" height="566" alt="image" src="https://github.com/user-attachments/assets/78d586fa-3539-4f65-9af9-9dd0b8b95121" />

## Network Security Group (NSG) Overhaul
I configured a "Wildcard Rule" (`*`) to allow all traffic on all ports.

<img width="484" height="664" alt="image" src="https://github.com/user-attachments/assets/2316a2ee-2786-4fe8-b704-6ad53a84a4cf" />

## Internal Firewall Disabling
Inside the VM, I disabled the Windows Defender Firewall.

<img width="625" height="399" alt="image" src="https://github.com/user-attachments/assets/913e0adb-4915-4456-82e8-94d0c5aa78e5" />

## Local Log Verification (Pre-Cloud Validation)

Before moving to the cloud, I verified that the VM was properly recording attack attempts by simulating a failed login using fake credentials (e.g., username: `Good-Guy`). I then opened Windows Event Viewer and filtered for **Event ID 4625** (*An account failed to log on*), confirming that every unauthorized "knock on the door" was being captured locally.

<img width="771" height="463" alt="image" src="https://github.com/user-attachments/assets/25f6f52f-c6c5-4721-aec3-b0265e2ecef3" />


## The "Why"
In a standard environment, an NSG is your first line of defense (Layer 4). By removing these barriers, we ensure that every automated bot on the internet can "knock on the door," providing us with a massive dataset of malicious traffic.

---

# 🕵️ Phase 2: The Centralized Brain (LAW & Sentinel)

Logs are useless if they stay on the machine. I needed a way to aggregate and analyze them at scale.

## Log Analytics Workspace (LAW)
Think of this as the "Database." It’s where raw data from the VM is stored and indexed.

<img width="626" height="163" alt="image" src="https://github.com/user-attachments/assets/771f01fc-2029-4d61-b026-5694b38f5300" />

## Microsoft Sentinel (SIEM)
While LAW holds the data, Sentinel is the "Security Intelligence" layer. It sits on top of the LAW to provide alerting, incident management, and advanced hunting capabilities.

<img width="907" height="410" alt="image" src="https://github.com/user-attachments/assets/8ca86d01-0098-4609-944f-b2c4388f4913" />

## The "Why"
We use a SIEM (Security Information and Event Management) because searching through individual text files on a server is impossible in a real-world SOC. Sentinel allows us to query millions of logs in seconds using KQL.

---

# 📡 Phase 3: The Data Pipeline (AMA & DCR)

To get the logs from the VM into our SIEM, I implemented the modern Azure Monitor Agent (AMA).

<img width="900" height="204" alt="image" src="https://github.com/user-attachments/assets/123e8a85-06a3-4681-b21b-4cbf137e772b" />

## Data Collection Rule (DCR)
I created a DCR to tell the agent exactly what to collect. In this case, I targeted "Security Events" (like failed logins).

<img width="882" height="529" alt="image" src="https://github.com/user-attachments/assets/7403843a-1ce6-4251-9802-00d415c69f98" />

## First Cloud Validation (Moment of Truth)

Once the pipeline was live, I ran my first **KQL (Kusto Query Language)** query in the logs. This was the moment of truth to see if the cloud was finally "hearing" the VM.

<img width="781" height="310" alt="image" src="https://github.com/user-attachments/assets/cd597991-c2a0-4e04-a61f-afa539271f32" />


## The "Why"
Older methods (like the Log Analytics Agent) are being phased out. Using AMA and DCRs is the "Modern Way" to handle data ingestion, offering better security and granular control over what data gets sent to the cloud.

---

# 🌍 Phase 4: Threat Enrichment (KQL & Watchlists)

A raw IP address like `185.x.x.x` doesn't tell a story. I wanted to know where the attacker was sitting.

## Watchlists
I imported a GeoIP CSV containing 54,000+ rows of IP-to-Location data.

## KQL (Kusto Query Language)
I wrote a custom query to perform an `ipv4_lookup`. This joined my "Failed Login" logs with my "GeoIP Watchlist" in real-time.

<img width="934" height="395" alt="image" src="https://github.com/user-attachments/assets/511ed4e7-b207-49ef-92dd-e278683794bd" />


## The "Why"
This is called Threat Enrichment. By adding City, Country, and Coordinates to a log entry, a SOC analyst can immediately see patterns (e.g., "Why are we getting 5,000 hits from a single city in another country?").

---

# 📊 Phase 5: The War Room (Sentinel Workbooks)

The final step was building a visual dashboard.

## Workbooks
I used JSON-based templates to create a dynamic Attack Map.

## Real-World Observation
After deploying the workbook, I had to wait approximately **5 hours** to see a significant volume of global attack data populate the map. This reinforced an important lesson: threat intelligence visualization improves over time as more telemetry is ingested.

## The "Why"
High-level stakeholders and SOC Managers need visual summaries. A map that shows "Attack Intensity" via bubble size allows a team to prioritize which geographic regions or IP ranges need to be blocked first.

<img width="818" height="437" alt="image" src="https://github.com/user-attachments/assets/28558138-880a-4abb-97cb-e263036dc912" />

` Final Attack Map with the 3D globe and colored dots`

---

# 🧠 Final Reflection: Lessons Learned

- Proper exposure and logging are critical: NSGs, firewalls, and Event Viewer confirmed that every attack attempt was captured.  
- Cloud ingestion pipelines (AMA + DCR + LAW) must be carefully configured to ensure secure, structured data flow.  
- SIEM and KQL transform raw logs into actionable intelligence; enrichment with watchlists makes threat patterns visible.  
- Visualizations in Sentinel Workbooks help translate technical data into decision-ready insights for SOC operations.  

