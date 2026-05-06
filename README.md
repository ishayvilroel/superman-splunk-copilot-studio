# 🦸 Superman Splunk — Microsoft Copilot Studio Version

> **A complete Microsoft Copilot Studio migration design and planning package** that transforms the `superman-splunk` AI skill into an enterprise Splunk expert agent for Microsoft 365 environments.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/platform-Microsoft%20Copilot%20Studio-blue)](https://copilotstudio.microsoft.com)
[![Source Skill](https://img.shields.io/badge/source-superman--splunk-blueviolet)](https://github.com/ishayvilroel/superman-splunk)
[![M365](https://img.shields.io/badge/Microsoft%20365-Copilot%20Ready-green)](https://github.com/ishayvilroel/superman-splunk-copilot-studio)

---

## Current Status

This is a **v1.2 implementation-ready design package**. It is not a deployed agent, a tested system, or a plug-and-play product. It is a detailed migration blueprint: structured instructions, a knowledge architecture, a phased tool design, a migration plan, and a gap analysis intended for implementation by a qualified team.

To deploy a working Copilot Studio agent from this package, you need: a Microsoft 365 / Copilot Studio environment, a Dataverse environment, a SharePoint site, a Splunk SME to review and adapt the instructions to your environment, and a testing phase against real prompts. See [`migration-plan.md`](copilot-studio-version/migration-plan.md) for prerequisites and phase exit criteria.

---

## Who This Is For

- **Splunk architects and administrators** who want to expose Splunk expertise through a Microsoft 365 enterprise agent
- **Security engineers and SOC teams** working in M365 environments who use Splunk ES, SOAR, or ITSI
- **Microsoft 365 / Copilot Studio implementers** who have been asked to build a Splunk advisory agent and want a well-structured starting point
- **Enterprise architects** evaluating whether a Superpowers-style AI skill can be translated into a governed Copilot Studio deployment

This package assumes familiarity with both Splunk administration and Microsoft Copilot Studio. It is not an introductory guide to either platform.

---

## What Is This?

This repository contains the **Microsoft Copilot Studio migration** of the [`superman-splunk`](https://github.com/ishayvilroel/superman-splunk) skill — a Splunk expert AI designed for enterprise Microsoft 365 environments.

The original `superman-splunk` skill was built for Superpowers-compatible agents (Claude Code, GitHub Copilot CLI, Gemini CLI). This version translates that same deep Splunk expertise into **Copilot Studio's native agent format**: agent instructions, SharePoint knowledge sources, suggested prompts, and a phased tool/action design.

---

## What's Inside

```
copilot-studio-version/
├── agent-instructions.md          # ← Paste directly into Copilot Studio Instructions
├── copilot-studio-paste-ready.txt # ← Plain-text version optimized for the Instructions field
├── knowledge-mapping.md           # ← How to set up SharePoint knowledge sources
├── suggested-prompts.md           # ← 12 enterprise starter prompts (SPL, Arch, ES, Dev, Admin)
├── actions-tools-design.md        # ← Phase 1/2/3 tool strategy with REST API actions
├── migration-plan.md              # ← Step-by-step migration from skill repo → live agent
└── gap-analysis.md                # ← What ports well, what needs redesign, what can't transfer
```

---

## Quick Start

> **Note:** "Quick Start" here means: start the implementation process. Building a working enterprise agent requires prerequisites (licensing, SharePoint, Splunk SME review) and a multi-phase rollout. The steps below outline the path, not a 30-minute setup. Read [`migration-plan.md`](copilot-studio-version/migration-plan.md) for full phase detail.

### 1. Confirm prerequisites

Confirm Microsoft 365 Copilot or Copilot Studio standalone licensing, a provisioned Dataverse environment, and a SharePoint site designated for agent knowledge. Identify the Splunk SME who will review and adapt the instructions for your environment.

### 2. Create the agent in Copilot Studio

1. Go to [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com)
2. Create a new agent → name it **"Splunk Expert"**
3. Open the **Instructions** field
4. Paste the contents of [`copilot-studio-version/copilot-studio-paste-ready.txt`](copilot-studio-version/copilot-studio-paste-ready.txt)

### 3. Add knowledge sources

Convert the 5 reference files from [superman-splunk](https://github.com/ishayvilroel/superman-splunk) to `.docx` format (Markdown is not a supported Copilot Studio knowledge type — use `pandoc input.md -o output.docx`), upload to SharePoint, then add to the agent's knowledge:

| SharePoint Document Title | Source File |
|--------------------------|-------------|
| Splunk Architecture & Infrastructure | `references/architecture.md` |
| Splunk SPL Guide & Knowledge Objects | `references/spl-guide.md` |
| Splunk App & TA Development | `references/development.md` |
| Splunk Administration & Configuration | `references/administration.md` |
| Splunk Security, ES & SOAR | `references/security-es.md` |

See [`knowledge-mapping.md`](copilot-studio-version/knowledge-mapping.md) for chunking strategy, retrieval failure modes, and SharePoint governance structure.

### 4. Add starter and validation prompts

Add the prompts from [`suggested-prompts.md`](copilot-studio-version/suggested-prompts.md). The file includes 12 enterprise operational prompts and 3 validation prompts for acceptance testing.

### 5. Follow the migration plan

See [`migration-plan.md`](copilot-studio-version/migration-plan.md) for the full phased rollout — from prerequisites and exit criteria through pilot deployment and production governance.

---

## File Details

### [`agent-instructions.md`](copilot-studio-version/agent-instructions.md)
Production-grade agent instructions in Copilot Studio format. Covers:
- Splunk expert persona
- SPL and configuration response rules
- Required clarifying questions (Enterprise vs Cloud, version, topology)
- Production safety rules (change scope, restart requirements, cluster-wide impact)
- What the agent must not do

### [`copilot-studio-paste-ready.txt`](copilot-studio-version/copilot-studio-paste-ready.txt)
Plain-text version of the instructions, optimized for direct paste into Copilot Studio's Instructions field (~8,000-character limit). At ~4,200 characters it leaves substantial headroom for future additions.

### [`knowledge-mapping.md`](copilot-studio-version/knowledge-mapping.md)
Maps each of the 5 reference files to a recommended Copilot Studio knowledge strategy. Includes:
- Recommended SharePoint document titles
- Chunking and grouping strategy per file
- Update frequency recommendations
- Governance and access control guidance
- Retrieval optimization tips

### [`suggested-prompts.md`](copilot-studio-version/suggested-prompts.md)
12 enterprise-quality starter prompts grouped by domain:
- **SPL** (3): security detection, error rate analysis, cross-index correlation
- **Architecture** (2): cluster design, SmartStore planning
- **Administration** (2): conf file help, performance troubleshooting
- **Enterprise Security & ES** (3): RBA setup, notable event triage, threat intel
- **Development** (2): TA development, custom command development

### [`actions-tools-design.md`](copilot-studio-version/actions-tools-design.md)
Phased tool design for live Splunk connectivity:
- **Phase 1** (recommended start): Knowledge-only, no live connections
- **Phase 2**: Read-only REST API tools (search dispatch, index stats, deployment status)
- **Phase 3**: Controlled admin actions (with approval workflows and audit trails)
Includes authentication model, risk level, and governance guidance per action.

### [`migration-plan.md`](copilot-studio-version/migration-plan.md)
8-phase migration plan from this repo to a live production agent:
- Phase 0: Prerequisites & Assessment
- Phase 1–2: Extract instructions, prepare knowledge files
- Phase 3–5: Create agent, add knowledge, configure prompts
- Phase 6: Testing & validation with acceptance criteria
- Phase 7–8: Pilot rollout → production governance

### [`gap-analysis.md`](copilot-studio-version/gap-analysis.md)
Structured comparison of the current skill format vs Copilot Studio. Covers 10 skill elements with porting difficulty ratings. Identifies what cannot be directly ported (deterministic activation, file-loading logic, portable packaging) and what is actually better in Copilot Studio (enterprise governance, SharePoint-native knowledge, Teams/M365 channels).

---

## Source Material

This Copilot Studio version is derived from:
- **[superman-splunk](https://github.com/ishayvilroel/superman-splunk)** — the original Superpowers skill (research methodology: 300+ web sources via Google NotebookLM)
- The superman-splunk source skill achieved a **96% pass rate vs 73% without-skill (+23pp)** across 5 Splunk benchmark scenarios. That benchmark covers the source skill only — this Copilot Studio migration package has not been independently evaluated. Use the acceptance criteria and suggested prompts in [`migration-plan.md`](copilot-studio-version/migration-plan.md) and [`suggested-prompts.md`](copilot-studio-version/suggested-prompts.md) as your evaluation framework when validating the deployed agent.

---

## License

MIT © [ishayvilroel](https://github.com/ishayvilroel)

---

*Built with [Superpowers AI](https://github.com/superpowers-ai/superpowers) skill-creator tooling.*
