---
title: "Automating Threat Hunting with IOC-Driven Queries"
description: "Generate platform-specific queries (QRadar, Elastic, Defender) from IPs, domains, or hashes to speed up your threat hunting workflow."
pubDate: "July 1 2025"
heroImage: "/post_img.webp"
badge: "NEW"
tags: ["threat-hunting","python", "automation"]
---

Threat hunting often starts with a handful of indicators—IP addresses, domain names, or file hashes—and a big question: “Where did this come from, and who saw it first?” Whether you're working in QRadar, Elastic, Microsoft Defender, or another platform, writing queries manually for each can be tedious and error-prone.

That’s why I built a simple Python script to automatically generate platform-specific queries from input lists of indicators. The script supports query formats for:

- QRadar (AQL)
- Elasticsearch (KQL)
- Microsoft Defender Advanced Hunting (Kusto)

The main goal? Quickly identify the first point of contact with a potential threat.

Security teams often receive IOCs (Indicators of Compromise) from threat intel feeds, internal alerts, or during incident response. The sooner you can plug those into your SIEM or EDR platform to look for initial connections, the faster you can detect lateral movement, exfiltration, or compromised assets.

In summary, this tool saves time, reduces manual error, and helps you focus on the investigation rather than formatting syntax. Whether you're in a SOC or running your own home lab, it's a handy utility to have in your toolkit.

You can find the full source code on GitHub here:
 https://github.com/olofmagn/iocqueryx


