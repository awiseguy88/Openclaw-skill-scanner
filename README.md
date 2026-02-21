

https://openclawskillscanner.aaronwiseai.com/


# Openclaw-skill-scanner
Tells you exactly what an openclaw skill does if it's safe if it is malicious 
# 🦞 ClawScan

**Local-first security scanner for [OpenClaw](https://openclaw.ai) skills — runs entirely in your browser, nothing sent to any server.**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![OpenClaw](https://img.shields.io/badge/Built%20for-OpenClaw-00e5a0)](https://openclaw.ai)
[![ClawHub](https://img.shields.io/badge/Skills%20Registry-ClawHub-00b87a)](https://clawhub.ai)
[![VirusTotal Partnership](https://img.shields.io/badge/Inspired%20by-VirusTotal-3949ab)](https://virustotal.com)

---

## What is ClawScan?

ClawScan is a **static security scanner** for OpenClaw skill packages (`.zip` files or raw code). Before you install a skill from ClawHub or anywhere else, drop it into ClawScan and get a full threat report in seconds — locally, with no data ever leaving your machine.

Think of it as a lightweight VirusTotal for OpenClaw skills, built specifically to understand the OpenClaw skill format, SKILL.md metadata, and the real threat surface of AI agent code.

> **OpenClaw** is a massively popular open-source personal AI assistant (100k+ GitHub stars) that runs locally and extends via community-built "skills." With over 5,700 skills in the ClawHub registry, not all of them are safe. ClawScan was built to close that gap.

---

## 🚀 Quick Start

**No install. No server. Just open the file.**

1. Download [`openclaw-scanner.html`](openclaw-scanner.html)
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge)
3. Drag and drop a skill `.zip` or paste code directly
4. Read the report

That's it. Every byte is analyzed in your browser using real JavaScript regex matching and JSZip extraction. Nothing phones home.

---

## 🔍 What It Detects

ClawScan runs **56 detection rules** across **11 threat categories**, inspired by real VirusTotal "Code Insights" findings on OpenClaw skills in the wild:

| Category | Rules | What It Catches |
|---|---|---|
| 🧠 **Prompt Injection** | 5 | Ignore-previous-instructions, DAN/jailbreak, role hijacking, safety bypass |
| 🎯 **Prompt Injection Vectors** | 3 | External prompt files, credential handling + injection, automated financial transactions |
| 🔒 **Code Obfuscation** | 8 | eval() abuse, base64 blobs, hex chains, new Function(), char codes, minification |
| 🔐 **Encrypted Payload** | 4 | OpenSSL runtime decrypt, AES/Fernet, zlib decompress, ROT13 |
| 📦 **Supply Chain Risk** | 3 | `npm install -g` global installs, unverified binary downloads, chmod+exec |
| 📡 **Network / C&C** | 8 | External URLs, curl-to-bash RCE, ngrok tunnels, reverse shells, raw IPs |
| 💾 **Data Exfiltration** | 7 | SSH/AWS dotfiles, /etc/passwd, Keychain dumps, clipboard reads, shell history |
| ⚙️ **System Tampering** | 7 | rm -rf /, crontab persistence, nohup backdoors, /tmp execution, LaunchAgents |
| 🔧 **Dangerous Defaults** | 3 | Hardcoded admin creds, unsanitized inputs, content extraction/scraping |
| ⚠️ **Behavioral Risk** | 5 | Eval of project code (RCE surface), file mutation, package installs, sub-agents |
| 📋 **Metadata Mismatch** | 1 | Env vars used in code but not declared in SKILL.md frontmatter |

### What Makes ClawScan Different

**Context-aware analysis.** A security scanner that scans *for* `eval()` patterns should not itself be flagged for containing the word `eval`. ClawScan understands this — it checks surrounding lines before flagging, and automatically downgrades false positives with an explanation.

**Auto-decode obfuscated content.** When a base64 blob or hex escape chain is found, ClawScan decodes it on the spot and shows you the plaintext right in the report. You see exactly what was being hidden — without copy-pasting anything to CyberChef.

**Exact location with context window.** Every finding shows the matched file, line number, the exact matched string highlighted in red, and 4 lines of surrounding code above and below — so you can understand what the suspicious code is actually doing.

**Smart env var tracking.** ClawScan distinguishes between environment variables a user must supply (secrets, API keys) vs. script-internal state variables that the scripts define and consume themselves. This cuts false-positive noise dramatically.

**Trusted registries.** Calls to `registry.npmjs.org`, `pypi.org`, `crates.io` are flagged as ✅ safe — they are dependency verification, not exfiltration. `localhost` and `127.0.0.1` are trusted as the normal OpenClaw gateway. Private IP ranges (`10.x`, `192.168.x`, `172.16–31.x`) are excluded from external URL checks.

---

## 📊 Scoring

| Score | Verdict |
|---|---|
| 80–100 | ✅ **Safe** — No significant threats |
| 50–79 | ⚠️ **Review Required** — Patterns of concern found |
| 0–49 | 🚨 **High Risk** — Do not install without understanding every finding |

Severity weights: `critical = -35pts`, `high = -15pts`, `medium = -6pts`, `low = -2pts`, `info = 0pts`

---

## 🧪 Real-World Test Cases

These skills were scanned against VirusTotal's Code Insights to validate ClawScan's detection accuracy:

| Skill | VirusTotal Finding | ClawScan Detection |
|---|---|---|
| `wreckit v2.1.0` | eval "$TEST_CMD" RCE surface, openclaw.json read | ✅ `beh1` + `ex2b` — correct context on both |
| `kagi-summarizer` | Binary download from GitHub, no checksum | ✅ `sc2` + `sc3` — supply chain flagged |
| `polymarket-bot` | Prompt injection vector + private key handling + crypto trades | ✅ `piv1` + `piv2` + `piv3` — all three caught |
| `skillgate-gov` | `npm install -g @skillgate/openclaw-skillgate` | ✅ `sc1` — global install flagged as critical |
| `dom-observer-pro` | Content extraction / scraping capability | ✅ `dd3` — noted with appropriate context |
| `rpe-grafana` | Hardcoded admin default + unsanitized API params | ✅ `dd1` + `dd2` — both flagged |

---

## 🏗️ Architecture

ClawScan is intentionally a **single HTML file** with zero dependencies beyond two CDN libraries:

```
openclaw-scanner.html
├── CSS                     — Brutalist dark theme (IBM Plex Mono + Syne)
├── HTML                    — Upload zone, paste panel, results area
└── JavaScript
    ├── CHECKS[]            — 56 detection rules as regex objects
    ├── readAllFiles()      — JSZip extraction + FileReader API (real, not mocked)
    ├── runAnalysis()       — Line-by-line matching with context capture
    ├── parseSkillMeta()    — SKILL.md YAML frontmatter parser
    ├── renderResults()     — Full report renderer
    ├── tryDecodeBase64()   — Auto-decode base64 → plaintext reveal
    ├── tryDecodeHex()      — Auto-decode hex chains → plaintext reveal
    └── renderContext()     — 4-line context window renderer per finding
```

**Libraries used:**
- [`jszip@3.10.1`](https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js) — ZIP extraction in the browser
- [Google Fonts](https://fonts.google.com) — IBM Plex Mono + Syne

**No backend. No telemetry. No accounts. No API keys.**

---

## 🔒 Security Philosophy

ClawScan was built with the same security mindset it applies to the skills it scans:

- **Zero network calls** during analysis — all processing is local
- **No localStorage** — nothing persisted between sessions
- **No eval()** — the scanner itself uses no dynamic code execution
- **Read-only** — ClawScan never writes to disk
- **Open source** — every rule is readable in the HTML source

---

## 🛠️ Contributing New Rules

Each detection rule follows this structure:

```javascript
{ id:"category_n",
  cat:"Category Name",
  icon:"🔒",
  sev:"critical|high|medium|low|info",
  label:"Short human-readable label",
  re:/your-regex-here/i,
  desc:"What this pattern means and why it matters.",
  action:"What the user should do about it.",
  contextExempt: true,                          // optional
  contextExemptIf: /surrounding-context-re/i    // optional — auto-downgrade if context matches
}
```

**Severity guide:**
- `critical` — Do not install. Clear malicious or catastrophic-risk pattern.
- `high` — Serious concern. Requires investigation before installing.
- `medium` — Pattern of concern. May be legitimate but warrants review.
- `low` — Minor flag. Likely benign but worth noting.
- `info` — Informational only. Expected behavior, no action needed.

---

## 🔗 Resources
https://openclawskillscanner.aaronwiseai.com/

**OpenClaw Ecosystem:**
- [OpenClaw](https://openclaw.ai) — The personal AI assistant ClawScan protects
- [ClawHub](https://clawhub.ai) — OpenClaw skill registry (5,700+ skills)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw) — Open source core
- [OpenClaw Security](https://github.com/openclaw/openclaw/security) — Official security policy
- [OpenClaw Docs](https://docs.openclaw.ai) — Skills, gateway, configuration

**Security References:**
- [VirusTotal](https://virustotal.com) — ClawScan's detection patterns were validated against real VirusTotal Code Insights reports on OpenClaw skills
- [Awesome OpenClaw Skills](https://github.com/VoltAgent/awesome-openclaw-skills) — Curated skill directory (3,002 skills reviewed)
- [OWASP Top 10 for LLMs](https://owasp.org/www-project-top-10-for-large-language-model-applications/) — Prompt injection, supply chain, and other AI-specific risks
- [CyberChef](https://gchq.github.io/CyberChef) — Decode base64, hex, and other obfuscation manually

---

## 👤 Author

**Aaron Wise** — AI & ML Consultant based in Honesdale, Pennsylvania

I help small businesses implement AI that actually works. ClawScan was built because I was scanning OpenClaw skills and needed something better than manual code review. So I built it.

- 🌐 [aaronwiseai.com](https://aaronwiseai.com) — AI consulting for small business
- 💼 [LinkedIn](https://www.linkedin.com/in/aaron-wise-31b00349)
- 📧 [wisesol2001@gmail.com](mailto:wisesol2001@gmail.com)
- 📞 570-299-1596

**Certifications:** Microsoft AI & ML Engineering · Google Project Management · Foundations of AI and Machine Learning · Statistical Modeling & Scalable Systems

---

## 📄 License

MIT — use it, fork it, improve it, ship it.

---

## ⭐ If This Helped You

If ClawScan caught something nasty before you installed a skill, consider:
- Reporting the malicious skill on ClawHub
- Sharing ClawScan with the OpenClaw community
- Opening an issue here with the detection pattern so others benefit

> *"Agent skills can include prompt injections, tool poisoning, hidden malware payloads, or unsafe data handling patterns. Always review the source code before installing."*
> — [Awesome OpenClaw Skills](https://github.com/VoltAgent/awesome-openclaw-skills)

ClawScan makes that review automatic. 🦞
