# AWS OTT development and run-cost estimate

**Estimate date:** 22 July 2026  
**Currency/region:** USD, US East (N. Virginia), on-demand list pricing  
**Accuracy:** planning estimate, normally ±30–50% until traffic geography, codecs, catalogue, RTO/RPO, DRM and support contracts are fixed

This estimate applies the linked cost-estimator framework to the architecture graph: managed identity, a small modular EKS application, Aurora as authority, rebuildable OpenSearch, static VOD packages in S3, managed live origin, serverless analytics, GitOps, security telemetry, an immutable evidence vault and tiered disaster recovery. The graph specifically confirms that MediaConnect is optional, VOD avoids just-in-time packaging, and regional live DR is a separate commercial decision; those choices prevent three large speculative costs.

## Executive answer

- A full production implementation is approximately **$4,251,000** of one-time development: 21,800 base hours at a $125/hour blended loaded rate, 1.30 integration complexity and 1.20 risk buffer.
- A useful 1K-viewer production launch is approximately **$2.76M** cumulative development; 100K readiness is the full **$4.25M** scope.
- The production-ready platform before media traffic is approximately **$1,197/month** at zero viewers. It is not zero because EKS control planes, minimum workers, database/search, network, logging, backup and recovery remain provisioned.
- Under the standard combined workload, estimated AWS spend ranges from **$2.2K/month idle** to **$204K/month at 100K viewers**. CloudFront transfer is the dominant scale cost.
- If nothing is deployed, zero viewers costs $0. A lean single-account demo can be held around $200–$500/month by removing HA, recovery and always-on OpenSearch; that is not the drawn production architecture.

## Definitions and assumptions

Viewer count means **monthly VOD viewers** for VOD tables and **average concurrent viewers while the channel is running** for live tables. The standard combined case uses the same count for both: each VOD viewer watches 10 hours/month, and the live service runs 20 hours/month at that average concurrency. This is deliberately explicit because viewer count alone cannot price video.

Core assumptions:

- 730 hours/month; three-AZ primary region; pilot-light recovery region.
- CloudFront pay-as-you-go, US/Canada transfer tiers, first 1 TB and first 10M HTTPS requests free; approximately 650 HLS/DASH requests per viewer-hour.
- Static VOD HLS/DASH packages in S3; no VOD MediaPackage charge.
- Live MediaPackage has a 99% CDN cache-hit ratio and origin packaging is $0.05/GB; live HD fixed encoding plus ingest is $4.03/channel-hour.
- MediaTailor model: VOD one ad per five minutes at $0.25/1,000 insertions; live two breaks/hour and three ads/break at $0.50/1,000 insertions.
- Medium content programme: 1,000 catalog hours, 20 GB/source-hour archived, 6 GB/output-hour in S3, one recovery copy, and 100 new source hours transcoded monthly.
- Viewer data formula: `GB = viewers × hours × Mbps × 3,600 ÷ 8 ÷ 1,024`.

## Development cost

Formula from the requested skill: `base hours × blended loaded rate × complexity factor × risk buffer`.

| Phase | Base hours | Estimated cost |
|---|---|---|
| Discovery, requirements, threat model, cost model | 1,200 | $234,000 |
| AWS foundation, IaC, EKS, GitOps, IAM, observability | 2,500 | $487,500 |
| Identity, accounts, catalog, rights, search, playback API | 3,000 | $585,000 |
| VOD ingest, malware scan, ABR, two QC gates, DRM | 2,400 | $468,000 |
| Live contribution, MediaLive, MediaPackage, SSAI, failover | 2,600 | $507,000 |
| QoE analytics, lake, dashboards, recommendations, ad-tech | 2,400 | $468,000 |
| Web, iOS/Android and TV reference clients | 4,500 | $877,500 |
| Automation, performance, DR, security validation, launch | 3,200 | $624,000 |
| **Total** | **21,800** | **$4,251,000** |

