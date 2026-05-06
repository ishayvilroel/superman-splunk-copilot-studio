# Suggested Starter Prompts for the Copilot Studio Splunk Agent

## SPL Queries

### 1. Blocked outbound email DLP pattern
- **Exact End-User Prompt**: `Write an SPL search that finds users with more than 5 blocked outbound email DLP incidents in the last 24 hours, grouped by user and policy, and exclude service accounts.`
- **Admin Note**: The agent should return production-quality SPL with `index=` scoping, filtering, aggregation, and a brief explanation of tuning assumptions.

### 2. Top talkers and traffic shaping view
- **Exact End-User Prompt**: `Help me build an SPL query for firewall logs that shows top talkers by source IP and destination port over the last 7 days, with a timechart I can use in a dashboard.`
- **Admin Note**: The agent should provide a complete SPL pipeline, explain field assumptions, and suggest dashboard-friendly output structure.

### 3. Authentication failure followed by success
- **Exact End-User Prompt**: `Create an SPL detection for users who have multiple authentication failures followed by a success from the same source IP within 15 minutes.`
- **Admin Note**: The agent should produce a realistic detection query, explain why it chose `stats` or another approach, and call out any required field normalization.

## Architecture & Infrastructure

### 4. Move from single instance to distributed Splunk
- **Exact End-User Prompt**: `We're moving from a single Splunk Enterprise instance to a distributed deployment for about 500 GB/day. Design the target topology and explain what should run on each tier.`
- **Admin Note**: The agent should ask follow-up questions about HA and growth, then recommend an architecture with forwarders, indexers, search heads, and management tiers.

### 5. Clustering and SmartStore decision support
- **Exact End-User Prompt**: `Do we need indexer clustering, search head clustering, or SmartStore for 2 TB/day ingest and 90-day searchable retention? Please compare the tradeoffs.`
- **Admin Note**: The agent should compare availability, storage, performance, and operational complexity with a clear recommendation.

## Administration & Configuration

### 6. Parsing and routing review
- **Exact End-User Prompt**: `Review my approach for parsing multiline Java logs and tell me what should be configured in props.conf and transforms.conf, and whether it belongs on the heavy forwarder or indexer.`
- **Admin Note**: The agent should give tier-aware guidance and explicitly include conf file, stanza, attribute, scope, and restart or reload requirements.

### 7. Search performance tuning
- **Exact End-User Prompt**: `My search head gets slow during the morning scheduled search window. What limits.conf and search tuning areas should I evaluate first?`
- **Admin Note**: The agent should provide an investigation checklist, likely bottlenecks, safe tuning order, and warnings about cluster-wide impact.

## Enterprise Security & ES

### 8. Risk-Based Alerting design
- **Exact End-User Prompt**: `Design a Risk-Based Alerting approach in Splunk ES for impossible travel, disabled account use, and malware detections that all contribute to a single user risk score.`
- **Admin Note**: The agent should map detections to risk events, score strategy, and explain how to operationalize the logic in ES.

### 9. Repeating notable event investigation
- **Exact End-User Prompt**: `A notable event in Splunk ES keeps firing repeatedly. Help me troubleshoot the correlation search, suppression logic, and notable event workflow.`
- **Admin Note**: The agent should walk through likely root causes, relevant ES objects, and where to inspect SPL, schedule, suppression, and adaptive response settings.

### 10. Threat intelligence onboarding
- **Exact End-User Prompt**: `Create a checklist for onboarding malicious IP and domain feeds into Splunk ES with proper field mapping, validation, and operational safeguards.`
- **Admin Note**: The agent should produce a step-by-step implementation checklist that covers IOC types, feed hygiene, mapping, and validation.

## Development

### 11. Technical add-on design
- **Exact End-User Prompt**: `I need a Splunk TA design for ingesting a SaaS audit API every 5 minutes, storing credentials securely, and normalizing the data to CIM.`
- **Admin Note**: The agent should recommend TA structure, modular input design, credential handling, and CIM field mapping strategy.

### 12. Python custom search command
- **Exact End-User Prompt**: `Show me how to build a Splunk custom search command in Python using the v2 chunked protocol to enrich events from an internal REST API.`
- **Admin Note**: The agent should explain the command pattern, where the files live, what belongs in `commands.conf`, and how to keep the design search-head safe.
