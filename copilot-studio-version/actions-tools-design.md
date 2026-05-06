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
| Get platform and version info | Confirm Splunk version, build, server role, and edition context before answering operational questions. | `GET /services/server/info` | Low | Splunk authentication token (generated via `/services/auth/tokens`) or basic auth. **Note:** Splunk's REST API does not natively support OAuth 2.0. OAuth 2.0 requires an API gateway or proxy layer. | Yes |
| List indexes and retention settings | Show available indexes, storage paths, and retention-related metadata for architecture and admin troubleshooting. | `GET /services/data/indexes` | Low | Service account with read-only REST permissions; Splunk auth token preferred | Yes |
| List data inputs | Inspect configured monitor, network, scripted, and HEC-related inputs without changing them. | `GET /services/data/inputs/all` | Medium | Service account with tightly scoped read permissions | Yes |
| List saved searches and correlation-search-style objects | Review schedules, actions, app context, and ownership for admin or ES troubleshooting. | `GET /servicesNS/-/-/saved/searches` | Medium | Service account or user-delegated Splunk auth token | Yes |
| Execute constrained diagnostic search | Run a time-bounded SPL query to validate fields, counts, or configuration effects. Use the two-step create-then-poll pattern (`POST /services/search/jobs` + `GET /services/search/jobs/{sid}/results`) for searches that run longer than a few seconds. Use `POST /services/search/jobs/export` only for fast, simple searches — it is a streaming endpoint and will stall or time out on longer queries. | `POST /services/search/jobs` + `GET /services/search/jobs/{sid}/results` | Medium | User-delegated Splunk auth token preferred; service account only with strict allowlists and result-size limits | Yes |
| Read cluster status | Inspect indexer cluster or search head cluster health before giving change advice. **Version note:** `/services/cluster/manager/info` was renamed from `/services/cluster/master/info` in Splunk 9.0. On Splunk 8.x, use the `/master/` path. Always retrieve Splunk version first via `GET /services/server/info`. | `GET /services/cluster/manager/info` (Splunk 9.x+) or `GET /services/cluster/master/info` (Splunk 8.x) | Low | Service account limited to admin-read endpoints | Yes |
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
| Create a HEC token | Provision a new onboarding token for approved sources or teams. | `POST /servicesNS/{user}/{app}/data/inputs/http` (use the correct owner and app namespace for your environment; the `nobody/system` namespace may return 404 on some versions) | High | Dedicated provisioning service account, isolated from search access | Yes, only through a governed onboarding workflow | Require formal request, security review, and target-index approval | Log token name, indexes, source system, TTL or rotation policy, requester, approver, and vault reference |
| Create a new index | Provision a new index with approved retention and sizing settings. | `POST /services/data/indexes` | High | Dedicated infrastructure admin service account | Yes, but often better through an infrastructure workflow than direct agent execution | Require CAB or platform-owner approval in production | Capture full parameter set, retention values, storage class, approver, requester, and rollback or decommission plan |

### Controlled admin design notes
- Keep write tools disabled in the first production release.
- Expose write actions only to a separate administrative agent or topic set, not the general analyst-facing assistant.
- Add pre-execution confirmation text that summarizes the pending change in plain language.
- Prefer actions that target one known object instead of free-form administrative write access.

## Network Connectivity Prerequisites

**This is the most common blocker for Phase 2 adoption.** The Splunk REST API runs on port 8089. In most enterprise deployments, Splunk is on-premises or in a private cloud segment. Power Automate cloud connectors run in Azure. Without explicit network path planning, the connector cannot reach Splunk.

Resolve this before committing to Phase 2:

| Scenario | Required Component |
|---|---|
| On-premises Splunk, Power Automate cloud flows | On-premises data gateway installed on a server with network access to Splunk port 8089 |
| On-premises Splunk, Azure-hosted flows | VPN, ExpressRoute, or API gateway proxy that exposes port 8089 to Azure |
| Splunk Cloud (Victoria or Classic) | Splunk Cloud allows REST access on port 8089; verify allowlisted IPs or firewall rules for Azure egress IPs |
| Dev/test sandbox | Direct connectivity or SSH tunnel is acceptable; do not use in production |

A network and security review for a new connector path typically takes **2–8 weeks** in a governed enterprise. Plan this in Phase 0, not Phase 2.

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