The 21,800 hours are about 136 person-months: roughly a 10–12 person cross-functional team for 12–15 months. The client estimate includes reference web, mobile and TV apps, not every OEM-specific certification programme.

### Cumulative development by target scale

Development is not proportional to traffic. These are cumulative product-maturity gates, not recurring charges.

| Target viewers | Delivery maturity | Cumulative development | First-year AWS | Dev + AWS year 1 |
|---|---|---|---|---|
| 0 | Architecture/prototype | $850,200 | $26,611 | $876,811 |
| 1 | Functional alpha | $1,487,850 | $26,612 | $1,514,462 |
| 100 | Closed beta | $2,125,500 | $33,917 | $2,159,417 |
| 1K | Production launch | $2,763,150 | $98,253 | $2,861,403 |
| 10K | Scale hardened | $3,485,820 | $439,563 | $3,925,383 |
| 100K | Enterprise/high-scale | $4,251,000 | $2,451,197 | $6,702,197 |

## Always-on AWS platform cost

These are capacity envelopes for the control plane and supporting services. They include the services that remain even when CDN/media traffic is zero.

| Monthly component | 0 | 1 | 100 | 1K | 10K | 100K |
|---|---|---|---|---|---|---|
| EKS control planes (primary + recovery) | $146 | $146 | $146 | $146 | $146 | $146 |
| EKS worker compute + EBS | $203 | $203 | $203 | $405 | $1,216 | $5,405 |
| ALB, NAT gateways, endpoints, cross-AZ | $140 | $140 | $140 | $220 | $500 | $1,800 |
| Aurora capacity, storage, I/O | $108 | $108 | $180 | $360 | $1,500 | $6,000 |
| OpenSearch search projection | $100 | $100 | $100 | $250 | $900 | $4,500 |
| Observability, analytics, security, evidence | $250 | $250 | $300 | $600 | $1,800 | $8,000 |
| Backups and recovery copies | $150 | $150 | $200 | $400 | $1,200 | $6,000 |
| API, identity, queues, events, secrets, DNS | $100 | $100 | $120 | $200 | $500 | $2,000 |
| **Platform subtotal** | **$1,197** | **$1,197** | **$1,389** | **$2,581** | **$7,762** | **$33,851** |

Sizing logic: the low tier keeps two EKS control planes and three small Graviton workers; higher tiers add workers, Aurora capacity, OpenSearch nodes, telemetry ingestion and recovery capacity. EKS itself is $0.10/cluster-hour under standard support. Three NAT gateways alone are about $99/month before data, so VPC endpoints are assumed for S3/ECR-heavy paths.

## VOD delivery scenarios

| Monthly VOD scenario | 0 | 1 | 100 | 1K | 10K | 100K |
|---|---|---|---|---|---|---|
| Light: 2 h/viewer, 3 Mbps | $0 | $0 | $1 | $143 | $2,137 | $15,327 |
| Standard: 10 h/viewer, 5 Mbps | $0 | $0 | $103 | $1,752 | $13,204 | $72,995 |
| Heavy: 30 h/viewer, 8 Mbps | $0 | $0 | $817 | $7,416 | $41,875 | $262,515 |

These VOD figures are **variable delivery, requests and MediaTailor ad insertion only**. Add the platform subtotal and content storage/transcoding. Approximate delivered size per viewer-hour is 1.32 GB at 3 Mbps, 2.20 GB at 5 Mbps, and 3.52 GB at 8 Mbps.

### VOD ingest and asset size

MediaConvert planning assumes Professional tier AVC at 30 fps, one SD plus three HD outputs: `1 + 2 + 2 + 2 = 7 normalized minutes per source minute`, priced at $0.012/normalized minute. Exact codec, frame rate, captions, DRM, accelerated transcoding and Automated ABR settings change this materially.

