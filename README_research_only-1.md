# 🛡️ SENTINEL-ONEWAY

**AI-Powered Passive Cyber Threat Detection for One-Way Network Monitoring**

> Research & Documentation Repository — **Smart India Hackathon 2026** — Team **Cyberknight007**

![status](https://img.shields.io/badge/status-research%20phase-yellow) ![license](https://img.shields.io/badge/license-MIT-blue)

> 📌 **Note:** This repo currently holds our research, problem analysis, and presentation only. The actual prototype/codebase is still being built and will be pushed here once it's ready — didn't want to sit on the research and not have it up anywhere in the meantime.

---

## 🧾 Table of Contents

- [Problem Statement](#-problem-statement)
- [Our Research Approach](#-our-research-approach)
- [Repository Structure](#-repository-structure)
- [System Architecture (Planned)](#-system-architecture-planned)
- [Presentation](#-presentation)
- [Research and References](#-research-and-references)
- [What SENTINEL-ONEWAY Aims to Solve](#-what-sentinel-oneway-aims-to-solve)
- [Feasibility Analysis](#-feasibility-analysis)
- [Potential Challenges & Risks](#-potential-challenges--risks)
- [Impact and Benefits](#-impact-and-benefits)
- [What's Coming Next](#-whats-coming-next)
- [Contributors](#-contributors)
- [License](#️-license)

---

## 👉 Problem Statement

**PS ID:** SIH26145
**Title:** AI-Based Detection of Cyber Threats in Unidirectional IP Traffic
**Organization:** National Technical Research Organisation (NTRO)
**Theme:** Blockchain & Cybersecurity
**Category:** Software

**Background (in short):** Some networks — power grids, defence systems, air-gapped government infra — use a "data diode" so traffic can only ever move in ONE direction. Nothing comes back. Great for security, but it also breaks every normal intrusion-detection tool, because they all assume you can see both sides of a conversation, or at least probe something to check. This PS asks: can you still catch attacks when you're only ever allowed to watch, never talk back?

---

## 💡 Our Research Approach

This PS genuinely confused us for the first couple of days before it clicked.

1. Broke the problem statement down line by line — "unidirectional IP traffic" sounds simple until you try to actually picture a monitoring system that physically can't send anything back.
2. Read up on data diodes and unidirectional gateways and realised this isn't some made-up constraint — NIST has an entire section on it in SP 800-82. That's when we stopped treating this as "just another IDS problem" and started treating the constraint itself as the actual challenge.
3. Went through public network-security datasets (UNSW-NB15, TON_IoT, BoT-IoT, CIC-IDS2018) trying to figure out what a "one-way" version of this data would even look like, since none of them are captured from a real diode.
4. Found out NetFlow records are technically one-directional at the flow level already — genuinely useful, meant we didn't have to invent a new data format from scratch.
5. Used Claude and a bit of Perplexity to speed up organizing all this reading and to sanity-check our architecture ideas against the papers — not to generate the idea for us, just to move faster through it.
6. Went back and forth a lot on deep learning vs. simpler models, and landed on Random Forest + Isolation Forest for the core design since we can actually explain what each one is doing, with LSTM-based stuff kept as a stretch goal.
7. Mapped out five attack types we can realistically demo (SYN flood, port scanning, C2 beaconing, DNS tunnelling, weird encrypted-traffic behaviour) and researched what a passive-only detector could actually observe for each.
8. Kept one question in the back of our heads the whole time: "why can't you just use a normal IDS here?" — every design choice had to have a real answer to that, not just "because AI."

Full write-up with everything we found, all the sources, and the honest gaps in the research (what we couldn't verify) is in the research report linked below.

---

## 📂 Repository Structure

```
sentinel-oneway/
│
├── docs/
│   ├── SIH26145_Research_Report.docx     # Full research report (20+ pages)
│   ├── presentation.pptx                 # SIH Idea Presentation deck
│   └── architecture-diagram.png          # System architecture (planned)
│
├── README.md
└── LICENSE
```

*(This will expand into a full codebase — see [What's Coming Next](#-whats-coming-next) below.)*

---

## 🧱 System Architecture (Planned)

```
Normal User + Attack Simulator  (Lab Network)
              │
              ▼
     Port Mirror / TAP  (passive copy only)
              │
              ▼
   One-Way Monitoring Path  — read-only, no return channel
              │
              ▼
  Traffic Ingestion + Feature Extraction
              │
              ▼
      One-Way-Aware Feature Engine
              │
     ┌────────┴─────────┐
     ▼                   ▼
Known-Threat Model   Anomaly Model
(Random Forest /     (Isolation Forest)
   XGBoost)
     └────────┬─────────┘
              ▼
        Threat Score Fusion
              ▼
    Evidence + Confidence Engine
              ▼
     Attack Correlation (timeline)
              ▼
        SOC Dashboard → Alert
```

This is the design we're building toward — implementation is in progress.

---

## 👓 Presentation

📊 [SENTINEL-ONEWAY — SIH26145 Idea Presentation](docs/presentation.pptx)

---

## 📚 Research and References

Full annotated report (20+ pages, 50+ sources) is in [`docs/SIH26145_Research_Report.docx`](docs/SIH26145_Research_Report.docx). A few of the sources that shaped our design directly:

- NIST SP 800-82 Rev. 3 — Guide to Operational Technology (OT) Security: [csrc.nist.gov](https://csrc.nist.gov/pubs/sp/800/82/r3/final)
- UNSW-NB15 Dataset — UNSW Canberra Cyber Range: [research.unsw.edu.au](https://research.unsw.edu.au/projects/unsw-nb15-dataset)
- NetFlow-based NIDS datasets (NF-* variants) — University of Queensland: [staff.itee.uq.edu.au/marius/NIDS_datasets](https://staff.itee.uq.edu.au/marius/NIDS_datasets/)
- MITRE ATT&CK Framework: [attack.mitre.org](https://attack.mitre.org)
- "Why Are You Weird? Infusing Interpretability in Isolation Forest" (arXiv:2112.06858)
- "Shift Detection and Adaptation for Network Intrusion Detection" (arXiv:2508.15100)
- NCIIPC — National Critical Information Infrastructure Protection Centre: [nciipc.gov.in](https://nciipc.gov.in)

---

## ✅ What SENTINEL-ONEWAY Aims to Solve

**1. Detecting threats with no return path**
Problem: Most IDS tools assume they can see both directions of traffic, or at least probe the network to check something. On a data-diode / one-way link, that's physically impossible.
Planned solution: A detection pipeline built entirely on features computable from outbound-only observation — no probing, no querying, no assumed response.

**2. Telling analysts *why*, not just *that***
Problem: A bare "Threat Detected — 91%" alert doesn't help a SOC analyst decide anything, and alert fatigue is a well-documented problem in this space.
Planned solution: Every alert paired with the actual behavioural evidence behind it, not just a confidence score.

**3. Making sense of scattered alerts**
Problem: Individual alerts in isolation don't tell you if you're looking at a coordinated attack or unrelated noise.
Planned solution: An attack-correlation layer linking related events into a rough timeline.

**4. Being honest about what AI can and can't see**
Problem: A lot of "AI security" projects quietly assume more visibility than they actually have.
Planned solution: Explicitly separating "the model isn't sure" from "we structurally can't observe this."

---

## 📈 Feasibility Analysis

- **Technical:** Plan relies entirely on open-source ML/network tooling — no proprietary NDR platform needed for the prototype.
- **Operational:** Fully passive by design — never touches or modifies the network it watches.
- **Data:** No public "real" one-way dataset exists as far as our research found, so the plan is to replay standard datasets unidirectionally and generate our own lab traffic. Flagging this limitation upfront rather than glossing over it.

---

## ⚓ Potential Challenges & Risks

- **Missing reverse-path info** → plan to mitigate with one-way-specific feature design
- **Encrypted traffic** → plan to handle via metadata-only analysis (JA3-style fingerprinting), not decryption
- **Dataset/domain shift** → known issue with ML-based IDS generally; plan to evaluate on held-out data rather than trust in-sample accuracy
- **False positives** → plan to tackle with evidence-based confidence scoring instead of a flat threshold
- **Unknown/novel attacks** → plan for multi-signal detection so no single feature carries all the weight

---

## 🪄 Impact and Benefits

- **Security:** Aims to extend real detection capability to environments currently relying almost entirely on the physical one-way guarantee alone.
- **Analyst workload:** Evidence-backed, correlated alerts instead of raw noise — targets the alert-fatigue problem SOC teams actually deal with.
- **Sector relevance:** Power grids, railways, defence networks, and other NCIIPC-classified critical infrastructure all use unidirectional monitoring patterns like this.
- **Not a replacement:** Meant to complement existing IDS/NDR specifically where return-path interaction isn't available — not replace it.

---

## 🚧 What's Coming Next

- [ ] 3-PC lab setup (normal traffic, attack simulator, passive monitor)
- [ ] Feature extraction pipeline
- [ ] Detection engine (Random Forest + Isolation Forest)
- [ ] Evidence + correlation engine
- [ ] SOC dashboard
- [ ] Live demo video

Code will be pushed to this same repo as each piece is ready — probably in stages rather than one big drop.

---

## ✨ Contributors — Team CyberNexus

1. [Your Name]
2. [Teammate 2]
3. [Teammate 3]
4. [Teammate 4]
5. [Teammate 5]
6. [Teammate 6]

*(swap these in with your actual names/GitHub handles before pushing)*

---

## 🛡️ License

This project is licensed under the [MIT License](LICENSE).
