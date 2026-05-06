# Gap Analysis: Migrating `superman-splunk` to Microsoft Copilot Studio

## Executive Summary

Migrating the `superman-splunk` skill to Microsoft Copilot Studio is very achievable, but it is not a lift-and-shift packaging exercise. The core expert persona, operating rules, and Splunk domain knowledge transfer well into Copilot Studio through the instructions field and curated knowledge sources. Most of the subject-matter value in the current skill is content-based, and that part ports cleanly.

The harder part is behavioral parity. The current skill format assumes deterministic activation cues, explicit packaging, and direct reference-file logic. Copilot Studio is more orchestration-driven and retrieval-driven. That means the migration succeeds best when the team intentionally redesigns activation, grounding, and governance rather than trying to recreate the original skill mechanics one-for-one. In practice, this is a medium-complexity migration for knowledge and persona, and a higher-complexity migration for activation behavior, evaluation, and cross-platform portability.

| Current Skill Element | Current Purpose | Copilot Studio Equivalent | Porting Difficulty | Notes |
|---|---|---|---|---|
| YAML frontmatter description (activation trigger) | Describes when the skill should activate for Splunk-related requests. | Agent description, instructions, starter prompts, and generative orchestration signals. | Medium | Copilot Studio can be guided strongly, but it does not provide the same deterministic skill trigger contract as YAML frontmatter in a portable skill package. |
| Domain routing table in `SKILL.md` | Explains which Splunk subdomains the skill covers and when to use them. | Instructions plus knowledge source naming and descriptions; optional custom topics for special workflows. | Low | The routing intent transfers well if the instructions and knowledge descriptions are explicit. |
| "Read this reference file" logic | Forces the assistant to consult a specific file for specific domains. | Knowledge sources grounded through SharePoint or other supported knowledge stores. | High | Copilot Studio retrieval is relevance-based, not deterministic file loading. You can influence it, but not guarantee a specific file will be used every time. |
| Expert behavior rules | Enforces style and safety expectations such as asking for version, noting restart impact, and distinguishing tiers. | Agent instructions. | Low | This is one of the easiest parts to port and should remain in the instructions field rather than only in knowledge files. |
| Quick reference tables (ports, conf files) | Provides compact operational facts the model can reuse frequently. | Short instruction snippets, SharePoint documents, or a dedicated glossary document. | Low | These transfer well and often retrieve better when broken into small reference documents. |
| Per-reference `.md` knowledge files | Supplies durable domain guidance across architecture, SPL, development, admin, and security. | SharePoint documents, SharePoint pages, Dataverse knowledge rows, or connector-based knowledge. | Low | The content ports well, but it should usually be re-chunked into smaller enterprise-friendly documents. |
| Eval prompts in `evals.json` | Tests whether the skill behaves correctly across expected scenarios. | Manual prompt test suite, pilot validation scripts, analytics review, and governed regression prompt set. | Medium | Copilot Studio lacks a direct equivalent to a portable skill-local eval file; testing becomes an operational process. |
| Skill packaging (`.skill` file) | Packages the skill for reuse across compatible assistant environments. | Copilot Studio agent, solution export, environment-managed assets. | High | Copilot Studio packages agents and solutions differently and ties them to Power Platform governance and environment lifecycle. |
| Automatic skill activation | Makes the skill available when the user mentions Splunk. | Generative orchestration, topic matching, starter prompts, and agent selection context. | High | You can approximate this behavior, but not with the same platform-agnostic activation semantics. |
| Multi-platform portability (Claude/Copilot CLI/Gemini) | Keeps one skill usable across multiple assistant runtimes. | No direct equivalent; build a Copilot Studio-specific agent configuration. | High | Copilot Studio is optimized for Microsoft ecosystem deployment, not neutral multi-runtime portability. |

## What Cannot Be Directly Ported

1. **Deterministic skill activation metadata**: Copilot Studio does not consume the original YAML frontmatter trigger model directly. Activation becomes orchestration- and prompt-driven.
2. **Deterministic file-loading behavior**: The original skill can conceptually direct the model to use a specific reference file. Copilot Studio knowledge grounding is retrieval-based and probabilistic.
3. **Portable skill packaging across assistant platforms**: A `.skill` package is fundamentally different from a Copilot Studio agent with Microsoft-specific governance, environment, and connector assets.
4. **Local skill eval packaging**: There is no one-to-one equivalent for shipping the original eval definitions inside the Copilot Studio agent package and expecting the same runtime behavior.

## What Transfers Better in Copilot Studio

1. **Enterprise governance**: Copilot Studio gives you Microsoft 365 access control, Power Platform DLP, environment management, and solution-based lifecycle controls that are stronger than a lightweight portable skill package.
2. **SharePoint-native knowledge grounding**: For organizations already standardized on SharePoint and Microsoft 365, knowledge publishing and access control become easier to operationalize.
3. **Channel deployment and authentication**: Publishing into Teams, Microsoft 365 Copilot, or other supported channels is much more natural in Copilot Studio than in portable skill systems.
4. **Future tool extensibility**: Once governance is in place, Copilot Studio can add connectors, flows, and controlled enterprise actions in a way that fits broader Power Platform operating models.

---

## What Is Lost in the Migration

These are real capability degradations or constraints that occur when moving from the original skill format to Copilot Studio. Acknowledge them rather than minimizing them.

