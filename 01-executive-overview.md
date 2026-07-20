# Executive Overview

Follow the business, media, event, analytics, and evidence paths without turning the executive view into a service inventory.

[Open the interactive diagram](./01-executive-overview.html) · [Back to suite](./index.html)

## Architecture decisions

- CloudFront is the only CDN until measured availability or commercial requirements justify multi-CDN operations.
- The application begins as a small set of modular deployables; service boundaries split only when ownership or scaling differs.
- Security Lake stores normalized security telemetry; case evidence is promoted to a separate Object Lock vault.

## Add only when

- Add a second CDN only after provider-diversity or regional performance objectives require it.
- Add independent microservices, caches, or streaming brokers only after measured load or team boundaries require them.

## Components

| Component | Technology | Scope |
|---|---|---|
| Viewer applications | Web · mobile · TV | customer sessions |
| Public edge | Route 53 · CloudFront · WAF | TLS + DDoS |
| Protected API | API Gateway · JWT | routing + throttle |
| OTT application | Modular services on EKS | business APIs |
| Operational data | Aurora · OpenSearch | authoritative + derived |
| Events + telemetry | EventBridge · SQS · Firehose | async + analytics |
| Approved media origin | Amazon S3 | manifests + segments |
| Global delivery | Amazon CloudFront | adaptive media |
| Content sources | Live feeds + uploads | contribution |
| Managed media processing | MediaConvert · MediaLive · MediaPackage v2 | live + VOD |
| Lake analytics | S3 · Athena · Amazon Quick | QoE + business insight |
| Security + evidence | Security Lake · Object Lock vault | separate stores |

## 1. Browse the catalog

- **What:** Browse an entitled and region-valid catalog.
- **Why:** Viewers need fast discovery without exposing internal services or unavailable titles.
- **How:** WAF and API Gateway validate the request; the application queries authoritative and derived data before returning policy-filtered results.

1. **Enter the public edge:** CloudFront serves the application and WAF rejects abusive traffic.
2. **Validate the API boundary:** API Gateway validates the JWT, route, and throttle policy.
3. **Call the OTT application:** The request crosses the private integration to the application boundary.
4. **Query catalog and rights:** The application supplies availability and entitlement constraints to the data query.
5. **Return safe result candidates:** Only candidates valid for the requested market and rights window are returned.
6. **Render the catalog:** A final application-side entitlement check protects against stale search data.

## 2. Authorize and play content

- **What:** Authorize one playback session and deliver protected video.
- **Why:** Business authorization and high-bandwidth media delivery must scale independently.
- **How:** The application returns short-lived access to an approved presentation; CloudFront then serves media directly from the protected S3 origin.

1. **Request playback authorization:** The player requests one title for a known device and session.
2. **Evaluate playback policy:** The application evaluates subscription, geography, concurrency, and device policy.
3. **Load rights and presentation:** Aurora provides rights and asset metadata; OpenSearch is not authoritative for playback.
4. **Return approved asset:** The result identifies the protected presentation and active rights window.
5. **Issue signed playback access:** The control plane returns a short-lived manifest URL or signed cookie.
6. **Request the manifest:** The player begins the high-bandwidth media path through CloudFront.
7. **Fetch on cache miss:** CloudFront reaches the private S3 origin only when the object is not cached.
8. **Return approved media:** S3 returns only objects in the approved origin namespace.
9. **Start adaptive playback:** The player receives cached manifests and segments without proxying video through EKS.

## 3. Publish content

- **What:** Turn a live feed or uploaded file into distributable media.
- **Why:** Devices require validated, encoded, packaged, protected, and approved outputs.
- **How:** Managed AWS media services process the source; only content that passes its publish gates enters the S3 origin and CloudFront.

1. **Ingest source content:** The appropriate managed workflow receives the live feed or immutable upload version.
2. **Publish approved outputs:** Only technically valid and playback-tested manifests and segments enter the origin.
3. **Distribute content:** CloudFront retrieves approved content from the protected origin.

## 4. Capture trustworthy evidence

- **What:** Create a trustworthy evidence trail across playback and platform activity.
- **Why:** Security, fraud, and piracy investigations need correlated records that existed before the incident.
- **How:** Privacy-safe telemetry lands through managed delivery; investigators promote only selected case material into the immutable evidence vault.

1. **Publish session telemetry:** Events carry stable session, device, asset, and consent-safe account identifiers.
2. **Land and query telemetry:** Managed delivery writes partitioned immutable events for Athena and dashboards.
3. **Promote selected case evidence:** Security telemetry stays in Security Lake; selected records and hashes enter the separate Object Lock vault.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
