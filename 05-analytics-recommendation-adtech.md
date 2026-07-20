# Analytics, Recommendation and Ad-Tech

Follow privacy-safe telemetry into S3 and Athena, managed recommendations, and external ad decisions stitched by MediaTailor.

[Open the interactive diagram](./05-analytics-recommendation-adtech.html) · [Back to suite](./index.html)

## Architecture decisions

- Firehose is the default telemetry delivery service; Kinesis Data Streams is not needed without real-time stream consumers.
- S3, Glue, Athena, and Amazon Quick are the initial analytics stack; Redshift is deferred.
- Editorial and popularity slates are the fallback; Amazon Personalize supplies managed VOD recommendations before any custom SageMaker model.
- MediaTailor calls an external ADS; campaign management and pacing are not rebuilt inside the OTT platform.

## Add only when

- Add Kinesis Data Streams when several real-time consumers need the same ordered telemetry stream.
- Add Redshift Serverless when Athena cannot meet measured dashboard concurrency, latency, or modeling requirements.
- Add SageMaker only when a custom model demonstrably outperforms Amazon Personalize on agreed business metrics.

## Components

| Component | Technology | Scope |
|---|---|---|
| Player / app | Playback + interaction | event producer |
| Telemetry API | Schema · consent · throttle | public ingestion |
| Amazon Data Firehose | Managed delivery | buffer + partition |
| S3 data lake | Immutable raw + curated | governed data |
| Glue + Athena | Catalog + SQL transforms | serverless analytics |
| Amazon Quick | QoE + business dashboards | governed metrics |
| Recommendation API | Entitlement safety gate | online response |
| Amazon Personalize | VOD recommender | managed model |
| CloudFront | Manifest and segment routing | SSAI edge |
| MediaTailor | Server-side insertion | personalized manifest |
| External ad server | VAST / VMAP | campaign decision |
| Ad measurement | Impression + quartiles | auditable events |

## 1. Analyze viewer behavior

- **What:** Transform player and application events into operational and business metrics.
- **Why:** Teams need trustworthy QoE, engagement, and revenue signals rather than raw event payloads.
- **How:** A protected ingestion API validates consent and schema; Firehose lands partitioned events in S3; Glue and Athena curate them for Amazon Quick.

1. **Publish a privacy-safe event:** The API validates schema version, consent state, size, and rate limits before accepting the event.
2. **Send to managed delivery:** The ingestion boundary forwards validated events without exposing AWS credentials to clients.
3. **Land immutable partitions:** Firehose buffers, compresses, and partitions records in the raw zone.
4. **Catalog and curate:** Schema-controlled transforms produce curated facts while preserving raw evidence of the event received.
5. **Publish governed metrics:** Amazon Quick presents agreed QoE, engagement, and business definitions.

## 2. Return recommendations

- **What:** Return an entitled personalized title slate.
- **Why:** Relevant discovery can improve engagement, but a custom ML platform is unnecessary until managed recommendations fail a measured requirement.
- **How:** Curated interactions feed Amazon Personalize; the recommendation API applies placement and final entitlement rules and records impressions through the common telemetry path.

1. **Import catalog and interactions:** Curated title and interaction data feeds the managed VOD recommender.
2. **Request a title slate:** The request contains account context and placement but no unnecessary profile attributes.
3. **Request managed ranking:** The API requests a ranked candidate set and falls back to editorial or popular content if unavailable.
4. **Return ranked candidates:** Amazon Personalize returns candidate IDs and recommendation context.
5. **Return entitled titles:** The API performs the final authoritative entitlement and regional safety check before returning the slate.
6. **Record impression and outcome:** Impressions and later watch outcomes return through the same consent-aware telemetry API.

## 3. Insert and measure an ad

- **What:** Select, stitch, deliver, and measure an advertisement.
- **Why:** The platform must monetize content while honoring consent and campaign rules without rebuilding an ad server.
- **How:** CloudFront routes manifest requests to MediaTailor; MediaTailor requests VAST/VMAP from the ADS, returns a personalized manifest, and measurement events re-enter the governed telemetry boundary.

1. **Request a playback manifest:** The player requests the manifest through the CDN with consent-safe session parameters.
2. **Route the manifest request:** CloudFront sends manifest requests to MediaTailor while segment paths remain cacheable.
3. **Request an ad decision:** MediaTailor calls the external ADS using privacy-safe audience and opportunity signals.
4. **Return eligible creative:** The ADS owns campaign selection, pacing, frequency caps, and tracking metadata.
5. **Return personalized manifest:** MediaTailor replaces ad markers with URLs for the selected ad segments.
6. **Deliver content and ads:** CloudFront serves the personalized manifest and routes content and ad segments to the correct origins.
7. **Report playback milestones:** Player or server-side beacons record auditable delivery milestones.
8. **Publish governed ad events:** Ad outcomes enter the same schema, consent, and retention controls as other events.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
