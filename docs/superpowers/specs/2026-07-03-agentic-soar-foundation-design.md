# Agentic-SOAR Foundation Design

Date: 2026-07-03

## Purpose

This document defines the foundation for an evolutionary agentic-SOAR project. In this context, SOAR means Security Orchestration, Automation and Response. The system is intended to protect and emulate a minimal enterprise environment, ingest security events, create investigation cases, compare deterministic SOAR analysis against local LLM-based agentic analysis, and recommend response actions with full auditability.

The project will grow through three progressive demos:

1. Demo 1: minimal reproducible lab.
2. Demo 2: small realistic enterprise lab.
3. Demo 3: complete cyber range.

Demo 1 is the first implementation slice. It validates the core architecture without requiring heavy infrastructure, real production tools, or autonomous response execution.

Project execution is governed by `docs/PROJECT-MANAGEMENT.md`. That document defines the spec-driven phase workflow, ADR policy, evaluation discipline, and coding-agent policy for this repository.

## Product Definition

The agentic-SOAR is a security operations platform where deterministic playbooks and local LLM agents analyze the same security cases. The system ingests normalized events, correlates them into alerts and cases, investigates affected entities, produces evidence-backed conclusions, and recommends response actions.

For Demo 1, the system is recommend-only:

- It may investigate events and cases.
- It may generate hypotheses.
- It may compare deterministic and LLM-based analysis.
- It may recommend response actions.
- It must not execute infrastructure-changing response actions.

The first success target is not complete threat coverage. The first success target is demonstrating the full operational flow:

```text
event -> alert -> case -> investigation -> deterministic/LLM comparison -> recommendation -> audit -> SOC dashboard
```

## Progressive Demo Strategy

### Demo 1: Minimal Reproducible Lab

Demo 1 uses Docker Compose or an equivalent local container runtime. It combines synthetic events with lightweight real logs from simple lab services. It includes a minimal SOC dashboard, PostgreSQL storage, deterministic analysis, and local LLM analysis.

Primary goals:

- Validate ingestion and normalization.
- Validate case creation.
- Validate evidence collection.
- Compare deterministic SOAR output against local LLM SOAR output.
- Generate response recommendations without executing them.
- Display cases, evidence, recommendations, and audit trails in a minimal SOC dashboard.

### Demo 2: Small Realistic Enterprise Lab

Demo 2 increases realism while preserving the same core contracts.

Expected additions:

- More realistic identity service, such as LDAP, AD-compatible service, or IAM simulator.
- More endpoints and server roles.
- Optional lightweight SIEM or log search layer.
- More real logs and fewer synthetic-only scenarios.
- Simulated response execution.
- More connectors and playbooks.

### Demo 3: Complete Cyber Range

Demo 3 turns the system into a richer cyber range for SOC operations and adversary emulation.

Expected additions:

- Segmented enterprise-like network.
- Controlled adversary emulation with tools such as Atomic Red Team or MITRE Caldera.
- Higher-volume telemetry.
- Controlled real response actions inside the lab.
- Advanced evaluation across detection, triage, investigation, response, false positives, and operational safety.

## Demo 1 Architecture

Demo 1 has these logical modules:

```text
virtual enterprise lab
  -> data sources
  -> ingestion + normalization
  -> detection/triggers
  -> case manager
  -> deterministic-analysis-engine
  -> agentic-analysis-engine
  -> comparison-engine
  -> policy/recommendation engine
  -> audit log
  -> SOC dashboard
```

Module responsibilities:

- `data sources`: produce lightweight real logs and synthetic security events.
- `ingestion + normalization`: convert raw events into the canonical event schema.
- `detection/triggers`: transform event patterns into alerts.
- `case manager`: create and update cases, affected entities, evidence, timelines, and status.
- `deterministic-analysis-engine`: run rules, thresholds, fixed correlations, and baseline playbooks.
- `agentic-analysis-engine`: use a local LLM to analyze compact case packets.
- `comparison-engine`: compare deterministic and LLM results against each other and against scenario ground truth.
- `policy/recommendation engine`: produce response recommendations and enforce recommend-only behavior for Demo 1.
- `audit log`: record inputs, decisions, prompts, model outputs, recommendations, and state changes.
- `SOC dashboard`: display alerts, cases, evidence, analysis outputs, comparisons, recommendations, and audit records.

## Demo 1 Virtual Enterprise Network

