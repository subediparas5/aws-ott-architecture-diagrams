# Live Streaming Architecture

Follow contribution, dual-input encoding, MediaPackage v2, CloudFront delivery, feed failover, harvest jobs, and live evidence.

[Open the interactive diagram](./04-live-streaming.html) · [Back to suite](./index.html)

## Architecture decisions

- MediaPackage v2 is the live packager and origin.
- MediaConnect is optional; bypass it when MediaLive can ingest the required protocols and no managed contribution routing is needed.
- Dual-feed failover protects contribution. Regional live DR is a separate commercial decision and is not implied by input failover.

## Add only when

- Reserve a secondary-region live pipeline only for events whose contractual or commercial loss exceeds its standby cost.
- Add multi-CDN delivery only after provider diversity or measured performance requires it.

## Components

| Component | Technology | Scope |
|---|---|---|
| Live sources | Cameras · studios | primary feed |
| Optional contribution | MediaConnect | route / convert protocols |
| MediaLive | Dual-input ABR encoding | standard channel |
| MediaPackage v2 | JIT packaging · SPEKE DRM | LL-HLS / DASH |
| CloudFront | Protected global delivery | low latency |
| Viewer player | Web · mobile · TV | live session |
| Operations | CloudWatch · alarms | runbooks |
| Harvested live-to-VOD | MediaPackage v2 → S3 | replay clips |
| Live evidence vault | S3 Object Lock | case-specific |
| Independent backup feed | Separate source and path | input failover |

## 1. Deliver a live event

- **What:** Deliver a live source as an adaptive protected stream.
- **Why:** Live events require continuous encoding, device compatibility, rights protection, and low-latency delivery.
- **How:** An optional contribution layer feeds a dual-pipeline MediaLive channel; MediaPackage v2 packages and protects, and CloudFront serves viewers.

1. **Acquire contribution:** MediaConnect is used only when managed routing, protocol conversion, or distribution is required.
2. **Feed the standard channel:** MediaLive receives the primary and independent backup inputs.
3. **Encode the ABR ladder:** MediaLive produces synchronized adaptive renditions and preserves required ad markers.
4. **Package and protect:** MediaPackage v2 applies JIT packaging and SPEKE DRM policy.
5. **Deliver live media:** CloudFront serves manifests and segments to the player.

## 2. Fail over the contribution feed

- **What:** Move encoding from an impaired primary input to an independent backup.
- **Why:** A contribution failure must not terminate a commercially important live event.
- **How:** Monitoring detects impairment, the managed input policy selects the backup, MediaLive continues output, and operations verifies recovery.

1. **Detect input impairment:** Loss, corruption, or unacceptable input quality crosses the failover threshold.
2. **Select the independent backup:** The backup uses an independent source and contribution path so one fault does not remove both feeds.
3. **Continue channel input:** MediaLive receives the backup input under the configured automatic failover policy.
4. **Verify output recovery:** Operations confirms encoded output and end-viewer playback before closing the incident.

## 3. Create live-to-VOD content

- **What:** Create replay, catch-up, or highlight content from a live event.
- **Why:** Live content should remain monetizable and accessible after the original broadcast window.
- **How:** A MediaPackage v2 harvest job writes the selected segment-aligned window to S3; CloudFront then serves the approved replay.

1. **Create a harvest job:** The requested start and end time must fit the endpoint startover window and supported duration.
2. **Publish the approved replay:** The harvested manifests and segments enter the protected replay namespace after required checks.
3. **Deliver replay content:** The player uses the protected CDN path for catch-up or highlights.

## 4. Preserve live forensic telemetry

- **What:** Preserve fingerprints, stream identifiers, and viewer-session records for one live event.
- **Why:** Piracy investigations must connect a recovered stream to its source and distributed session.
- **How:** Selected contribution, packaging, licence, CDN, and player records enter the Object Lock evidence vault under a case manifest.

1. **Record source fingerprint:** The platform records contribution fingerprints and trusted timestamps.
2. **Record stream protection IDs:** Segment timing, DRM key identifiers, and watermark mappings are preserved when material to the case.
3. **Record playback-session evidence:** Delivery and player records connect the distributed stream to a playback session without retaining unnecessary personal data.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
