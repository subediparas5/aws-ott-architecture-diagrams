# Analytics, Recommendation and Ad-Tech

Follow viewer events into analytics, model training and inference, personalized recommendations, and server-side advertising.

[Open the interactive diagram](./05-analytics-recommendation-adtech.html) · [Back to suite](./index.html)

## Components

| Component | Technology | Scope |
|---|---|---|
| Player / app | Playback + interaction | event producer |
| Kinesis | Event ingestion | ordered shards |
| S3 data lake | Raw + curated | governed data |
| AWS Glue | Catalog + ETL | schema controlled |
| Redshift | Business warehouse | modeled data |
| Amazon Quick | Dashboards | QoE + KPIs |
| SageMaker AI | Training + registry | model versions |
| Recommendation API | Online ranking | personalization |
| Ad decision | Audience + campaign | rules engine |
| MediaTailor | Server-side insertion | personalized stream |
| Ad measurement | Impression + quartile | reporting |

## 1. Analyze viewer behavior

- **What:** Transform player and application events into operational and business metrics.
- **Why:** Teams need trustworthy QoE, engagement, and revenue signals rather than raw event streams.
- **How:** Kinesis ingests versioned events, S3 retains them, Glue curates them, Redshift models them, and Amazon Quick presents dashboards.

1. **Publish player events:** The player emits versioned events for playback, QoE, and interaction.
2. **Land immutable events:** Events are partitioned by date and type in the data lake.
3. **Catalog and transform:** Glue validates schemas and produces curated datasets.
4. **Load analytical models:** Curated facts and dimensions enter Redshift.
5. **Publish operational insight:** Dashboards expose QoE, engagement, and business KPIs.

## 2. Generate recommendations

- **What:** Train a recommendation model and return a personalized title slate.
- **Why:** Relevant discovery improves engagement while entitlement rules still constrain what may be shown.
- **How:** Curated lake data trains a versioned SageMaker model; the recommendation API performs online ranking and captures feedback events.

1. **Build training features:** Curated viewing and catalog data become versioned features.
2. **Publish approved model:** Only an evaluated model version reaches the recommendation endpoint.
3. **Request a ranked slate:** The client supplies the account context and placement.
4. **Return personalized titles:** The API returns ranked entitled titles and impression identifiers.
5. **Close the feedback loop:** Impressions and watch outcomes become future training signals.

## 3. Insert and measure an ad

- **What:** Select, insert, and measure an advertisement in a playback session.
- **Why:** The platform must monetize content while honoring campaign, frequency, and consent rules.
- **How:** The player creates an ad opportunity, the decision service selects a campaign, MediaTailor stitches it, and measurement events return to analytics.

1. **Request an ad decision:** Playback context and privacy-safe audience signals reach the decision service.
2. **Return campaign decision:** The decision service chooses eligible creative and tracking metadata.
3. **Deliver stitched stream:** MediaTailor inserts the selected ad into the viewer-specific stream.
4. **Report playback milestones:** The player or server reports auditable measurement events.
5. **Publish ad events:** Ad outcomes join content and business analytics.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
