# Live Streaming Architecture

Step through dual-feed ingest, live encoding and packaging, operational failover, live-to-VOD, and forensic telemetry.

[Open the interactive diagram](./04-live-streaming.html) · [Back to suite](./index.html)

## Components

| Component | Technology | Scope |
|---|---|---|
| Live sources | Cameras · studios | primary + backup |
| MediaConnect | Contribution ingest | redundant flow |
| MediaLive | ABR encoding | standard channel |
| MediaPackage | JIT packaging · DRM | HLS / DASH |
| CloudFront | Global delivery | low-latency |
| Viewer player | Web · mobile · TV | live session |
| Operations | CloudWatch · alarms | operator console |
| Live archive | S3 live-to-VOD | segments |
| Evidence vault | Logs · fingerprints | Object Lock |
| Backup feed | Independent path | failover source |

## 1. Deliver a live event

- **What:** Deliver a live source to viewers as an adaptive protected stream.
- **Why:** Live events require continuous processing, device compatibility, and global low-latency delivery.
- **How:** Redundant feeds enter MediaConnect, MediaLive creates ABR outputs, MediaPackage protects them, and CloudFront delivers them.

1. **Acquire contribution feeds:** MediaConnect receives redundant contribution feeds.
2. **Deliver clean input:** MediaLive receives the selected contribution input.
3. **Encode the ABR ladder:** MediaLive produces synchronized adaptive renditions.
4. **Package and originate:** MediaPackage applies packaging and protection policies.
5. **Deliver live segments:** CloudFront delivers manifests and segments to viewers.

## 2. Fail over the contribution feed

- **What:** Move the live channel from an impaired primary feed to a backup feed.
- **Why:** A contribution failure must not terminate a commercially important live event.
- **How:** CloudWatch detects input impairment, the managed ingest path selects the independent backup, and operators verify MediaLive output recovery.

1. **Detect input impairment:** Monitoring detects loss, corruption, or unacceptable input quality.
2. **Select backup contribution:** MediaConnect or the channel selects the independent backup path.
3. **Continue encoding:** MediaLive continues output after the controlled input switch.
4. **Confirm recovery:** Operations confirms output health and documents the failover.

## 3. Create live-to-VOD content

- **What:** Create replay, catch-up, or highlight content from a live event.
- **Why:** Live content should remain monetizable and accessible after the original broadcast window.
- **How:** MediaPackage writes segment-aligned media to S3, creates a replay presentation, and republishes it through the protected CDN path.

1. **Archive live segments:** Selected manifests and segments are written to durable storage.
2. **Create replay presentation:** A defined time range becomes a replay or highlight presentation.
3. **Publish replay:** The approved replay uses the same protected delivery path.

## 4. Preserve live forensic telemetry

- **What:** Preserve fingerprints, packaging identifiers, and viewer-session records for a live event.
- **Why:** Piracy and incident investigations must connect a recovered stream to its source and distributed session.
- **How:** Contribution, packaging, watermark, CDN, and player telemetry converge in a retention-locked evidence vault.

1. **Record source fingerprint:** The platform records contribution fingerprints and trusted timestamps.
2. **Record stream identifiers:** Segment timing, DRM, and watermark identifiers are preserved.
3. **Record playback session:** Delivery and player records tie the distributed stream to a playback session.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
