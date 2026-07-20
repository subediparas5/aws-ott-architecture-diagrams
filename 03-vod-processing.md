# Interactive AWS VOD Architecture

Follow immutable upload, managed malware status, ABR processing, technical QC, packaging and DRM, playback QC, protected delivery, and piracy evidence.

[Open the interactive diagram](./03-vod-processing.html) · [Back to suite](./index.html)

## Architecture decisions

- VOD uses static HLS/DASH packages in S3; MediaPackage is not in this path unless just-in-time packaging becomes a real requirement.
- Technical QC runs before packaging and protection; a second end-to-end gate tests manifests, DRM, captions, and representative playback.
- GuardDuty Malware Protection for S3 supplies upload scan status; the workflow handles duplicate events and keys execution by object version.
- Source fingerprinting is separate from session-specific forensic watermarking and licence issuance.

## Add only when

- Add MediaPackage or another JIT packager only when output-format proliferation makes static packaging uneconomical.
- Add a commercial watermark vendor only when piracy attribution is a funded business requirement.
- Use a specialist QC vendor when the required codec, HDR, caption, or device matrix exceeds the internal QC service.

## Components

| Component | Technology | Scope |
|---|---|---|
| Content owner / CMS | Mezzanine + metadata | upload + status |
| S3 raw ingest | Versioned original | quarantine namespace |
| Scan + workflow | GuardDuty · EventBridge · Step Functions | idempotent asset |
| MediaConvert stage 1 | ABR renditions | jobId |
| Technical QC | Decode · audio · captions | PASS / FAIL |
| Package + protect | MediaConvert · SPEKE DRM | static HLS / DASH |
| Playback QC | Manifest · DRM · device smoke | PASS / FAIL |
| Approved origin + CDN | S3 · CloudFront OAC | signed media path |
| Viewer player | Web · mobile · TV | playback session |
| DRM + watermark service | Licence + session mapping | vendor integration |
| Failed-output quarantine | Isolated S3 namespace | not publishable |
| Evidence vault | S3 Object Lock | case-specific originals |
| Piracy investigation | Athena + case workflow | verified working copies |

## 1. Publish a protected VOD asset

- **What:** Publish one immutable source as protected adaptive VOD.
- **Why:** Raw mezzanine content cannot be delivered safely or efficiently to diverse devices.
- **How:** An object-versioned workflow waits for malware status, creates ABR renditions, runs technical QC, packages and encrypts, runs playback QC, and promotes only passing output to S3 and CloudFront.

1. **Upload immutable source and metadata:** The CMS stores the original object version separately from every processed output.
2. **Start one safe asset execution:** The workflow starts only after GuardDuty reports a non-malicious object; bucket, key, and version form the idempotency key.
3. **Create ABR renditions:** Validation records the asset ID, source checksum, supported tracks, and selected output profile before stage-one encoding.
4. **Run technical media QC:** The first gate decodes stage-one outputs and validates essence, audio, captions, and encoding conformance.
5. **Package and encrypt:** A second managed job creates static HLS/DASH packages in staging and obtains content keys from the approved DRM provider.
6. **Run end-to-end playback QC:** The second gate verifies manifests, segments, licence acquisition, captions, seek behavior, and representative playback.
7. **Promote approved output:** Only content that passes both gates enters the approved S3 namespace exposed through CloudFront OAC.
8. **Return publication status:** The content owner receives the exact approved presentation and immutable processing identifiers.

## 2. Handle a technical QC failure

- **What:** Stop and remediate an asset that fails technical QC.
- **Why:** Defective essence, audio, captions, or encoding must never enter the customer-facing origin.
- **How:** The technical gate emits a structured failure, quarantines stage-one output, applies controlled retry or review policy, and returns an actionable status.

1. **Detect a blocking defect:** The first gate rejects a publish-blocking rule such as decode integrity, A/V sync, or required-caption failure.
2. **Quarantine failed outputs:** The failed rendition set is tagged and isolated so no later step can promote it.
3. **Choose controlled remediation:** Step Functions records the failure and selects a bounded retry or operator review.
4. **Return the exact failure:** The CMS receives the failed rule and remediation state instead of a generic processing error.

## 3. Authorize and deliver playback

- **What:** Authorize one protected VOD playback session.
- **Why:** Rights must be enforced without proxying high-bandwidth video through EKS.
- **How:** The playback edge validates short-lived access, resolves DRM and session watermark policy, returns cached media, and lets the player request its licence directly.

1. **Request the signed manifest:** The player presents short-lived authorization tied to account, device, title, and session.
2. **Resolve session protection:** The edge resolves the DRM policy and, when enabled, creates a session-specific watermark mapping.
3. **Return protected session policy:** The service returns licence metadata, approved presentation, and optional watermark mapping.
4. **Return manifests and segments:** CloudFront serves protected media directly from the approved S3 origin.
5. **Request a DRM licence:** The player presents its DRM challenge and playback-session context directly to the rights service.
6. **Issue licence and record mapping:** The response enables playback and records the minimum session, licence, and watermark identifiers needed for audit.

## 4. Investigate suspected piracy

- **What:** Investigate a suspected pirated copy using preserved provenance and playback records.
- **Why:** Attribution requires integrity-verified records that connect the recovered copy to an asset and session.
- **How:** The evidence vault stores selected originals and manifests; investigators use verified working copies and specialist watermark results.

1. **Preserve source provenance:** The case manifest records the asset ID, original object version, source hash, and workflow identifiers.
2. **Preserve QC evidence:** The exact tool version, checks, measurements, result, and UTC timestamp are retained.
3. **Preserve session protection records:** DRM licence and watermark mappings are linked by stable asset and playback-session identifiers.
4. **Create a verified working set:** Originals remain locked; the investigator receives hash-verified working copies and custody records.
5. **Resolve recovered watermark:** A specialist extraction result maps the suspected copy to an asset and authorized playback session.
6. **Contain and document:** Authorized responders revoke the affected session or account and preserve the decision in the case record.

## QC policy

Technical QC checks decoded essence: corrupt or frozen frames, A/V sync, loudness and channel mapping, caption presence and timing, codec, frame rate, bitrate, color, and HDR conformance. Playback QC runs after packaging and protection and verifies manifests, segment continuity, DRM licence acquisition, captions, seek behavior, and representative devices.

## Evidence keys

Asset ID, S3 object version, source SHA-256, workflow execution ID, MediaConvert job IDs, both QC result IDs, DRM licence ID, watermark ID, playback-session ID, CDN request ID, and normalized UTC timestamps connect the case.

## Chain of custody

Original evidence remains in an Object Lock bucket. Every acquisition records the source account, Region, service, object version, collector identity, UTC time, SHA-256, retention state, and case ID. Analysis uses hash-verified working copies; access, export, transfer, action, and disposition are recorded.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
