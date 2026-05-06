# Copilot Studio Knowledge Mapping for the Splunk Expert Agent

## `references/architecture.md`

- **Content Summary**: Splunk deployment topology, forwarder and heavy forwarder roles, indexer pipeline stages, bucket lifecycle, SmartStore, indexer clustering, search head clustering, deployment server behavior, knowledge bundle handling, configuration precedence, and platform performance tuning.
- **Recommended Knowledge Type**: SharePoint document
- **Recommended Title**: Splunk Architecture and Core Platform Guide
- **Chunking Strategy**: Split into separate documents or clearly delimited sections for topology patterns, ingestion pipeline, bucket lifecycle, clustering, deployment mechanics, and performance tuning. If possible, create one chunk per major concept so retrieval does not mix SHC guidance with SmartStore or deployment server guidance.
- **Update Frequency**: Quarterly, and immediately after architecture standard changes or Splunk version upgrades.
- **Access/Governance Notes**: Store in an IT architecture or platform engineering SharePoint site. Grant read access to Splunk admins, SecOps engineering, SRE, and Copilot Studio makers. Restrict edit rights to the Splunk platform owner and designated documentation maintainers.

## `references/spl-guide.md`

- **Content Summary**: SPL fundamentals, search pipeline behavior, common commands, performance guidance, eval and rex usage, lookups, macros, event types, tags, workflow actions, field extraction patterns, dashboards, accelerated data models, and reusable analytic patterns.
- **Recommended Knowledge Type**: SharePoint document
- **Recommended Title**: Splunk SPL and Knowledge Objects Guide
- **Chunking Strategy**: Chunk by command family and task type: search pipeline basics, transforming commands, eval and rex, lookup design, knowledge objects, dashboards, and common detection or operations patterns. Keep command examples close to their explanation so retrieval returns both syntax and usage guidance.
- **Update Frequency**: Monthly for active environments, or whenever internal SPL standards, macro libraries, or dashboard conventions change.
- **Access/Governance Notes**: Publish broadly to SOC analysts, detection engineers, operations analysts, and Splunk developers. Keep edit access limited to the SPL standards owner or Splunk SME group.

## `references/development.md`

- **Content Summary**: Splunk app and TA structure, custom search commands, V1 versus V2 protocol guidance, modular inputs, secure credential handling, alert actions, REST API coverage, Python SDK usage, CIM normalization, and data model acceleration.
- **Recommended Knowledge Type**: SharePoint document
- **Recommended Title**: Splunk App Development and Integration Guide
- **Chunking Strategy**: Split by development surface: app structure, custom commands, modular inputs, alert actions, REST API and SDK, CIM normalization, and data models. For best retrieval, separate code-extension topics from admin topics so the agent does not confuse app packaging advice with search runtime advice.
- **Update Frequency**: Monthly in active development programs, and after every internal framework or SDK pattern change.
- **Access/Governance Notes**: Store in a development engineering SharePoint site. Grant read access to Splunk developers, integration teams, platform engineering, and approved Copilot makers. Restrict edit rights to app framework maintainers and lead developers.

## `references/administration.md`

- **Content Summary**: Configuration precedence, major `.conf` files, ingestion settings, parsing and timestamp controls, routing and masking, indexes.conf retention and storage, clustering and TLS settings, deployment server usage, licensing administration, and Monitoring Console focus areas.
- **Recommended Knowledge Type**: SharePoint document
- **Recommended Title**: Splunk Administration and Configuration Guide
- **Chunking Strategy**: Chunk by conf file and operating task. The best retrieval model is one chunk per configuration domain such as `inputs.conf`, `props.conf`, `transforms.conf`, `indexes.conf`, `server.conf`, deployment server, licensing, and Monitoring Console. This makes it easier for Copilot Studio to return the right file and stanza guidance.
- **Update Frequency**: Quarterly minimum, and immediately after production standards change for onboarding, retention, TLS, or deployment practices.
- **Access/Governance Notes**: Keep in a controlled operations SharePoint site. Read access should include Splunk admins, SRE, SOC engineering, and IT operations. Edit access should be limited to the Splunk service owner and change-authorized administrators.