| What Is Lost | Why It Matters | Mitigation |
|---|---|---|
| **Deterministic reference file loading** | The original skill can explicitly load and ground against a specific file for a given query. Copilot Studio retrieves by relevance; the wrong document may be retrieved or two documents may conflict. | Pre-chunk documents by domain; write strong source descriptions; test retrieval with a fixed prompt suite after any knowledge change. |
| **Multi-platform portability** | The original skill runs on Claude Code, Copilot CLI, Gemini CLI, and any compatible skill host. The Copilot Studio agent is a Microsoft 365–bound configuration asset. | Maintain the original skill for non-Microsoft environments. Treat the Copilot Studio agent as a separate product, not a replacement. |
| **Portable skill evals** | `evals.json` ships with the skill package and can be run in CI. Testing in Copilot Studio is a manual or operational process with no equivalent to run-anywhere eval pipelines. | Build a static prompt test file and run it manually at each instruction or knowledge update. Track results in a spreadsheet or test-management system until platform evaluation tooling matures. |
| **Skill package reproducibility** | The original skill is git-controlled, shareable, and re-deployable from the source repo in minutes. A Copilot Studio agent lives in a Dataverse environment and is exported as a solution file, not as a readable source format. | Export the solution on a cadence and version it in source control. Document configuration in markdown alongside the exported solution. |
| **Sub-100ms activation latency** | Skill loading is nearly instant in lightweight CLI environments. Copilot Studio generative orchestration adds inference and retrieval overhead. | Set user expectations. Knowledge-only responses typically take 3–10 seconds. With tool-call round trips, 15–30 seconds is realistic. |
| **Zero infrastructure dependency** | The original skill requires no backend infrastructure beyond a compatible assistant runtime. Copilot Studio requires Dataverse, SharePoint, Power Platform licensing, and network connectivity for tools. | Ensure Phase 0 prerequisites are fully satisfied before piloting with real users. |

---

## What Is Gained in the Migration

These are real benefits the Copilot Studio format delivers that the original skill does not.

| What Is Gained | Why It Matters |
|---|---|
| **Microsoft 365 native access control** | Knowledge files inherit SharePoint permissions. Sensitive ES and security content is restricted to appropriate groups automatically. This is difficult to enforce in a portable skill that loads local files. |
| **Enterprise governance and DLP** | Power Platform DLP policies govern what data connectors can move, where data goes, and who can publish agents. This is a requirement in most large enterprise environments. |
| **Teams, M365 Copilot channel deployment** | Users interact with the Splunk expert agent inside existing productivity tools without additional client installation or CLI tooling. Adoption is lower friction in enterprise settings. |
| **Approval workflows and Power Automate integration** | Phase 3 admin actions can be wrapped in approval loops using native Power Automate functionality. This capability does not exist in the lightweight skill format. |
| **Solution lifecycle management** | Agents can be promoted across dev, test, and production environments using Power Platform solution management. Change approval and rollback follow the same governance model as other enterprise Power Platform assets. |
| **Analytics and conversation telemetry** | Copilot Studio provides built-in conversation analytics, escalation signals, resolution rates, and satisfaction tracking. Lightweight skill runtimes offer no equivalent observability. |

---

## What Must Be Redesigned, Not Merely Ported

These areas require intentional redesign rather than a copy-and-adjust approach. Simply porting the original content without addressing these will produce a Copilot Studio agent that behaves worse than the original skill.

### 1. Activation Semantics
The original skill relies on a contract between the skill host and the YAML frontmatter trigger. Copilot Studio uses generative orchestration, topic matching, and starter prompts to decide when to engage knowledge or topics. This is a different model. The agent instructions and description must do the work that the YAML frontmatter did — clearly signal scope, persona, and domain so the orchestration layer behaves predictably.

### 2. Knowledge Grounding Strategy
The original skill explicitly loads reference files as context. Copilot Studio retrieves from knowledge sources. If the knowledge files are large, poorly chunked, weakly described, or structured identically to each other, retrieval will perform poorly regardless of how good the instructions are. The knowledge layer must be actively designed, not passively migrated. This means chunking by task or conf domain, naming sources precisely, and testing retrieval as a first-class concern.

### 3. Evaluation and Regression Testing
The original skill package includes eval prompts that can be run repeatably and portably. Copilot Studio has no portable eval mechanism yet. This must be redesigned as an operational practice: a static prompt test file, a named Splunk SME who reviews results, and a clear trigger for re-testing (instructions change, knowledge documents update, Splunk version milestone). Without this, quality degrades invisibly.

### 4. Version and Platform Branching
The original skill contains version-aware and platform-aware guidance in reference files. In Copilot Studio, this branching must happen at two levels: in the instructions (behavior rules that tell the agent to ask before answering) and in the knowledge documents (content that explicitly labels version ranges and platform applicability). Neither layer alone is sufficient. Both must be redesigned in coordination.

### 5. Governance and Lifecycle Ownership
A lightweight skill can be owned by a single developer and updated in a text editor. A Copilot Studio agent in a governed enterprise is a shared service with multiple stakeholders, change control, licensing implications, and data access boundaries. This requires formal ownership structure, change approval, and a refresh cadence — none of which exist in the original skill format and all of which must be designed from scratch.
