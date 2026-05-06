# Tool and Action Design for a Copilot Studio Splunk Agent

## Phase 1: Knowledge Only

Start with a knowledge-only agent before adding any live Splunk connection.

### What the agent can do with knowledge only
- Draft and optimize SPL searches
- Explain Splunk Enterprise versus Splunk Cloud differences
- Review proposed `props.conf`, `transforms.conf`, `inputs.conf`, `indexes.conf`, `server.conf`, and `authorize.conf` changes
- Recommend deployment topologies for forwarders, indexers, search heads, SHC, and indexer clustering
- Explain Enterprise Security concepts such as correlation searches, notable events, RBA, threat intelligence, and CIM mapping
- Provide add-on, dashboard, modular input, alert action, and custom command design guidance
- Produce troubleshooting checklists and safe change plans

### Why this is the recommended starting point
A knowledge-only rollout is the lowest-risk way to migrate the Splunk skill into Copilot Studio. It requires no direct Splunk credential storage, no network path from Microsoft to Splunk, and no change-management approval for live administrative operations. It also gives the team time to validate instruction quality, knowledge retrieval quality, and enterprise governance before any tool can touch production data or configuration. For most organizations, this phase already delivers immediate value to analysts, admins, and engineers.

## Phase 2: Read-Only Tools

Add read-only REST actions only after the knowledge-only agent is performing reliably.

| Action | Purpose | Splunk REST API Endpoint | Risk Level | Authentication Model | Recommended for Enterprise Deployment |
|---|---|---|---|---|---|
| Get platform and version info | Confirm Splunk version, build, server role, and edition context before answering operational questions. | `GET /services/server/info` | Low | Service account with read-only REST permissions, or user-delegated auth | Yes |
| List indexes and retention settings | Show available indexes, storage paths, and retention-related metadata for architecture and admin troubleshooting. | `GET /services/data/indexes` | Low | Service account preferred | Yes |
| List data inputs | Inspect configured monitor, network, scripted, and HEC-related inputs without changing them. | `GET /services/data/inputs/all` | Medium | Service account with tightly scoped read permissions | Yes |
| List saved searches and correlation-search-style objects | Review schedules, actions, app context, and ownership for admin or ES troubleshooting. | `GET /servicesNS/-/-/saved/searches` | Medium | Service account or user-delegated auth | Yes |
| Execute constrained diagnostic search | Run a time-bounded SPL query to validate fields, counts, or configuration effects. | `POST /services/search/jobs/export` | Medium | User-delegated preferred for data minimization; service account only with strict guardrails | Yes, with allowlists and result-size limits |
| Read cluster status | Inspect indexer cluster or search head cluster health before giving change advice. | `GET /services/cluster/manager/info` and `GET /services/shcluster/captain/info` | Low | Service account limited to admin-read endpoints | Yes |
| List installed apps | Show which apps or TAs are present and their versions before recommending app-local changes. | `GET /services/apps/local` | Low | Service account preferred | Yes |
| Read health or license messages | Surface licensing warnings or health signals that affect troubleshooting and planning. | `GET /services/licenser/messages` or `GET /services/server/health` | Low | Service account preferred | Yes |

### Read-only design guardrails
- Limit live search actions to approved indexes, macros, or saved searches where possible.
- Enforce time-range caps and row-count caps.
- Suppress raw event payloads unless the user is explicitly authorized to see them.
- Separate read-only service accounts by environment: dev, test, and production.

## Phase 3: Controlled Admin Actions

Do not expose write actions until the organization has an approval workflow, audit logging, environment separation, and clear ownership.