Demo 1 represents a minimal enterprise network with enough components to produce identity, network, web, DNS, host, and SOAR telemetry.

```text
attacker
  -> router-firewall-sim
  -> dmz-web
  -> internal-linux-host
  -> identity-service-sim
  -> dns-sim
  -> log-collector / ingestion-api
  -> agentic-soar-core
  -> soc-dashboard
  -> storage/db
```

### Components

| Component | Purpose | Primary data produced |
|---|---|---|
| `attacker` | Runs controlled attack emulation scripts. | Synthetic attack activity, network activity. |
| `router-firewall-sim` | Represents perimeter and basic segmentation. | Network flow logs, allow/deny events. |
| `dmz-web` | Minimal web server or vulnerable app in a DMZ-like segment. | HTTP access logs, application errors. |
| `internal-linux-host` | Internal host with SSH and normal activity simulation. | SSH auth logs, system events. |
| `identity-service-sim` | Simulates users, groups, roles, and identity events. | Login events, failed auth, suspicious identity events. |
| `dns-sim` | Produces normal and suspicious DNS events. | DNS query logs, suspicious domain activity. |
| `log-collector / ingestion-api` | Receives raw logs and synthetic events. | Normalized canonical events. |
| `agentic-soar-core` | Manages cases, analysis, recommendations, and audit. | Cases, analysis results, recommendations, audit records. |
| `soc-dashboard` | Minimal analyst UI. | UI views over alerts, cases, evidence, and recommendations. |
| `storage/db` | PostgreSQL database for durable demo state. | Events, alerts, cases, recommendations, audit logs, fixtures. |

### Resource Sizing

The sizing below is for Demo 1 as a local development lab. It is not production sizing. Container limits should be configurable to support smaller or larger machines.

| Component | CPU recommended | RAM recommended | Disk recommended | Notes |
|---|---:|---:|---:|---|
| `attacker` | 0.25 vCPU | 256 MB | 1 GB | Runs controlled attack scripts. |
| `router-firewall-sim` | 0.25 vCPU | 256 MB | 1 GB | Produces flow logs and allow/deny events. |
| `dmz-web` | 0.5 vCPU | 512 MB | 2 GB | Minimal Nginx or vulnerable web app plus logs. |
| `internal-linux-host` | 0.5 vCPU | 512 MB | 2 GB | SSH, auth logs, and normal host activity. |
| `identity-service-sim` | 0.25 vCPU | 256 MB | 1 GB | Simulated users, groups, roles, and login events. |
| `dns-sim` | 0.25 vCPU | 256 MB | 1 GB | Normal and suspicious DNS event generation. |
| `log-collector / ingestion-api` | 0.75 vCPU | 1 GB | 5 GB | Normalization, buffering, replay, and event intake. |
| `agentic-soar-core` | 1 vCPU | 2 GB | 5 GB | Case manager, analysis engines, recommendations, audit. |
| `soc-dashboard` | 0.5 vCPU | 512 MB | 1 GB | Minimal SOC UI. |
| `storage/db` | 0.5 vCPU | 1 GB | 10 GB | PostgreSQL for events, cases, recommendations, audit, fixtures. |

Total recommended sizing:

| Resource | Minimum functional | Recommended |
|---|---:|---:|
| CPU | 2 vCPU | 4-6 vCPU |
| RAM | 4 GB | 8 GB |
| Disk | 15 GB | 30 GB |
| Network | Docker bridge local | Docker bridge with simulated segmentation |
| Runtime | Docker Compose | Docker Compose |

Demo 1 should run on a development laptop with approximately 8 GB of free system RAM. The local LLM may require fallback to CPU or a small quantized model on constrained GPUs.

## Initial Threat Scenarios

Demo 1 starts with four threat scenarios:

1. SSH brute force / credential attack.
2. Port scan / reconnaissance.
3. Web attack against `dmz-web`.
4. Suspicious outbound beacon / possible C2.

Each scenario must define:

- Low-level source events.
- Aggregated alert.
- Affected entities.
- Ground truth.
- MITRE ATT&CK mapping.
- Expected severity.
- Expected evidence.
- Expected deterministic result.
- Expected LLM analysis behavior.
- Expected recommendations.
- Evaluation criteria.

## Data Sources

Initial data sources:

```text
web_access_logs
auth_logs
identity_events
network_flow_logs
dns_logs
synthetic_alerts
soar_audit_logs
```

