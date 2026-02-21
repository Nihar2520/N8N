# 🚨 Mini SOAR Platform with n8n

### Automated Threat Enrichment & Simulated Containment

A lightweight **Security Orchestration, Automation, and Response (SOAR)** workflow built using **n8n**, integrating VirusTotal and AbuseIPDB for threat intelligence enrichment and automated IP containment simulation.

---

## 📌 Overview

This project demonstrates how to build an automated SOC-style workflow that:

* Ingests security alerts via Webhook
* Enriches IP reputation using multiple threat intelligence sources
* Performs rule-based risk scoring
* Triggers automated response actions
* Simulates firewall block operations

This implementation serves as a practical proof-of-concept SOAR pipeline suitable for:

* Cybersecurity portfolios
* SOC automation labs
* Blue team demonstrations
* Interview technical showcases

---

## 🏗 Architecture

```
Alert (Webhook)
        ↓
VirusTotal API
        ↓
AbuseIPDB API
        ↓
Merge Intelligence Data
        ↓
Risk Scoring Engine (JavaScript)
        ↓
IF Condition (High Risk)
        ↓
Simulated Firewall Block (HTTP POST)
```

---

## 🛠 Environment Setup

### 1️⃣ Infrastructure

* Ubuntu (VirtualBox NAT)
* Docker
* n8n (Docker container)

### Install Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
```

### Run n8n

```bash
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Access in browser:

```
http://localhost:5678
```

---

## 🔑 API Configuration

### VirusTotal

* Create API key at: [https://www.virustotal.com](https://www.virustotal.com)
* Add in n8n using HTTP Header Authentication:

  ```
  Header Name: x-apikey
  ```

### AbuseIPDB

* Create API key at: [https://www.abuseipdb.com](https://www.abuseipdb.com)
* Add headers:

  ```
  Key: YOUR_API_KEY
  Accept: application/json
  ```

---

## 🚦 Workflow Breakdown

### 🔹 1. Webhook (Alert Ingestion)

POST endpoint:

```
/security-alert
```

Example payload:

```json
{
  "ip": "185.220.101.1",
  "event_type": "failed_login",
  "severity": "medium"
}
```

Simulates SIEM alert ingestion.

---

### 🔹 2. VirusTotal Enrichment

GET request:

```
https://www.virustotal.com/api/v3/ip_addresses/{{ $json["body"]["ip"] }}
```

Extract:

```
data.attributes.last_analysis_stats.malicious
```

---

### 🔹 3. AbuseIPDB Enrichment

GET request:

```
https://api.abuseipdb.com/api/v2/check?ipAddress={{$json["body"]["ip"]}}&maxAgeInDays=90
```

Extract:

```
data.abuseConfidenceScore
data.totalReports
```

---

### 🔹 4. Merge Node

Combine both API responses into a unified schema for processing.

---

### 🔹 5. Risk Scoring Engine (JavaScript Node)

```javascript
let vtMalicious = items[1].json.data.attributes.last_analysis_stats.malicious;
let abuseScore = items[0].json.data.abuseConfidenceScore;
let ip = items[0].json.data.ipAddress;

let risk = "low";

if (vtMalicious >= 5 || abuseScore >= 70) {
  risk = "high";
} else if (vtMalicious >= 1 || abuseScore >= 30) {
  risk = "medium";
}

return [{ ip, vtMalicious, abuseScore, risk }];
```

### Risk Classification Logic

| Condition            | Risk Level |
| -------------------- | ---------- |
| VT ≥ 5 OR Abuse ≥ 70 | High       |
| VT ≥ 1 OR Abuse ≥ 30 | Medium     |
| Otherwise            | Low        |

---

### 🔹 6. Conditional Branching

IF node condition:

```
{{$json["risk"]}} equals high
```

Only high-risk IPs trigger automated response.

---

### 🔹 7. Simulated Firewall Block

POST request:

```
https://httpbin.org/post
```

Body:

```json
{
  "action": "block",
  "ip": "={{$json['ip']}}",
  "risk": "={{$json['risk']}}",
  "timestamp": "={{$now}}"
}
```

This simulates:

* Firewall API call
* WAF rule insertion
* Cloud security group modification

---

## 🧠 Security Concepts Demonstrated

* Threat Intelligence Correlation
* Multi-source Enrichment
* Rule-Based Risk Scoring
* SOC Workflow Automation
* Conditional Orchestration
* Automated Containment Simulation

---

## 🔒 Future Improvements

* Replace simulated block with:

  * UFW via SSH
  * iptables automation
  * AWS Security Group API
  * Cloudflare Firewall API
* Add Slack/Jira alerting
* Store incidents in Elasticsearch
* Integrate with TheHive or MISP
* Implement confidence-weighted scoring model
* Add logging & telemetry layer

---

## 📊 Use Cases

* SOC automation training
* Threat response simulation
* Interview demonstration project
* Blue team lab exercise
* Automation engineering portfolio

---

## ⚠️ Disclaimer

This project is for educational and demonstration purposes only.
The simulated firewall block does not modify real firewall rules unless integrated with actual infrastructure APIs.

---

## 🏁 Conclusion

This project demonstrates how n8n can function as a lightweight SOAR orchestration engine capable of:

* Processing security alerts
* Enriching threat data
* Performing risk analysis
* Triggering automated containment workflows

It provides a modular and extensible foundation for real-world SOC automation systems.

---

## 📝 License

MIT License - Feel free to use this for educational and portfolio purposes.

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

---

## 👤 Author

**[Your Name]**

* GitHub: [@yourusername](https://github.com/Nihar2520)
* LinkedIn: [Your LinkedIn](https://www.linkedin.com/in/nihar-mehta-247339227/)

---

## ⭐ Show Your Support

Give a ⭐️ if this project helped you learn about SOAR automation!