## `references/security-es.md`

- **Content Summary**: RBAC, TLS, Enterprise Security operations, correlation searches, notable events, Risk-Based Alerting, threat intelligence, UEBA concepts, SOAR, ITSI, cloud-versus-enterprise boundaries, and a security hardening checklist.
- **Recommended Knowledge Type**: SharePoint document
- **Recommended Title**: Splunk Security, ES, and Governance Guide
- **Chunking Strategy**: Split by security capability: RBAC, TLS, ES detections, notable triage, RBA, threat intel, SOAR, ITSI, and hardening. Keep Splunk Cloud versus Enterprise restrictions in a dedicated chunk so the agent can retrieve platform-specific guardrails cleanly.
- **Update Frequency**: Monthly for security content, and immediately after changes to ES content standards, hardening baselines, or cloud governance requirements.
- **Access/Governance Notes**: Store in a restricted security SharePoint site or security knowledge library. Grant read access only to security engineering, SOC leads, approved Splunk admins, and Copilot makers with a documented business need. Treat edit rights as controlled security documentation ownership.

## Knowledge Library Structure

Recommended SharePoint folder layout:

```text
SharePoint Site: /sites/copilot-splunk-knowledge
  /01-architecture-core
    Splunk Architecture and Core Platform Guide.docx
    Splunk Deployment Patterns - Standalone vs Distributed.docx
    Splunk Clustering and SmartStore.docx
  /02-spl-search-knowledge-objects
    Splunk SPL and Knowledge Objects Guide.docx
    SPL Transforming Commands and Performance.docx
    Lookups, Macros, and Event Types.docx
  /03-development-integration
    Splunk App Development and Integration Guide.docx
    Custom Search Commands V2.docx
    Modular Inputs and Secure Credential Handling.docx
  /04-administration-configuration
    Splunk Administration and Configuration Guide.docx
    inputs.conf and Onboarding.docx
    props.conf and transforms.conf Parsing Guide.docx
    indexes.conf and Retention Guide.docx
  /05-security-es
    Splunk Security, ES, and Governance Guide.docx
    ES Correlation Searches and Notables.docx
    Risk-Based Alerting and Threat Intel.docx
  /99-glossary-and-synonyms
    Splunk Terminology and Acronyms.docx
```

## Retrieval Optimization Tips

1. **Convert reference files to `.docx` before uploading to SharePoint.** Markdown (`.md`) is not an officially supported Copilot Studio knowledge source file type. Use Pandoc (`pandoc input.md -o output.docx`) or a similar converter. Uploading raw `.md` may produce degraded chunking because markdown syntax characters pollute the extracted text. Supported types include `.docx`, `.pdf`, `.pptx`, and `.txt`.
2. Break large documents into smaller SharePoint files aligned to one task or one conf file at a time.
3. Use highly descriptive knowledge source names and descriptions in Copilot Studio; the description materially improves retrieval under generative orchestration.
4. Create separate documents for Splunk Enterprise and Splunk Cloud operational differences so the agent can ground to the correct platform-specific answer.
5. Keep examples near the explanation. A chunk that contains only syntax without context often retrieves poorly.
6. Add a glossary document for terms such as UF, HF, SHC, RF, SF, CIM, RBA, HEC, and notable event to improve matching on acronym-heavy user questions. Promote this from optional to a **Phase 2 required deliverable** — it has an outsized impact on retrieval accuracy.
7. Restrict access at the SharePoint library or site level for security-sensitive ES and hardening content so Copilot Studio respects Microsoft 365 access controls.
8. Re-test retrieval after every major knowledge refresh by running a fixed prompt set for SPL, admin, Cloud, and ES scenarios; tune titles, descriptions, and chunk size when the wrong document is cited.
9. Note the SharePoint document size limit of 50 MB. For large reference files, split into sub-documents before upload rather than relying on automatic chunking.
