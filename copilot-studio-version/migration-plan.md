# Splunk Skill to Copilot Studio Migration Plan

## Phase 0: Prerequisites & Assessment

**Estimated Effort:** 0.5–1.5 days (developer sandbox) / 2–6 weeks (governed enterprise with change control, security review, and IT governance)  
**Primary Owner:** Copilot Studio admin with Splunk SME and IT governance support  
**Key Risks:** Missing licensing, unclear ownership, no approved SharePoint location, unclear production versus non-production boundaries, network connectivity to Splunk port 8089 not yet approved

> **Enterprise timing note:** Effort estimates in this plan reflect a developer sandbox with Dataverse, SharePoint, and licensing already in place. For a governed enterprise deployment — with change control, security review, DLP policy assessment, and IT governance — Phase 0 through Phase 3 realistically takes 4–8 weeks. Plan network connectivity prerequisites (see `actions-tools-design.md`) before committing to Phase 2.

### Objectives
- Confirm the target Copilot Studio environment and licensing model
- Confirm whether the agent will serve Microsoft Teams, Microsoft 365 Copilot, a custom website, or multiple channels
- Identify Splunk stakeholders: platform owner, ES owner, app developers, and security governance
- Decide whether the first release is knowledge-only or will include live tools

### Checklist
1. Confirm Microsoft 365 Copilot licensing (E3/E5 + Copilot add-on) or Copilot Studio standalone licensing. These are different SKUs with different feature access — verify which applies to your environment before proceeding.
2. Confirm Copilot Studio access, Dataverse environment, and publishing rights.
3. Confirm Microsoft 365 access model and the SharePoint site that will host knowledge files.
4. Confirm the initial operating model: knowledge-only, read-only tools, or staged rollout.
5. Identify which Splunk environments are in scope: dev, test, production; Enterprise, Cloud, or both.
6. **Plan network connectivity** if Phase 2 tools are planned: Splunk REST API runs on port 8089. Confirm whether a VPN, ExpressRoute, on-premises data gateway, or API proxy is required to reach it from Power Automate. See `actions-tools-design.md` for details.
7. Agree on ownership for instructions, knowledge updates, testing, and approval of future tools.

## Phase 1: Extract & Prepare Instructions

**Estimated Effort:** 0.5 day  
**Primary Owner:** Splunk SME  
**Key Risks:** Overly long instructions, losing critical safety behavior, not translating skill activation logic into Copilot Studio behavior

### Steps
1. Convert the existing `SKILL.md` persona into Copilot Studio instruction language using direct statements such as "You are... Your role is... When the user asks..., you should...".
2. Remove YAML frontmatter and any references to manually reading local reference files.
3. Preserve the expert behavior rules: version awareness, deployment-awareness, Cloud-versus-Enterprise distinctions, SPL performance guidance, configuration precedence, and production safety warnings.
4. Add explicit instruction requirements for every configuration answer: conf file, stanza, attribute, scope, and restart or reload requirement.
5. Trim the final instruction block so it fits comfortably within Copilot Studio's instruction limit and leaves room for future revisions.
6. Produce both a markdown-maintained version and a paste-ready plain-text version for the Copilot Studio instructions field.

## Phase 2: Prepare Knowledge Files

**Estimated Effort:** 1-2 days  
**Primary Owner:** Splunk SME with SharePoint content owner  
**Key Risks:** Poor retrieval due to oversized documents, mixing Cloud and Enterprise guidance in one chunk, weak access control on security content

### Steps
1. Take each reference markdown file and **convert it to `.docx` using Pandoc** (`pandoc input.md -o output.docx`) before uploading to SharePoint. Markdown (`.md`) is not an officially supported Copilot Studio knowledge file type. Raw `.md` uploads may degrade chunking quality because markdown syntax characters pollute the extracted text. Supported knowledge file formats include `.docx`, `.pdf`, `.pptx`, and `.txt`.
2. Split large files into smaller knowledge assets by task area: architecture, SPL, app development, admin configuration, and security or ES.
3. Separate high-risk or security-sensitive material into a restricted SharePoint library or site.
4. Use descriptive titles and detailed descriptions for every knowledge source; Copilot Studio retrieval quality improves when the description clearly states when that source should be used.
5. Add a small glossary or acronym document for UF, HF, SHC, RF, SF, CIM, RBA, HEC, and notable event.
6. Validate document permissions with Microsoft 365 groups before connecting them to the agent.

## Phase 3: Create the Copilot Studio Agent

**Estimated Effort:** 0.5 day  
**Primary Owner:** Copilot Studio admin  
**Key Risks:** Incorrect environment selection, wrong authentication mode, not enabling the right orchestration behavior

