# 📐 Xploit Hub AI — Architecture Documentation

> **A complete technical breakdown of how Xploit Hub AI thinks, attacks, and learns.**

This directory contains the full architectural documentation for Xploit Hub AI — an Autonomous Agentic Cybersecurity Framework built to automate the entire Red Team kill chain.

---

## 📁 Document Index

| Document | What It Covers |
|----------|---------------|
| [**CORE_ARCHITECTURE.md**](./CORE_ARCHITECTURE.md) | System overview, Brain-Tool-Executor paradigm, agentic vs scripted philosophy |
| [**TOOL_PIPELINE.md**](./TOOL_PIPELINE.md) | Autonomous tool chaining, stage-based execution, pre-built kill chains, fault tolerance |
| [**API_INTEGRATIONS.md**](./API_INTEGRATIONS.md) | Every external API used (Shodan, NVD, DeepSeek, crt.sh), cost model, fallback strategies |
| [**KNOWLEDGE_BASE.md**](./KNOWLEDGE_BASE.md) | Attack pattern library, service playbooks, self-learning evolution engine, MITRE ATT&CK mapping |
| [**FUTURE_ROADMAP.md**](./FUTURE_ROADMAP.md) | Vision for v3.0+: custom model fine-tuning, real-time C2, multi-agent swarm |
| [**COMMUNITY.md**](./COMMUNITY.md) | Live Discord server, bot commands, community info |

---

## 🏗️ High-Level System Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                        DISCORD INTERFACE                         │
│                    (User sends target/command)                   │
└──────────────────────────┬───────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                      XPLOIT AI ENGINE                            │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐  |
│  │  LLM Brain   │◄─▶│  Tool Router │◄─▶│  Tool Pipeline      │  |
│  │  (DeepSeek/  │   │  (Decides    │   │  (Chains tools       │  |
│  │   OpenAI)    │   │   which tool │   │   automatically)     │  |
│  │              │   │   to call)   │   │                      │  |
│  └──────────────┘   └──────────────┘   └──────────────────────┘  |
└──────────────────────────┬───────────────────────────────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
     ┌──────────────┐ ┌──────────┐ ┌──────────────┐
     │ Recon Tools  │ │ Exploit  │ │ Post-Exploit │
     │              │ │ Tools    │ │ Tools        │
     │ • track_ip   │ │ • fuzz   │ │ • cred_spray │
     │ • port_scan  │ │ • chain  │ │ • save_loot  │
     │ • subdomain  │ │ • poc    │ │ • KB learn   │
     │ • shodan     │ │ • search │ │              │
     └──────┬───────┘ └────┬─────┘ └──────┬───────┘
            │              │              │
            └──────────────┼──────────────┘
                           ▼
              ┌──────────────────────┐
              │    External APIs     │
              │  Shodan • NVD • crt  │
              │  ip-api • GitHub     │
              └──────────────────────┘
```

---

## 🎯 Key Engineering Decisions

| Decision | Why |
|----------|-----|
| **LLM as orchestrator, not executor** | LLMs hallucinate. We use them for reasoning only; actual network operations run as deterministic Python code. |
| **Pipeline continues on failure** | Real Red Team ops don't stop at first blocked port. Neither do we. |
| **Python 2→3 auto-converter** | Most public PoCs are legacy Python 2. We intercept and fix syntax before execution. |
| **Private IP detection** | NVD queries for `192.168.x.x` return garbage. We detect RFC-1918 ranges and switch to a local exploit knowledge table. |
| **InternetDB fallback** | Shodan API keys expire. InternetDB provides free port/CVE data as a resilient backup. |
| **Evolution DB** | Every success/failure is logged. The system's accuracy improves over time through recorded outcomes. |

---

*Built with the philosophy: "Automate the boring. Accelerate the lethal. Remember everything."*

---

## 🌐 Community & Live Demo

This architecture is not theoretical — the bot is **live and running** inside the Xploit Hub Discord server.

<div align="center">

### **[➜ Join Xploit Hub — discord.gg/4DTWQZG6rF](https://discord.gg/4DTWQZG6rF)**

</div>

Interact with the AI directly using only one slash commands `/ask prompt:`.
Full command list and details: **[COMMUNITY.md](./COMMUNITY.md)**

> **⚠️ Bot Offline Notice**
> The bot is hosted on a personal instance and may occasionally appear offline due to maintenance or server restarts.
> If the bot is offline:
> - Wait up to **24 hours** — it will restart automatically in most cases.
> - Or **DM the developer** directly via Discord for a faster response.
> There is no cause for concern — all data and configurations are persistent and will resume normally on restart.

---

*"We don't just automate attacks. We build the system that automates the attacker."*
