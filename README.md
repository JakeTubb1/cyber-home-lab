# Cybersecurity Home Lab

## Overview
This repository documents my personal cybersecurity home lab, created to develop practical skills in penetration testing, security monitoring and threat detection.

Using VMware, I have built a virtual network to simulate cyberattacks and investigate how security tools detect suspicious activity.

## Technologies Used
- **Kali Linux** – Penetration testing
- **Wazuh** – SIEM and security monitoring
- **Security Onion** – Network intrusion detection
- **Windows Server 2022** – Target machine
- **Sysmon** – Windows event logging
- **VMware** – Virtualisation
- **Python & Ollama** – AI-assisted SOC development

## Practical Experience
- Network scanning and reconnaissance using Nmap
- Password-guessing simulations using CrackMapExec
- Analysing Wazuh and Security Onion alerts
- Configuring Sysmon for Windows event monitoring
- Investigating security detection capabilities
- Developing an AI-assisted SOC alert triage system

## Ongoing Development – AI-Assisted SOC (University Dissertation)

As part of my final-year Cyber Security dissertation at the University of the West of England (UWE), I am expanding this home lab to investigate the effectiveness of AI in Security Operations Centre (SOC) alert triage.

The project aims to develop a locally hosted AI assistant using Python, Ollama and Wazuh to analyse security alerts and support threat classification.

### Project Objectives
- Integrate Wazuh and Sysmon logs with a locally hosted Large Language Model (LLM).
- Simulate controlled cyberattack scenarios aligned with MITRE ATT&CK.
- Develop an AI pipeline to classify security alerts and identify relevant attack techniques.
- Compare AI-generated classifications against a rule-based baseline and predefined ground-truth labels.
- Evaluate classification accuracy, false positives, false negatives and processing time.
- Investigate the strengths and limitations of AI-assisted SOC analysis.

**Status:** Ongoing – Final-year university dissertation.

## Documentation
Full documentation, including screenshots, commands and findings:

[View Home Lab Documentation](./DocForLab.md)

---
*All security testing was conducted within my own controlled home lab for educational purposes.*