### Steps
1. Open Copilot Studio and select the target environment.
2. Create a new agent named something like `Splunk Expert` or `Splunk Platform Advisor`.
3. Add a concise description that tells the orchestration layer when this agent is appropriate, for example: Splunk architecture, SPL, administration, ES, and development guidance.
4. Paste the prepared instructions into the agent instructions field.
5. Confirm the authentication model for the intended channel, especially if SharePoint knowledge will be used through Teams or Microsoft 365 Copilot.
6. Save the draft agent before adding knowledge or tools.
7. If using solution management, place the agent and related connector assets in a governed Power Platform solution from day one.

## Phase 4: Configure Knowledge & Grounding

**Estimated Effort:** 0.5-1 day  
**Primary Owner:** Copilot Studio admin with SharePoint owner  
**Key Risks:** Incorrect SharePoint permissions, weak source descriptions, poor retrieval from oversized or ambiguous documents

### Steps
1. In the agent, open **Knowledge** or use **Add knowledge** from the agent overview.
2. Add the SharePoint site, folders, or individual files prepared in Phase 2.
3. Name each knowledge source clearly and write a detailed description explaining when it should be used.
4. Keep architecture, SPL, admin, development, and ES content as separate sources rather than one combined library entry.
5. If using SharePoint lists or other real-time knowledge sources, verify user authentication behavior and access inheritance.
6. Turn on improved retrieval features available in your tenant, such as Work IQ where licensed and supported.
7. Test representative prompts immediately after each source is added rather than waiting until the end.

## Phase 5: Configure Prompts & Persona

**Estimated Effort:** 0.5 day  
**Primary Owner:** Copilot Studio admin with Splunk SME  
**Key Risks:** Starter prompts that are too generic, welcome content that conflicts with expert persona, prompt wording that pulls the wrong knowledge source

### Steps
1. Add the curated starter prompts for SPL, architecture, administration, ES, and development use cases.
2. Ensure the welcome experience signals the agent's scope without making risky promises such as direct production changes.
3. Verify the persona remains technical, direct, and safety-aware.
4. If custom topics are used, add a small set only for governed workflows such as escalation, tool-use confirmation, or change-request handoff.
5. Keep prompt examples realistic and enterprise-specific so users understand the intended value immediately.

## Phase 6: Testing & Validation

**Estimated Effort:** 1-2 days  
**Primary Owner:** Splunk SME with Copilot Studio admin and pilot users  
**Key Risks:** Hallucinated configuration guidance, missing Splunk Cloud caveats, inconsistent conf-file formatting, overconfident answers when version or topology is unknown

### What to test
- SPL generation quality for security, operations, and troubleshooting use cases
- Enterprise versus Cloud differentiation
- Version-sensitive and topology-sensitive clarifying questions
- Configuration answers that must include conf file, stanza, attribute, scope, and restart or reload requirements
- ES and security guidance quality
- Grounding behavior against the correct knowledge source

### Acceptance criteria
- The agent asks for version or deployment type when it materially affects the answer.
- Configuration guidance consistently includes file, stanza, attribute, scope, and restart or reload notes.
- The agent distinguishes Splunk Cloud constraints from Splunk Enterprise filesystem or admin behavior.
- SPL answers are complete, realistic, and use production-aware patterns.
- Retrieval consistently cites the relevant architecture, SPL, admin, development, or security source.

### Red flags
- The agent gives filesystem advice for Splunk Cloud without warning.
- The agent proposes production edits in `/default`.
- The agent omits restart impact or cluster-wide scope.
- The agent retrieves the wrong source for a straightforward question.

## Phase 7: Pilot Rollout

**Estimated Effort:** 1-2 weeks  
**Primary Owner:** IT governance with Copilot Studio admin and Splunk service owner  
**Key Risks:** Releasing too broadly before quality is stable, no feedback loop, insufficient user expectations for knowledge-only versus tool-enabled behavior

### Steps
1. Publish the agent to a limited Microsoft 365 group such as the Splunk admin team, SOC engineering, and a few senior analysts.
2. Communicate the initial scope clearly: guidance-only or read-only tools only.
3. Collect structured feedback on answer quality, missing content, incorrect assumptions, and high-value future tools.
4. Review conversation analytics and failed-answer patterns weekly during the pilot.
5. Tune instructions, knowledge descriptions, and knowledge chunking before broadening access.

## Phase 8: Production Deployment & Governance

**Estimated Effort:** 1-3 days for go-live, then ongoing operational ownership  
**Primary Owner:** IT governance and service owner  
**Key Risks:** No lifecycle owner, uncontrolled instruction drift, undocumented knowledge refresh process, tool rollout without approval controls

### Steps
1. Publish the validated agent to the approved production channels.
2. Assign a named service owner, backup owner, and change approver.
3. Define a refresh cadence for instructions, knowledge sources, and starter prompts.
4. Document how Splunk version changes, ES content changes, and Cloud policy changes trigger knowledge updates.
5. If Phase 2 or Phase 3 tools are planned, require separate design review, security approval, and production readiness review before enabling them.
6. Review agent analytics, unresolved questions, and risky answer patterns on a monthly cadence.
7. Treat the agent as a governed enterprise service, not as static documentation.
