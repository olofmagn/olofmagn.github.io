---
title: "Automating IDS Rule Deployments"
description: "Lessons from automating IDS rule deployment and evaluating Stamus NDR."
pubDate: "Mars 10 2025"
heroImage: "/design-ids.png"
tags: ["ids", "suricata", "stamus"]
---

Understanding the vast landscape of IDS rules—and deploying them effectively within a CI/CD pipeline — can be quite challenging. To tackle this, I’ve been working on a home automation setup that attempts to automate this kind of process. It handles everything from fetching datasets and rules for potential environments, to building Ansible-compatible rulesets and efficiently distributing new or updated rules across all systems.

Though this approach works effectively, it comes with significant drawbacks: it's resource-intensive and demands ongoing development maintenance. However, if you are working in a SOC or similiar environment with a dedicated person for IDS development, this approach could be feasible. The design and methodology is documented above and in more detail in https://github.com/olofmagn/ids_service_identifier.

But before commiting to a custom solution, it is always worth evaluating existing alternatives that include these capabilities out of the box, especially when time or resources are limited for custom developments.

One platform I'm currently exploring in more detail is Stamus NDR. I'm evaluating their community version to better understand its CI/CD capabilities and overall value as a Network Detection and Response (NDR) solution.

More on this in upcoming posts.