Demo 1 will use a hybrid data strategy:

- Lightweight real logs from lab services.
- Synthetic events generated by scenario scripts.
- Optional fixtures or importers from public cybersecurity datasets.

Public datasets are useful references and optional input sources, but Demo 1 must not depend on downloading large datasets to run.

Candidate public datasets:

- CIC-IDS2017.
- CSE-CIC-IDS2018.
- LANL Comprehensive Multi-Source Cyber-Security Events.
- OTRF Security Datasets / Mordor.
- Splunk BOTS.
- TON_IoT for later heterogeneous telemetry scenarios.

## Canonical Event Schema

The project will use a minimal internal canonical schema inspired by ECS and OCSF. It must be simple enough for Demo 1 while remaining mappable to standard schemas later.

Required base fields:

```text
event_id
timestamp
source_type
event_type
severity
asset
user
src_ip
dst_ip
src_port
dst_port
protocol
process
url
domain
labels
attack.technique_id
attack.tactic
raw_ref
```

Schema principles:

- Keep the internal schema stable across demos.
- Preserve raw references for audit and troubleshooting.
- Allow nullable fields when a source does not provide a value.
- Prefer structured fields over free-form strings.
- Include mapping metadata for ECS, OCSF, and MITRE ATT&CK where useful.

## Case Model

A case represents the investigation unit used by both the deterministic engine and the agentic LLM engine.

Required case data:

```text
case_id
title
status
severity
created_at
updated_at
source_alerts
affected_entities
timeline
evidence
ground_truth
deterministic_analysis
agentic_analysis
comparison_result
recommendations
audit_refs
```

Case status values for Demo 1:

```text
new
investigating
analysis_complete
recommendation_ready
closed
```

## Deterministic SOAR Baseline

The deterministic engine provides a transparent baseline for comparison. It uses rules, thresholds, fixed correlations, and playbook-style logic.

Examples:

- SSH brute force if many failed logins from one source IP within a time window.
- Higher severity if failed logins are followed by a successful login.
- Port scan if one source touches many ports or hosts within a time window.
- Web attack if HTTP paths, query parameters, status codes, or payload indicators match known suspicious patterns.
- Possible C2 if DNS or outbound network events show periodic suspicious beacon-like behavior.

Output fields:

```text
engine
rule_ids
severity
confidence
summary
evidence_refs
recommended_actions
limitations
```

## Agentic LLM SOAR

The agentic engine uses a local LLM from the beginning of the project. The LLM is not a fixed model. It is an interchangeable local inference layer.

```text
agentic-analysis-engine
  -> llm-provider-interface
      -> local-runtime-adapter
          -> ollama for Demo 1
          -> llama.cpp as future option
          -> vLLM as future server-GPU option
```

The first implementation should recommend Ollama as the Demo 1 runtime because it simplifies local setup and model switching. The architecture must still keep the runtime behind an interface so the project can later move to llama.cpp, vLLM, or another local runtime.

### Hardware Profiles

The initial local development profile targets a machine with an NVIDIA GeForce RTX 3050 with 4 GB VRAM.

```yaml
llm_profiles:
  local_dev_4gb_vram:
    runtime: ollama
    model_class: small_instruction_model_quantized
    quantization: q4_or_equivalent
    context_strategy: compact_case_packet
    fallback: cpu_allowed
```

Future server profile:

```yaml
llm_profiles:
  gpu_server:
    runtime_candidates:
      - ollama
      - llama.cpp
      - vllm
    model_class: larger_instruction_model
    quantization: configurable
    context_strategy: larger_case_packet
    fallback: configurable
```

### Case Packet Strategy

The LLM must not receive all raw logs. It receives a compact case packet prepared by the system.

Required case packet fields:

```text
case summary
alert
entities
reduced timeline
top evidence
deterministic baseline result
policy constraints
allowed recommendation types
```

This keeps analysis feasible on constrained hardware and makes LLM behavior easier to audit.

### LLM Audit Metadata

Each LLM analysis must record:

- Runtime used.
- Model name.
- Model parameters.
- Prompt template version.
- Case packet version.
- Input evidence references.
- Structured model output.
- Errors or timeouts.
- Latency.
- Token counts when available.

## Recommendation Model

Demo 1 recommendations are proposed actions only. They are never executed.

Required fields:

