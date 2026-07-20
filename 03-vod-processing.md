# Interactive AWS VOD Architecture

[Open the interactive diagram](./03-vod-processing.html) · [Back to suite](./index.html)

Companion guide for `vod-architecture-interactive.html`.

## Flow descriptions

### vod publish

- **What:** Publish a source asset as protected adaptive VOD content.
- **Why:** Raw mezzanine files cannot be delivered safely or efficiently to diverse viewer devices.
- **How:** S3 events start Step Functions; Lambda validates the asset; MediaConvert transcodes it; QC gates MediaPackage, DRM, origin, and CDN publication.

### qc failure

- **What:** Stop and remediate an asset that fails post-transcode quality control.
- **Why:** Defective video, audio, captions, or manifests must never enter the customer-facing origin.
- **How:** QC emits a structured failure, the workflow quarantines outputs, applies retry or review policy, and returns an actionable status.

### playback

- **What:** Authorize and deliver a protected VOD playback session.
- **Why:** The platform must enforce rights while keeping the high-bandwidth media path scalable and cacheable.
- **How:** The player presents short-lived authorization, resolves DRM and watermark policy, and receives signed manifests and segments through CloudFront.

### forensic

- **What:** Investigate suspected piracy using preserved asset and playback evidence.
- **Why:** Attribution requires integrity-verified records linking the recovered copy to a known asset and session.
- **How:** Source hashes, QC records, DRM, watermark, CDN, and player identifiers enter the evidence vault and are analyzed from verified working copies.

## Purpose

The interactive diagram explains four complete scenarios:

1. Publishing a VOD asset successfully.
2. Handling a post-transcode quality-control failure.
3. Authorizing and delivering protected playback.
4. Investigating suspected piracy using preserved digital evidence.

The HTML diagram supports flow tabs, next/previous controls, autoplay, keyboard navigation, node selection, draggable nodes, fullscreen mode, and dark/light themes.

## Components

| Component | AWS services or function | Responsibility |
|---|---|---|
| Content upload | Content owner or CMS | Supplies the mezzanine video and required metadata. |
| S3 raw ingest | Amazon S3 | Preserves the original source separately from processed outputs. |
| Workflow | Amazon EventBridge and AWS Step Functions | Starts and coordinates one idempotent asset-processing execution. |
| Validate and identify | AWS Lambda and security scanning | Validates the source, checks malware status, assigns an asset ID, and calculates SHA-256. |
| MediaConvert | AWS Elemental MediaConvert | Produces the adaptive-bitrate video, audio, and caption renditions. |
| Post-transcode QC | External or custom automated QC | Decodes and validates MediaConvert outputs before publication. |
| Package and protect | AWS Elemental MediaPackage, DRM integration, and forensic watermarking | Creates HLS/DASH presentations, applies content protection, and assigns traceable watermark data. |
| Origin and CDN | Amazon S3 and Amazon CloudFront | Stores approved manifests and segments and delivers them globally. |
| Viewer player | Web, mobile, TV, or streaming-device application | Requests authorized playback, obtains DRM rights, adapts bitrate, and emits telemetry. |
| Quarantine | Isolated Amazon S3 prefix or bucket | Prevents failed media from entering the origin namespace. |
| Evidence vault | Amazon S3 Object Lock and Amazon Security Lake | Preserves immutable evidence and normalizes security telemetry. |
| Investigator | Amazon Detective, Athena, and the case process | Builds a verified working set, reconstructs events, and documents findings and actions. |

## Flow 1 - Publish a VOD asset

1. The CMS uploads the original mezzanine and metadata to S3 raw ingest.
2. The S3 object-created event reaches EventBridge and starts Step Functions.
3. Lambda validates the trust boundary, checks malware status, assigns the immutable asset ID, and records the source SHA-256.
4. The workflow submits an ABR job to MediaConvert.
5. Completed outputs enter a separate post-transcode QC process.
6. A passing result allows MediaPackage, DRM, and forensic-watermark processing.
7. Only approved outputs are promoted to the S3 origin and CloudFront distribution.
8. An authorized player receives the manifest and streams adaptive segments.

## Flow 2 - Handle a QC failure

QC is a publish gate rather than a decorative monitoring step.

1. QC evaluates the completed MediaConvert outputs.
2. A blocking defect produces a structured failure result.
3. Failed outputs enter quarantine and remain unavailable to the delivery origin.
4. Step Functions records the failure and applies the approved retry or operator-review policy.
5. The content owner receives the exact rule failure and remediation status.

Typical QC checks:

- Decode integrity and corrupt frames.
- Black or frozen frames.
- Audio presence, loudness, channel mapping, and A/V synchronization.
- Codec, resolution, frame rate, bitrate, color, and HDR conformance.
- Caption presence, format, and synchronization.
- Manifest, segment, and playback integrity.

## Flow 3 - Authorize and deliver playback

1. The player requests a manifest with short-lived playback authorization.
2. The protected presentation resolves DRM policy and a session-specific forensic-watermark identifier.
3. CloudFront returns the protected manifest and segments without proxying video through general application compute.
4. The player obtains the required DRM license, adapts rendition selection, and emits QoE events containing the playback-session identifier.

## Flow 4 - Investigate suspected piracy

1. Source provenance is preserved using the asset ID, original object version, and SHA-256 digest.
2. QC tool version, checks, result, and normalized UTC time are preserved.
3. Playback, CDN, DRM-license, and watermark records are correlated using stable identifiers.
4. The investigator receives verified working copies while originals remain retention-locked.
5. A watermark extracted from a suspected copy is resolved to the asset and playback session.
6. Authorized responders may revoke sessions, restrict accounts, update controls, and preserve a legal hold.

## Evidence keys

| Key | Use |
|---|---|
| Asset ID | Connects source, renditions, catalog entry, protection policy, and evidence. |
| Source SHA-256 | Verifies that evidence or recovered content matches a known asset. |
| Workflow execution ID | Connects orchestration events and processing decisions. |
| MediaConvert job ID | Identifies the precise transformation job and configuration. |
| QC result ID | Identifies the checks, tool version, measurements, and outcome. |
| DRM license ID | Connects rights issuance to account, device, title, and session. |
| Watermark ID | Connects a recovered copy to a distributed playback session. |
| Playback session ID | Connects authorization, DRM, CDN, and player telemetry. |
| CDN request ID | Supports delivery-path reconstruction and troubleshooting. |
| UTC timestamp | Enables a consistent cross-system timeline. |

## Chain of custody

Every material evidence item should include:

- Case ID and evidence ID.
- Source account, Region, service, resource, and object version.
- Collector identity and collection time in UTC.
- SHA-256 calculated or verified at acquisition.
- Evidence-vault bucket, object key, version, and retention status.
- Every access, export, copy, transfer, analysis action, and disposition.
- Verification that analysis used a working copy rather than modifying the preserved original.

## Operational decisions to confirm

- Supported codecs, resolutions, HDR formats, audio layouts, captions, and playback devices.
- QC vendor or implementation and which rules block publication.
- Retry limits and which failures require operator approval.
- DRM providers, license policies, and forensic-watermark integration.
- Evidence-retention periods, Object Lock mode, data residency, and legal-hold authority.
- Required concurrent-viewer capacity, publication latency, availability, RPO, and RTO.

## Opening the diagram

On macOS:

```bash
open outputs/vod-architecture-interactive.html
```

The file is self-contained except for optional Google Fonts; it falls back to system fonts when offline.
