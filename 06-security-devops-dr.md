# Security, DevOps and Disaster Recovery

Explore deployment, security detection, regional recovery, and evidence-backed incident response as separate operational scenarios.

[Open the interactive diagram](./06-security-devops-dr.html) · [Back to suite](./index.html)

## Components

| Component | Technology | Scope |
|---|---|---|
| Developer | Git repository | signed change |
| CodePipeline | Release orchestration | approved stages |
| CodeBuild | Test + scan + build | ephemeral |
| Amazon ECR | Signed container images | immutable tag |
| Primary EKS | Production workloads | multi-AZ |
| Observability | CloudWatch + tracing | alerts |
| Detection | GuardDuty + Security Hub | findings |
| Audit trail | CloudTrail + Config | organization logs |
| Recovery Region | Standby application | warm standby |
| Replicated data | S3 + Aurora secondary | cross-region |
| Evidence vault | Security Lake + Object Lock | immutable |
| Investigation | Detective + Athena | case workflow |

## 1. Deploy an application release

- **What:** Build and progressively deploy an approved application release.
- **Why:** Production changes need repeatability, security checks, health gates, and rollback capability.
- **How:** CodePipeline orchestrates CodeBuild checks, publishes an immutable ECR digest, deploys to EKS, and verifies service-level signals.

1. **Submit an approved change:** A reviewed and signed change starts the release pipeline.
2. **Build and verify:** The build runs tests, dependency checks, image scanning, and policy validation.
3. **Publish immutable image:** The pipeline pushes the verified image under an immutable digest.
4. **Deploy progressively:** The release deploys using health gates and rollback criteria.
5. **Verify service health:** Operations confirms latency, error rate, saturation, and business signals.

## 2. Detect and triage a threat

- **What:** Detect, enrich, and triage a security finding.
- **Why:** A raw alert lacks the audit and configuration context needed to make a reliable response decision.
- **How:** GuardDuty and Security Hub findings are enriched with CloudTrail and Config records, preserved, and opened as a verified investigation case.

1. **Emit security signals:** Workload, identity, and network events produce security findings.
2. **Enrich with audit history:** The finding is correlated with management actions and configuration state.
3. **Preserve relevant records:** Relevant logs and configurations are copied to retention-locked evidence storage.
4. **Open verified case:** Investigators receive verified working copies and a custody record.

## 3. Fail over to the recovery Region

- **What:** Restore service in a secondary AWS Region after primary-region impairment.
- **Why:** Regional failure must stay within the platform’s agreed recovery time and data-loss objectives.
- **How:** Monitoring triggers the runbook, replicated data is promoted, the standby EKS stack is validated, and Route 53 shifts traffic.

1. **Detect regional impairment:** Health checks establish that the primary service cannot meet its objective.
2. **Activate replicated state:** The recovery environment promotes replicated data under controlled automation.
3. **Validate recovery workload:** Synthetic transactions and operational checks confirm service readiness.
4. **Shift production traffic:** Traffic moves only after the recovery stack passes validation.
5. **Record recovery actions:** Every promotion and routing change enters the organization audit trail.

## 4. Investigate and contain an incident

- **What:** Collect evidence, reconstruct an incident, and apply authorized containment.
- **Why:** Response actions must be defensible, reproducible, and linked to unmodified source evidence.
- **How:** Security Lake and the evidence vault preserve records; Detective and Athena correlate them; responders revoke or isolate affected resources.

1. **Collect the finding set:** Findings, flow logs, audit records, workload logs, and relevant snapshots are preserved.
2. **Verify audit evidence:** Evidence acquisition records include origin, collector, timestamp, and digest.
3. **Reconstruct the timeline:** The investigator correlates principal, resource, request, IP, and time.
4. **Contain the affected workload:** Approved response revokes credentials, isolates workloads, or updates controls.
5. **Preserve response history:** Containment actions and approvals become part of the case record.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
