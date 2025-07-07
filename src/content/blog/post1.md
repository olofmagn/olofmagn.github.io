---
title: "IDS rulesets"
description: "Understanding the wide number of rules in categories in IDS systems"
pubDate: "Mars 10 2025"
heroImage: "/design-ids.png"
tags: ["ids", "suricata", "stamus"]
---

Understanding the vast landscape of IDS rules—and deploying them effectively within a CI/CD pipeline—can be quite challenging. To tackle this, I’ve been working on a home automation setup that streamlines the process. It handles everything from fetching datasets and gathering rules for potential customers, to building Ansible-compatible rulesets and efficiently distributing new or updated rules across all client systems.

Although this approach does require ongoing maintenance and some domain expertise, it opens up the possibility of tailoring IDS configurations to match the specific needs of each customer or environment—without spending any extra money. 

The main issue I’ve encountered during this process is that the rules aren’t always properly validated, which can lead to broken rulesets in the pipeline. In many cases, the build process fails due to specific rules that either require a different Suricata version or depend on other deployment-specific factors. To address this, some custom logic is needed to proper validate that the rules fetched from open-source threat sources are compatible and won't break when dispatched.


While this approach is certainly feasible, the major drawback is that it’s time-consuming and requires continuous development effort. It might be worth exploring alternative solutions that already have these factors built in. That said, I’m sharing the design and process I’ve developed so far in the hope that it helps others facing similar challenges when developing custom IDS solutions.


One platform I'm currently exploring in more detail is Stamus NDR. I'm evaluating their community version to better understand its CI/CD capabilities and overall value as a Network Detection and Response (NDR) solution.

More on this in upcoming posts.
