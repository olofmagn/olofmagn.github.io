---
title: "Mapping the Battlefield: Automated Recon Like an Adversary"
description: "Automate Your Way into the Attack Surface."
pubDate: "Mar 16 2026"
heroImage: "/post_img.webp"
badge: "NEW"
tags: ["recon", "python", "automation"]
badge: "NEW"
---

You've got five terminals open, three half-finished scans, and a downloads folder full of files named `output_final_v2.txt`. Recon without structure is noise.
`scripts` is a small collection of shell and Python tools — subfinder, jsfinder, archivefinder, bypass403, and a few others — built to make passive and active recon repeatable and organized across every target.

In this post I'll walk through each script briefly, what it does, and in which context they are useful.

**archivefinder.sh - Mining the Wayback Machine for Forgotten Files**

The first script in the toolkit is `archivefinder.sh`, and it's often where I normally start. Targets leave traces like old backups, exposed configs/js files, leaked keys, forgotten archives — they get crawled by the Wayback Machine and indexed forever, even if the file was taken down years ago. 

It hits the Wayback Machine CDX API and pulls every possible URL for the target and deduplicates it using `uro` and filters down to sensitive extension like `.env`, `.pem`, `.sql`, `.zip` and more. 

```bash
./archivefinder.sh target.com
```

Not every hit will be a live file - but the value can also be derived from the file pattern to analyse how they operate/technologies in use. 

---

**subfinder.sh - Subdomain Enumeration Across Six Sources**

Any single subdomain tool will miss things - and often takes too long to complete. `subfinder.sh` runs six sources in parallel: `amass`, `subfinder`, `assetfinder`, `github-subdomains`, `shodan` and `crt.sh` and merges everything with the use of `anew` and checks what is alive using `httpx` and screenshot every responding host.

```bash
./subfinder.sh target.com
```

GitHub and Shodan are particually interesting because developers tend to commit internal hostnames and shodan indexes infrastructure that never appears in the app itself. Tokens are loaded from the environment variables or `~/github_token` and `~/.shodan_token`.

---

**bypass403.sh - Breaking Through Access Controls**

A 403 during recon isn't a dead end — it's an invitation. `bypass403.sh` is a comprehensive HTTP 403 bypass tester that throws everything possible at a restricted endpoint: path manipulation, header spoofing, method tampering, encoding tricks, and user-agent rotation. 

```bash
./bypass403.sh -u https://target.com/admin
```

Every hit prints the exact curl command to reproduce it. The header bypass tests are usually effective against endpoints sitting behind nginx or load balancers that forward headers like `X-Original-URL` unchecked. 

---

**jsfinder.sh - Extracting Secrets from JavaScript Files**

JavaScript files are one of the most overlooked attack surfaces in web testing. It is very common that developers bundle API keys, internal endpoint paths, auth tokens, and other credentials directly into client-side JS. `jsfinder.sh` is built to find those files and analyse them for anything sensitive.

```bash
./jsfinder.sh target.com
```

This is actually how I found my first high severity finding - a hardcoded API key in a bundled JS file that granted access to an internal API.


---
**redirect.sh - Testing Open Redirect Vulnerabilities**

Open redirects are easy to miss and easy to underestimate - especially when they can get chained with other vectors like phishing. `redirect.sh` gives you a structured way to test a suspected redirect parameter with 13 different bypass techniques in one run.

```bash
./redirect.sh -u 'https://target.com/login?next={{INJECT}}' -d evil.com -s https
```

It covers protocol-relative URLs, backslash confusions, symbol bypass, encoded variations through the most common parameter names that some pure header-based tools miss.

