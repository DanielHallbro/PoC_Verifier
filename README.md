# PoC-Verifier 
> A lightweight Python framework for verifying vulnerability reports and analyzing target technology context.

### Status: Current focus is on **Architectural Design** and **Core Module Prototyping**. 

## Overview
**PoC-Verifier** is a project I’m planning on building to handle the initial verification when a new vulnerability report arrives. By automatically matching the reported bug against the actual tech stack and defensive layers, I can quickly filter out what’s impossible and focus my time on the high-impact findings that actually matter.

---

## Core Features
* **Stack Fingerprinting:** Identifies CMS, web servers, and backend frameworks to ensure the reported vulnerability is technologically feasible on the target.
* **WAF Detection:** Analyzes HTTP response signatures and status code behavior to identify defensive layers (Cloudflare, Akamai, etc.) that may impact PoC execution.
* **Non-Destructive Probing:** Generates safe "canary" payloads (for example, unique reflection strings for XSS) to verify vulnerability presence without disrupting production environments. Aim to cover as many **OWASP Top10** vulnerabilites as is feasable for a automated tool on v1.0.0 launch.
* **Confidence Scoring:** Assigns a probability score to each report based on environment match and WAF presence. It's a concept I like but I can see the dangers that flawed logic here can create.
* **Re-usability:** Aim to utilize a standardized logging module (adapted from [IOC Analyzer](https://github.com/DanielHallbro/IOC_Analyzer)) to ensure auditable usage. Concider utilizing more previously written modules.

---

## Input & Workflow
To support both manual research and automated batch processing, I am aiming for **PoC-Verifier** to support multiple input streams:
* **Interactive CLI:** For rapid, single-target verification.
* **Batch Processing:** Support for importing structured reports via JSON/CSV. Could futher be developed by creating standalone translation scripts to convert raw company-specific report forms into a format the verifier can process automatically.

---

## Technical Preview: WAF Detection Logic
*Snippet demonstrating the signature-based analysis used to identify security headers and fingerprints. First working version will include more signatures*

```python
import requests

def check_waf(url):
    signatures = {
        "Cloudflare": ["cf-ray", "__cfduid", "cf-cache-status"],
        "Akamai": ["x-akamai-transformed", "akamai-origin-hop"],
        "AWS WAF": ["x-amzn-requestid", "awselb"],
        "Imperva": ["x-iinfo", "incap_ses"]
    }
    
    try:
        response = requests.get(url, timeout=5)
        headers = {k.lower(): v for k, v in response.headers.items()}
        
        for waf, keys in signatures.items():
            if any(key in headers for key in keys):
                return f"WAF Detected: {waf}"
                
        return "No obvious WAF fingerprints found."
    except Exception as e:
        return f"Connection Error: {e}"
```

---

## Roadmap (v0.1.0-alpha)

[ ] Input Translation Layer: Develop "adapter" logic to convert various report formats (JSON/CSV) into standardized internal objects.

[ ] Stack-to-Vuln Matcher: Build the logic engine that compares discovered tech (e.g., Nginx) against reported bugs (e.g., PHP-specific CVEs).

[ ] Verification Library: Develop a library of "Canary" payloads for non-destructive testing (XSS reflection and Time-based SQLi).

[ ] CLI Dashboard: Simple clean CLI summary output for rapid decision-making results.