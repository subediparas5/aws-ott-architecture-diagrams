# Executive Overview

Explore business requests, media delivery, event analytics, and forensic telemetry without mixing every internal service into one canvas.

[Open the interactive diagram](./01-executive-overview.html) · [Back to suite](./index.html)

## Components

| Component | Technology | Scope |
|---|---|---|
| Viewer applications | Web · mobile · TV | millions of sessions |
| Protected edge | Route 53 · CloudFront · WAF | TLS + DDoS |
| API layer | API Gateway | auth + throttle |
| OTT services | Microservices on EKS | business APIs |
| Operational data | Aurora · cache · search | catalog + users |
| Event backbone | MSK / Kinesis | player + app events |
| Media origin | S3 manifests + segments | approved content |
| Global delivery | CloudFront / multi-CDN | adaptive media |
| Content sources | Live feeds + uploads | contribution |
| Media processing | Elemental services | live + VOD |
| Analytics + ML | Warehouse · dashboards | recommendation |
| Forensics | Evidence + response | chain of custody |

## 1. Browse the catalog

- **What:** Browse an entitled and region-valid content catalog.
- **Why:** Viewers need fast discovery without exposing internal services or unavailable titles.
- **How:** The request crosses CloudFront and WAF, API Gateway, EKS catalog services, and the operational data layer before returning a filtered response.

1. **Enter protected edge:** CloudFront and WAF protect the public entry point.
2. **Route the API request:** API Gateway validates and routes the request.
3. **Call catalog services:** The application layer applies catalog, entitlement, and personalization rules.
4. **Read operational data:** The services read the systems optimized for each access pattern.
5. **Return catalog results:** Catalog results return to the service layer.
6. **Render the catalog:** The viewer receives only entitled, region-valid content.

## 2. Authorize and play content

- **What:** Authorize a playback session and deliver protected video.
- **Why:** Business authorization and high-bandwidth media delivery must scale independently.
- **How:** EKS evaluates entitlement, resolves the approved presentation, and returns signed access while S3 origin and CloudFront carry the media.

1. **Request playback:** The player requests a session for one entitled title.
2. **Evaluate entitlement:** Playback and entitlement services evaluate subscription, geography, and concurrency.
3. **Resolve presentation:** The application returns the approved protected presentation.
4. **Serve through CDN:** Origin content is cached and delivered globally.
5. **Start adaptive playback:** The player selects renditions and emits QoE events.

## 3. Publish content

- **What:** Turn live feeds or uploaded source files into distributable media.
- **Why:** Devices require validated, encoded, packaged, and protected outputs rather than source mezzanine files.
- **How:** AWS Elemental processing creates approved renditions and promotes them to the protected origin and CDN.

1. **Ingest source content:** Live feeds and uploaded files enter the appropriate media workflow.
2. **Create publishable media:** Processing produces approved manifests and segments.
3. **Distribute content:** The CDN retrieves approved content from the protected origin.

## 4. Capture forensic evidence

- **What:** Capture a trustworthy evidence trail across playback and platform activity.
- **Why:** Security, fraud, and piracy investigations require correlated records that existed before the incident.
- **How:** Session events enter the event backbone, are normalized by analytics, and relevant records are retained in the forensic evidence system.

1. **Publish session telemetry:** Playback records carry stable session, account, device, and asset identifiers.
2. **Normalize and correlate:** Analytics pipelines produce operational and business views.
3. **Preserve security evidence:** Relevant events are preserved under retention and custody controls.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
