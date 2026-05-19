# Tool Pipeline Engine — Autonomous Chaining

> **"The difference between a script kiddie and a Red Team operator is chaining."**

## What is the Tool Pipeline?

The Tool Pipeline is the **nervous system** of Xploit Hub AI. It connects individual tools into automated attack chains where the output of one tool automatically becomes the input of the next — zero human intervention between stages.

---

## How It Works

### Stage-Based Execution

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Stage 1     │───▶│  Stage 2     │───▶│  Stage 3     │───▶│  Stage 4     │
│  Port Scan   │    │  Vuln Scan   │    │  CVE Match   │    │  Exploit     │
│              │    │              │    │              │    │              │
│ Output:      │    │ Input:       │    │ Input:       │    │ Input:       │
│ [22,80,443]  │    │ ports=[22,   │    │ cves=[CVE-   │    │ poc_code +   │
│              │    │  80,443]     │    │  2021-41773] │    │ target IP    │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

Each stage:
1. **Receives** a shared context object containing all findings so far
2. **Executes** its tool with dynamically-built arguments
3. **Parses** its output using regex extractors (ports, CVEs, subdomains, credentials)
4. **Updates** the shared context so the next stage has richer data to work with

### Smart Argument Building (`_build_next_args`)

The pipeline doesn't blindly pass data. It intelligently maps context to the next tool's parameters:
- If open ports were found → injects the first port into exploit tools
- If CVEs were discovered → injects the top CVE into exploit search
- If services were identified → prioritizes high-value targets (SSH > HTTP > Unknown)

---

## Pre-Built Kill Chains

| Pipeline | Stages | Purpose |
|----------|--------|---------|
| `run_recon_pipeline` | IP → Ports → Subdomains → WHOIS → Shodan | Full passive + active reconnaissance |
| `run_exploit_pipeline` | Ports → Vulns → CVE Match → Exploit | Discovery to exploitation |
| `run_web_pipeline` | Google Dork → Vuln Scan → Fuzzing → Exploit Search | Web application attack chain |
| `run_credential_pipeline` | Service Detect → Credential Spray → Save Loot | Authentication attack chain |
| `run_full_kill_chain` | **All 6 phases** (Recon → Enumerate → Vuln → Exploit → Post-Exploit → Exfil) | Maximum autonomous coverage |

---

## Fault Tolerance

A critical design decision: **failed stages do NOT abort the pipeline**. 
- If Shodan returns a 401 (dead API key), the pipeline logs the failure and moves to the next stage.
- If port scan finds nothing, the vulnerability scanner still runs with the original target.
- Every stage result (success or failure) is recorded to `evolution.db` for the AI to learn from.

This mirrors real-world Red Team operations: one failed technique doesn't end the engagement.

---

## Data Flow Example: `run_full_kill_chain("192.168.1.50")`

```
Phase 1: track_ip("192.168.1.50")
  → Context: {target: "192.168.1.50", geo: "Private Range"}

Phase 1: port_scanner("192.168.1.50")
  → Context: {open_ports: [22, 80, 3306], services: {22: "ssh", 80: "http", 3306: "mysql"}}

Phase 1: subdomain_enum("192.168.1.50")
  → Context: {subdomains: []} (private IP, expected)

Phase 2: shodan_scan("192.168.1.50")
  → Context: {shodan: "InternetDB fallback — private IP"}

Phase 3: vulnerability_scanner("192.168.1.50")
  → Context: {cves: ["CVE-2021-41773"], critical_count: 1}

Phase 4: autonomous_exploit_chain("192.168.1.50")
  → Downloads PoC → Patches IPs → Converts Python 2→3 → Fires

Phase 5: credential_spray("192.168.1.50")
  → Tests admin:admin123 across SSH, HTTP, MySQL

Phase 6: save_loot(...)
  → Saves all findings to sessions/
```

---
*Each pipeline is a weapon. Chain them, and you own the kill chain.*