```text
recommendation_id
case_id
action_type
target
risk
required_approval
rationale
expected_effect
rollback_hint
source_engine
confidence
status
execution_allowed
created_at
```

For Demo 1:

```text
status = proposed
execution_allowed = false
```

Example recommendation types:

- Block source IP.
- Review affected account.
- Force credential reset.
- Monitor host for lateral movement.
- Inspect endpoint.
- Block domain.
- Add temporary detection.
- Escalate to incident.

## Policy Model

The policy layer defines what the SOAR is allowed to do in each demo.

Demo 1 policy:

- Investigation is allowed.
- Case creation is allowed.
- Recommendation generation is allowed.
- Infrastructure-changing actions are forbidden.
- Recommendations must explicitly mark that execution is not allowed.

Future demo policy progression:

- Demo 2 may allow simulated execution.
- Demo 3 may allow controlled real execution inside the lab with approval and rollback metadata.

## SOC Dashboard

The Demo 1 UI is a minimal SOC dashboard. It is not a full SIEM or ticketing system.

Required views:

### Alerts

- Alert list.
- Severity.
- Source.
- Scenario.
- Status.
- Created time.

### Cases

- Case list.
- Status.
- Severity.
- Affected entities.
- Scenario.
- Analysis state.

### Case Detail

- Case summary.
- Timeline.
- Relevant events.
- Evidence.
- Deterministic analysis output.
- Agentic LLM analysis output.
- Side-by-side comparison.
- Recommendations.
- Audit trail.
- Ground truth for emulated scenarios.

The UI should make it easy to answer:

- What happened?
- What assets, users, IPs, or domains are involved?
- What evidence supports the conclusion?
- What did the deterministic baseline conclude?
- What did the LLM conclude?
- Where do they agree or disagree?
- What response actions are recommended?
- Why are those actions recommended?

## Evaluation

Demo 1 includes evaluation from the beginning. Each scenario should have expected outcomes and ground truth.

Evaluation dimensions:

- Correct scenario detection.
- Correct or acceptable severity.
- Correct affected entities.
- Evidence cited correctly.
- Recommendations aligned with the scenario.
- Deterministic output quality.
- LLM output quality.
- Agreement and disagreement between engines.
- False positives on normal activity.
- Analysis latency.
- LLM runtime errors or timeouts.
- Audit completeness.

The comparison engine should help identify:

- When the LLM improves on deterministic analysis.
- When deterministic rules are sufficient.
- When the LLM overestimates severity.
- When the LLM omits evidence.
- When the LLM recommends an unsafe or unsupported action.

## Non-Goals For Demo 1

Demo 1 does not attempt to provide:

- Production-grade SIEM replacement.
- Real autonomous containment.
- Full enterprise identity infrastructure.
- Full endpoint detection and response.
- High-volume telemetry processing.
- Complete MITRE ATT&CK coverage.
- Multi-tenant production security controls.

These are intentionally deferred to later demos.

## Acceptance Criteria For Demo 1

Demo 1 is successful when:

1. The lab can be started locally with documented commands.
2. The four initial threat scenarios can generate events.
3. Events are normalized into the canonical schema.
4. Triggers create alerts from normalized events.
5. Alerts create or update cases.
6. Each case has entities, timeline, evidence, and ground truth.
7. The deterministic engine produces a baseline analysis.
8. The local LLM engine produces an agentic analysis using a compact case packet.
9. The comparison engine shows agreement and disagreement between both engines.
10. Recommendations are generated in recommend-only mode.
11. No infrastructure-changing action can execute in Demo 1.
12. PostgreSQL stores events, alerts, cases, recommendations, and audit records.
13. The SOC dashboard shows alerts, cases, case details, evidence, analysis outputs, comparisons, recommendations, and audit trail.
14. Resource usage fits the Demo 1 recommended profile or degrades gracefully on smaller machines.

## Open Implementation Notes

These are design constraints for the later implementation plan:

- Keep ingestion, case management, analysis engines, recommendation policy, and UI boundaries explicit.
- Do not couple the SOAR core to one LLM model.
- Do not couple the SOAR core to one local LLM runtime, even if Ollama is the first recommended runtime.
- Keep scenario fixtures small enough for repository-friendly development.
- Make larger public datasets optional and importable, not required for Demo 1 startup.
- Store raw event references so normalized events remain auditable.
- Preserve all deterministic and LLM outputs for comparison and reproducibility.