| Action | Purpose | Splunk REST API Endpoint | Risk Level | Authentication Model | Recommended for Enterprise Deployment | Approval Workflow Recommendation | Audit Trail Requirements |
|---|---|---|---|---|---|---|---|
| Enable or disable a saved search or correlation search | Pause a noisy search, re-enable a validated one, or change scheduling state in a controlled way. | `POST /servicesNS/-/-/saved/searches/{name}` | High | Dedicated admin service account, scoped to specific apps and objects | Yes, but only with object allowlists | Require change request or dual approval in production; single approval in non-prod | Record who requested it, who approved it, object name, old value, new value, timestamp, and ticket ID |
| Update a lookup file | Refresh a controlled CSV lookup used for enrichment, suppression, or watchlists. | `POST /servicesNS/{owner}/{app}/data/lookup-table-files` | Medium | Service account scoped to one app namespace | Yes | Require content-owner approval and schema validation before upload | Store checksum, previous file version, requesting user, approving user, and rollback location |
| Update ES notable status or owner | Support triage workflows such as assigning or closing notable events with policy guardrails. | `POST /servicesNS/nobody/SplunkEnterpriseSecuritySuite/notable_update` | Medium | User-delegated auth strongly preferred for case accountability | Yes, for SOC workflows | No separate approval for routine triage; elevated approval for bulk actions | Log notable IDs, previous status, new status, owner changes, user identity, and reason code |
| Create a HEC token | Provision a new onboarding token for approved sources or teams. | `POST /servicesNS/nobody/system/data/inputs/http` | High | Dedicated provisioning service account, isolated from search access | Yes, only through a governed onboarding workflow | Require formal request, security review, and target-index approval | Log token name, indexes, source system, TTL or rotation policy, requester, approver, and vault reference |
| Create a new index | Provision a new index with approved retention and sizing settings. | `POST /services/data/indexes` | High | Dedicated infrastructure admin service account | Yes, but often better through an infrastructure workflow than direct agent execution | Require CAB or platform-owner approval in production | Capture full parameter set, retention values, storage class, approver, requester, and rollback or decommission plan |

### Controlled admin design notes
- Keep write tools disabled in the first production release.
- Expose write actions only to a separate administrative agent or topic set, not the general analyst-facing assistant.
- Add pre-execution confirmation text that summarizes the pending change in plain language.
- Prefer actions that target one known object instead of free-form administrative write access.

## Connector Options

### Power Automate custom connectors
This is the best primary option for enterprise deployment. Custom connectors give you managed definitions, environment-aware connections, Power Platform DLP controls, connection references, and strong administrative visibility. They fit well when you want a reusable Splunk REST surface for Copilot Studio tools, flows, and future Power Platform workloads.

### MCP
MCP is a strong architectural option when your organization already runs a governed MCP layer for multiple enterprise systems. It is useful if you want one normalized tool layer across Copilot Studio, developer assistants, and internal agents. However, it usually requires an extra translation or gateway component because Copilot Studio governance, authentication, and maker experience are still more mature with Power Platform connectors. Treat MCP as an advanced integration pattern, not the default first migration path.

### Direct HTTP connector or HTTP-based flow action
This is the fastest prototype path, but usually the weakest long-term enterprise design. It can work for a narrow proof of concept behind an internal proxy, especially for one or two low-risk GET operations. It is less attractive than a custom connector for production because schema management, discoverability, lifecycle control, and governance are weaker.

## Security & Governance

### Service account isolation
Use separate service principals or service accounts for read-only actions, controlled admin actions, and each environment. Do not let the same credential both search sensitive data and create infrastructure objects.

### Permission scoping
Scope Splunk permissions to the smallest possible set of indexes, namespaces, apps, and endpoints. If the tool runs searches, restrict search indexes and forbid unrestricted wildcard access. If the tool manages objects, scope it to specific apps or approved objects.

### Data exfiltration risks
The biggest risk is not configuration change; it is broad raw data retrieval. Limit result volume, mask sensitive fields, and block unrestricted searches over high-sensitivity indexes. Treat ad hoc search execution as a governed data-access capability.

### PII and regulated data handling
Assume Splunk may contain credentials, personal data, incident details, and regulated event content. Mask or suppress raw payloads by default. Return summaries, counts, or field-level diagnostics whenever possible. Align retention of agent transcripts and tool logs with privacy and security policy.

### Auditing and operational control
Log every tool invocation, the requesting identity, the target environment, the endpoint called, the parameters sent, and the response class. For write actions, store before-and-after values where feasible and link to a ticket, approval record, or incident number.