| Asset | Duration min | Source GB | ABR output GB | Normalized min | One-time transcode | S3 Standard/month |
|---|---|---|---|---|---|---|
| Short episode | 30 | 5 | 2 | 210 | $2.52 | $0.16 |
| One-hour HD title | 60 | 20 | 6 | 420 | $5.04 | $0.60 |
| Two-hour premium title | 120 | 80 | 20 | 840 | $10.08 | $2.30 |

### Catalog storage assumptions

| Catalog | Hours | Raw archive | ABR output | Primary/month | Primary + recovery/month |
|---|---|---|---|---|---|
| Small | 100 | 2.0 TB | 600 GB | $22 | $44 |
| Medium | 1,000 | 19.5 TB | 5.9 TB | $218 | $436 |
| Large | 10,000 | 195.3 TB | 58.6 TB | $2,180 | $4,360 |

The standard combined estimate uses the medium row ($436/month) plus $504/month to encode 100 new source hours. Technical QC software, DRM licences, watermarking, subtitle vendors and content-rights fees are external contracts and are not included.

## Live-streaming scenarios

| Monthly live scenario | 0 | 1 | 100 | 1K | 10K | 100K |
|---|---|---|---|---|---|---|
| SD events: 20 h/mo, 1.8 Mbps | $42 | $42 | $96 | $1,343 | $11,229 | $62,097 |
| HD standard: 20 h/mo, 4.1 Mbps | $81 | $81 | $308 | $3,009 | $19,455 | $113,683 |
| HD busy schedule: 200 h/mo, 4.1 Mbps | $806 | $807 | $3,734 | $20,180 | $114,409 | $853,154 |
| High bitrate planning case: 20 h/mo, 8 Mbps | $150 | $150 | $670 | $5,455 | $31,495 | $192,218 |

The SD and HD fixed rates come from AWS's published live-streaming examples. The 8 Mbps line is a planning allowance, not a quoted MediaLive rate. MediaConnect is excluded because the architecture marks it optional; add it only when contribution routing or protocol requirements need it. A stopped MediaLive channel does not incur running channel charges.

### Live duration sensitivity before viewers

| HD live schedule | MediaLive + MediaPackage ingest |
|---|---|
| 1 hour | $4 |
| 20 hours/month | $81 |
| 200 hours/month | $806 |
| 24×7 / 730 hours | $2,942 |

Viewer delivery, MediaPackage origination and SSAI are added on top. At 10K concurrent HD viewers, one hour moves about 17.6 TB; this is why bitrate and CDN commercial terms matter far more than EKS for live scale.

## Standard combined monthly AWS estimate

Standard workload = 10 VOD hours/viewer/month at 5 Mbps + 20 live hours/month at 4.1 Mbps + medium catalog + 100 new VOD hours/month + the full platform. CloudFront free tier is applied once to the aggregated media traffic.

| Viewers | Media delivered | Platform | Catalog + encode | Media variable | AWS/month | AWS/year | $/viewer/month |
|---|---|---|---|---|---|---|---|
| 0 | 0 GB | $1,197 | $940 | $81 | $2,218 | $26,611 | — |
| 1 | 58 GB | $1,197 | $940 | $81 | $2,218 | $26,612 | $2,218 |
| 100 | 5.7 TB | $1,389 | $940 | $497 | $2,826 | $33,917 | $28 |
| 1K | 56.6 TB | $2,581 | $940 | $4,667 | $8,188 | $98,253 | $8 |
| 10K | 566.5 TB | $7,762 | $940 | $27,928 | $36,630 | $439,563 | $4 |
| 100K | 5.53 PB | $33,851 | $940 | $169,475 | $204,266 | $2,451,197 | $2 |

At zero viewers, the $81 media-variable value is the scheduled 20 live channel-hours. Subtract $81 when no live channel runs, $504 when no new content is encoded, and $436 when there is no stored catalog or recovery copy. The bare idle platform is therefore $1,197/month.

## Material sensitivities and optimizations

