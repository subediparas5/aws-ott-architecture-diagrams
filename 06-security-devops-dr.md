# Security, DevOps and Disaster Recovery

Explore production promotion, security detection, objective-driven regional recovery, and case-specific digital evidence.

[Open the interactive diagram](./06-security-devops-dr.html) · [Download Draw.io](./drawio/06-security-devops-dr.drawio) · [Back to suite](./index.html)

## Architecture decisions

- GitHub Actions uses repository- and branch-restricted OIDC roles; no long-lived AWS keys are stored in GitHub.
- Development may auto-update its GitOps branch. Production promotion is a reviewed GitOps pull request using an immutable ECR digest.
- Argo CD is outside the primary workload failure domain through a management plane or independent per-cluster instances.
- Security Lake is the normalized telemetry store; the Object Lock vault contains only selected case evidence.
- Recovery strategy is chosen per workload from RTO and RPO; warm standby is not the default for every component.

## Add only when

- Move from backup/restore to pilot light or warm standby only when tested recovery cannot meet the approved RTO.
- Add a second-region live media pipeline only for events with a matching availability objective and budget.
- Use centralized hub-and-spoke Argo CD when the number of clusters and teams justifies a dedicated management cluster.

## Components

| Component | Technology | Scope |
|---|---|---|
| Developer | Local Git checkout | reviewed commit |
| GitHub | Application repository | push event |
| GitHub Actions | OIDC · build · test | CI workflow |
| Amazon ECR | Scan · sign · immutable digest | release artifact |
| GitOps repository | Environment desired state | production PR |
| Argo CD | Management or per-cluster | reconciliation |
| Primary EKS | Production workloads | multi-AZ |
| Health + traffic | CloudWatch · Route 53 | SLO + routing |
| Detection | GuardDuty · Security Hub | findings |
| Security telemetry | CloudTrail · Config · Security Lake | OCSF + audit |
| Recovery prerequisites | Aurora · S3 · ECR · secrets/KMS | cross-region + backups |
| Recovery EKS | IaC · pilot light / warm | tiered by RTO |
| Evidence vault | S3 Object Lock | case-specific originals |
| Investigation | Detective · Athena · case system | working copies |

## 1. Deploy an application release

- **What:** Build an application change and deploy an approved immutable image to EKS.
- **Why:** CI should create artifacts while Git remains the auditable desired state for deployment.
- **How:** GitHub Actions uses OIDC to build, test, scan, sign, and publish an ECR digest; it opens a production GitOps PR; Argo CD reconciles the reviewed merge.

1. **Push the reviewed application change:** The developer pushes a reviewed commit to the application repository.
2. **Trigger the CI workflow:** GitHub starts build, test, dependency, and security checks.
3. **Publish a signed immutable image:** The workflow assumes a short-lived, repository-restricted IAM role, scans the image, signs it, and pushes its digest.
4. **Open the production promotion PR:** CI updates only the image digest. Development may auto-merge its branch; production requires review and protected-branch approval.
5. **Merge approved desired state:** After approval and merge, Argo CD detects the new Git revision and compares it with the target cluster.
6. **Reconcile the EKS workload:** Argo applies the approved manifests and verifies Kubernetes health policy.
7. **Verify release health:** CloudWatch, synthetic checks, and Argo health confirm the release or trigger rollback.

## 2. Detect and triage a threat

- **What:** Detect, enrich, and triage a security finding.
- **Why:** A raw alert lacks the identity, configuration, and audit context needed for response.
- **How:** GuardDuty and Security Hub produce findings; CloudTrail and Config enter Security Lake; investigators query working data and preserve only selected case material in Object Lock.

1. **Emit a security finding:** Workload, identity, and network signals produce a normalized finding.
2. **Correlate security telemetry:** Findings join organization CloudTrail, Config, and supported security sources in Security Lake.
3. **Open a triage case:** Investigators query normalized telemetry and related entities without changing source records.
4. **Preserve selected evidence:** Only records material to the case are copied or referenced with source, version, timestamp, and digest.
5. **Create a verified working set:** The investigator receives hash-verified copies while originals remain retention locked.

## 3. Recover the critical service tier

- **What:** Restore the critical service tier after a primary-region impairment.
- **Why:** Regional recovery must include every required dependency and avoid relying on an unavailable primary control plane.
- **How:** Health checks trigger the runbook; replicated state and artifacts activate in the recovery Region; independent Argo reconciles EKS; synthetic checks pass before Route 53 shifts traffic.

1. **Declare regional impairment:** SLO breaches and independent health checks satisfy the approved disaster declaration criteria.
2. **Activate recovery prerequisites:** The runbook promotes or restores Aurora, S3, ECR, secrets, KMS access, and other tier dependencies within the accepted RPO.
3. **Reconcile and scale recovery EKS:** Argo CD outside the failed cluster applies the same approved Git revision and scales the recovery workload.
4. **Run recovery validation:** Synthetic account, catalog, playback, and dependency checks confirm the stack can serve traffic.
5. **Shift production traffic:** Traffic moves only after recovery validation succeeds and split-brain controls are active.
6. **Record recovery and failback state:** Promotions, routing changes, approvals, and eventual failback enter the audit trail.

## 4. Investigate and contain an incident

- **What:** Collect selected evidence, reconstruct the event, and apply authorized containment.
- **Why:** Response actions must be reproducible and linked to unmodified source evidence.
- **How:** Security Lake supplies normalized telemetry, the evidence vault protects case originals, investigators work from verified copies, and containment returns to the audit trail.

1. **Collect the normalized finding set:** Findings, audit events, network records, and workload signals are correlated by principal, resource, request, IP, and time.
2. **Preserve the material case set:** The collector records origin, object version, timestamp, identity, and digest for each selected item.
3. **Reconstruct from working copies:** The investigator uses verified copies and records queries, exports, conclusions, and uncertainty.
4. **Apply authorized containment:** Approved response revokes credentials, isolates workloads, or changes policy while preserving service safety.
5. **Preserve response history:** Containment actions, approvals, outcomes, and rollback state become part of the audit and case history.

## Recovery inventory

A regional recovery test covers Aurora promotion or restore, S3 replication, ECR replication, secrets and KMS availability, GitOps and infrastructure state, OpenSearch rebuild or restore, event backlog handling, external DRM and ad dependencies, synthetic API and playback tests, DNS routing, split-brain prevention, and controlled failback. Backup recovery is still tested even when a warm environment exists.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
