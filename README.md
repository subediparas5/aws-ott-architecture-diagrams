# AWS OTT Architecture Diagram Suite

Use [index.html](./index.html) to open six focused interactive diagrams. The architecture starts with managed AWS primitives and records explicit triggers for adding heavier components.

01. [Executive Overview](./01-executive-overview.html) — Minimal control, media, telemetry, analytics, and evidence paths. [Markdown](./01-executive-overview.md) · [Draw.io](./drawio/01-executive-overview.drawio)
02. [Platform & Microservices](./02-platform-microservices.html) — Managed identity, private EKS APIs, transactional outbox, search, and secrets. [Markdown](./02-platform-microservices.md) · [Draw.io](./drawio/02-platform-microservices.drawio)
03. [VOD Processing Pipeline](./03-vod-processing.html) — Malware status, two QC gates, static packaging, DRM, delivery, and evidence. [Markdown](./03-vod-processing.md) · [Draw.io](./drawio/03-vod-processing.drawio)
04. [Live Streaming](./04-live-streaming.html) — Dual-feed encoding, MediaPackage v2, failover, harvest jobs, and evidence. [Markdown](./04-live-streaming.md) · [Draw.io](./drawio/04-live-streaming.drawio)
05. [Analytics, Recommendation & Ad-Tech](./05-analytics-recommendation-adtech.html) — Firehose lake analytics, Amazon Personalize, external ADS, and MediaTailor. [Markdown](./05-analytics-recommendation-adtech.md) · [Draw.io](./drawio/05-analytics-recommendation-adtech.drawio)
06. [Security, DevOps & Disaster Recovery](./06-security-devops-dr.html) — OIDC CI, production GitOps approval, tiered recovery, and separate evidence. [Markdown](./06-security-devops-dr.md) · [Draw.io](./drawio/06-security-devops-dr.drawio)
07. [Development and AWS Cost Estimate](./07-cost-estimate.md) — Scale scenarios for 0 to 100K viewers, VOD, live streaming, development, operations, and TCO assumptions.
08. [Nepal Hybrid / In-House Architecture and Cost Estimate](./08-nepal-hybrid-on-prem-cost-estimate.md) — AWS-vs-private-cloud TCO, Nepal CAPEX/OPEX, migration cost, CDN boundary, payback, and a four-page [Draw.io architecture](./drawio/07-nepal-hybrid-on-prem.drawio).
