# API Integrations — External Intelligence Sources

> **"An AI without data is just a chatbot. An AI with live intel is a weapon."**

Xploit Hub AI relies on multiple external APIs to gather real-time threat intelligence. This document explains what each API does, why we chose it, and how we handle failures gracefully.

---

## What is an API?

**API = Application Programming Interface**

Think of it as a **messenger between two systems**. Your program sends a structured request (like a question), the remote server processes it, and sends back structured data (the answer). 

In Xploit Hub, APIs are used to pull real-world intelligence that our local tools cannot generate on their own.

---

## API Map

```
                         ┌─────────────────────┐
                         │   XPLOIT HUB AI     │
                         │   (Orchestrator)    │
                         └─────────┬───────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
    │  LLM API        │  │  Intel APIs     │  │  Exploit APIs   │
    │  (The Brain)    │  │  (The Eyes)     │  │  (The Weapons)  │
    │                 │  │                 │  │                 │
    │  • DeepSeek     │  │  • Shodan       │  │  • NVD          │
    │  • OpenAI       │  │  • InternetDB   │  │  • GitHub       │
    │                 │  │  • ip-api.com   │  │    Advisory DB  │
    │                 │  │  • crt.sh       │  │  • ExploitDB    │
    └─────────────────┘  └─────────────────┘  └─────────────────┘
```

---

## 1. LLM APIs (The Brain)

### DeepSeek API / OpenAI API
- **Purpose:** Powers the AI decision-making engine. The LLM reads tool outputs, reasons about the next step, and decides which tool to call.
- **How it works:** We send a system prompt (persona) + user query + tool results as a conversation. The LLM responds with either a natural language answer or a `tool_call` JSON to invoke the next tool.
- **Cost Model:** Charged per **token** (roughly 1 token ≈ 0.75 words). A typical scan conversation costs ~5,000-10,000 tokens.
- **Fallback:** If DeepSeek is down, we rotate to OpenAI. If both fail, the system runs tools in a pre-defined pipeline order without AI reasoning.

### Token Economics
| Operation | Approx. Tokens | Approx. Cost |
|-----------|----------------|--------------|
| Simple query ("What is XSS?") | ~500 | $0.001 |
| Full recon + analysis | ~8,000 | $0.008 |
| Kill chain with exploit reasoning | ~15,000 | $0.015 |

---

## 2. Intelligence APIs (The Eyes)

### Shodan API
- **Purpose:** Search engine for internet-connected devices. Returns open ports, running services, SSL certificates, and known vulnerabilities for any IP.
- **Endpoint:** `https://api.shodan.io/shodan/host/{ip}`
- **Requires:** API Key (free tier: 100 queries/month)
- **Fallback:** When API key is expired/invalid (HTTP 401), we automatically switch to **InternetDB** (`https://internetdb.shodan.io/{ip}`) which is free, keyless, and returns ports + CVEs.

### ip-api.com
- **Purpose:** IP geolocation. Returns country, city, ISP, and organization for any IP address.
- **Endpoint:** `http://ip-api.com/json/{ip}`
- **Auth:** None required (rate limited to 45 requests/minute)

### crt.sh (Certificate Transparency)
- **Purpose:** Subdomain enumeration. Searches SSL certificate logs to discover all subdomains issued for a domain.
- **Endpoint:** `https://crt.sh/?q=%.{domain}&output=json`
- **Auth:** None required

---

## 3. Exploit Intelligence APIs (The Weapons)

### NVD — National Vulnerability Database
- **Purpose:** Search for known CVEs (Common Vulnerabilities and Exposures) by software name and version.
- **Endpoint:** `https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch={query}`
- **Our Enhancement:** We filter results to CVEs published after 2022 with CVSS score ≥ 6.0 to avoid irrelevant legacy vulnerabilities.
- **Fallback:** When NVD returns < 3 results, we query the **GitHub Advisory Database** as a secondary source.

### GitHub Advisory Database
- **Purpose:** Community-maintained vulnerability database with faster updates than NVD.
- **Endpoint:** `https://api.github.com/advisories?keyword={query}`
- **Auth:** None required for public advisories

### ExploitDB (via Google Dorking)
- **Purpose:** Download actual Proof-of-Concept exploit code for a given CVE.
- **Method:** We search `site:exploit-db.com {CVE-ID}` and parse the results for downloadable PoC scripts.

---

## Resilience: What Happens When APIs Fail?

| API | Failure Mode | Fallback Strategy |
|-----|-------------|-------------------|
| DeepSeek/OpenAI | Rate limit / Down | Rotate to alternate provider → Fall back to pipeline mode |
| Shodan | 401 Unauthorized | Auto-switch to InternetDB (free, no auth) |
| NVD | Empty results | Query GitHub Advisory DB → Use `_known_service_exploits` table |
| ip-api.com | Rate limited | Cache results, retry after 60s |
| crt.sh | Timeout | Return empty subdomain list, continue pipeline |

**Design Principle:** No single API failure should stop the entire operation. Every external dependency has a fallback path.

---
*APIs are the supply lines. Cut one, and we reroute. Cut all, and we still have the local arsenal.*