1. **CloudFront commercial model:** this estimate uses pay-as-you-go for transparent scaling. AWS now also offers flat-rate plans at $15, $200 and $1,000 per distribution with 50 TB allowances and no overages; Premium can be configured up to 600 TB for a new quoted price. For a media distribution under 50 TB/month, the $15 plan can be dramatically cheaper. Confirm that the plan and feature set fit the workload before replacing the PAYG calculation.
2. **Traffic geography:** US/Canada and Europe begin at $0.085/GB after the free TB, but India and APAC rates differ. A geographic traffic mix can move CDN cost by tens of percent.
3. **Committed discounts:** CloudFront Savings Bundle advertises up to 30%; EC2/RDS Savings Plans or reservations commonly reduce steady compute. No commitment discount is included.
4. **Codec and bitrate:** QVBR or HEVC can reduce delivery bytes, but HEVC/device/licensing trade-offs and additional encoding cost must be measured. A 20% bitrate reduction produces roughly a 20% CDN reduction.
5. **Live cache hit:** the model uses 99%. Falling to 97.5% makes MediaPackage origination 2.5× higher, though CDN transfer still dominates.
6. **Avoid idle media services:** stop MediaLive outside event windows; do not add MediaConnect or regional live DR until a business RTO/RPO requires them.
7. **Search and analytics:** the architecture correctly keeps OpenSearch rebuildable and starts with Firehose/S3/Glue/Athena/Quick rather than adding Redshift or a custom ML platform.

## Costs intentionally excluded

- DRM licence issuance, forensic watermarking, external ad decision server, payment processing and subscriber CRM.
- Technical/playback QC vendor licences, content acquisition, captioning/localization and device-lab certification.
- AWS Support plan, taxes, enterprise discounts and one-time cross-region migration/backfill transfer.
- Operations labour. A realistic 24×7 service adds roughly 3–5 platform/SRE/support FTE after launch; that labour is separate from AWS spend.
- Multi-CDN, hot regional live failover, dedicated CloudFront Anycast IPs and Shield Advanced unless contractually required.

## Official pricing references

- [Amazon CloudFront pay-as-you-go pricing](https://aws.amazon.com/cloudfront/pricing/pay-as-you-go/) and [flat-rate plans](https://aws.amazon.com/cloudfront/pricing/)
- [AWS Elemental MediaConvert pricing](https://aws.amazon.com/mediaconvert/pricing/)
- [AWS live-streaming cost examples](https://docs.aws.amazon.com/solutions/latest/live-streaming-on-aws/plan-your-deployment.html)
- [AWS Elemental MediaPackage pricing](https://aws.amazon.com/mediapackage/pricing/)
- [AWS Elemental MediaTailor pricing](https://aws.amazon.com/mediatailor/pricing/)
- [Amazon EKS pricing](https://aws.amazon.com/eks/pricing/)
- [Amazon Aurora pricing](https://aws.amazon.com/rds/aurora/pricing/)
- [Amazon OpenSearch Service pricing](https://aws.amazon.com/opensearch-service/pricing/)
- [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/)
- [Amazon VPC and NAT Gateway pricing](https://aws.amazon.com/vpc/pricing/) and [Elastic Load Balancing pricing](https://aws.amazon.com/elasticloadbalancing/pricing/)
- [Requested cost-estimator skill](https://github.com/alirezarezvani/claude-cto-team/blob/main/skills/cost-estimator/SKILL.md)

## Re-estimation inputs needed for a procurement-grade quote

Replace the planning assumptions with: viewer countries; monthly VOD viewer-hours; peak live concurrency and event-hours; bitrate distribution by device; title count, duration and codecs; new ingest hours/month; source/output storage per title; retention; cache-hit ratios; ad load; DRM/watermark quotes; RTO/RPO per tier; support plan; and negotiated CDN commitment. Once those are known, the uncertainty should fall from ±30–50% to roughly ±10–15%.
