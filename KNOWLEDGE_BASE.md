# Knowledge Base & Self-Learning Engine

> **"An operator who forgets past engagements is doomed to repeat failed attacks."**

Xploit Hub AI doesn't just execute — it **remembers and learns**. This document explains the Knowledge Base (KB) system and the Evolution Engine that allows the AI to improve with every operation.

---

## 1. The Knowledge Base (`knowledge_base.py`)

The KB is Xploit Hub's internal library of attack patterns, playbooks, and credentials. It serves two purposes:
1. **Quick Reference:** The AI can query it mid-operation to recall proven techniques.
2. **Context Injection:** Before attacking a service, the AI pulls the relevant playbook from KB to guide its strategy.

### KB Categories

```
BUILTIN_KNOWLEDGE
├── web_attack_patterns[]       ← SQLi, SSRF, JWT Bypass, Deserialization
│   ├── name
│   ├── payloads[]              ← Ready-to-fire attack strings
│   ├── code_snippet            ← Working Python exploit code
│   ├── mitre_id                ← MITRE ATT&CK mapping (T1190, T1557, etc.)
│   └── keywords[]              ← Search triggers
│
├── service_playbooks{}         ← SSH, HTTP, SMB, RDP attack strategies
│   ├── ssh: [Banner grab, Key check, Brute force, CVE-2018-10933]
│   ├── http: [Vhost enum, Dir bust, Tech fingerprint, API discovery]
│   ├── smb: [Enum shares, EternalBlue, Null session, IPC$]
│   └── rdp: [BlueKeep, NLA check, Dejavu analysis]
│
├── post_exploit_checklist[]    ← .bash_history, SUID bins, /etc/shadow, pivot scan
│
└── default_credentials[]       ← root:root, admin:admin123, postgres:postgres
```

### Smart Search (`query_kb`)

The search engine doesn't just do basic string matching. It uses **multi-layer keyword resolution**:

1. **Direct keyword match:** Query "sql injection" → matches `keywords: ["sqli", "sql", "injection"]`
2. **Reverse word match:** Query "how to attack ssh" → word "ssh" matches service playbook key
3. **Cross-category search:** A single query searches attack patterns, playbooks, credentials, AND post-exploit checklists simultaneously
4. **Synthetic entries:** Service playbooks and credential databases are returned as structured KB entries (with MITRE IDs) so the AI can process them uniformly

**Example:**
```
Input:  query_kb("ssh attack")
Output: [
  {name: "SSH Attack Playbook", payloads: ["Banner grabbing", "Key-based auth check", ...], mitre_id: "T1190"},
  {name: "Default Credentials Database", payloads: ["ssh: root / root", ...], mitre_id: "T1078"}
]
```

---

## 2. The Evolution Engine (`evolution.db`)

Every tool execution — success or failure — is recorded in a SQLite database called `evolution.db`. This is the AI's **long-term memory**.

### What Gets Recorded

| Field | Description |
|-------|-------------|
| `technique` | Tool name + context (e.g., `exploit_chain_CVE-2021-41773`) |
| `target_type` | Category of target (e.g., `auto_learn`, `web`, `internal`) |
| `service` | Detected service (e.g., `Apache`, `SSH`) |
| `port` | Target port |
| `success` | Boolean: did the technique work? |
| `notes` | First 500 chars of tool output |
| `ts` | Timestamp |

### How Learning Works

```
Operation Executed
       │
       ▼
  ┌──────────────────┐
  │ learn_from_result │ ← Called after every tool execution
  │   (topic,         │
  │    finding,       │
  │    success)       │
  └────────┬─────────┘
           │
           ▼
  ┌──────────────────┐
  │  evolution.db     │ ← SQLite: technique_results table
  │                   │
  │  INSERT INTO      │
  │  technique_results│
  │  (technique,      │
  │   target_type,    │
  │   success,        │
  │   notes, ts)      │
  └──────────────────┘
```

### Why This Matters

In a real-world Red Team engagement:
- If `credential_spray` with `admin:admin` failed 10 times on SSH, the system knows to deprioritize that combination.
- If `CVE-2021-41773` succeeded 3 times on Apache 2.4.49, it becomes a **high-confidence** exploit for future Apache targets.
- The pipeline records every stage (success or failure) so post-engagement analysis can identify which attack paths are most effective.

---

## 3. MITRE ATT&CK Integration

Every KB entry is tagged with a MITRE ATT&CK Technique ID. This allows:
- **Professional reporting:** "We leveraged T1190 (Exploit Public-Facing Application) to gain initial access."
- **Framework alignment:** Security teams worldwide use MITRE ATT&CK as the standard taxonomy for cyber threats.
- **Interview credibility:** Knowing MITRE ATT&CK IDs shows deep domain knowledge.

| Technique ID | Name | Our Tools |
|-------------|------|-----------|
| T1190 | Exploit Public-Facing Application | `autonomous_exploit_chain`, `autonomous_fuzz` |
| T1078 | Valid Accounts | `credential_spray`, KB default credentials |
| T1557 | Adversary-in-the-Middle | SSRF payloads in KB |
| T1550 | Use Alternate Authentication Material | JWT bypass patterns |

---
*Knowledge is the foundation. Memory is the edge. Evolution is the endgame.*
