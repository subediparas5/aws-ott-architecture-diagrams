# Platform and Microservices

Inspect catalog, search, playback, entitlement, and asynchronous service interactions inside the application VPC.

[Open the interactive diagram](./02-platform-microservices.html) · [Back to suite](./index.html)

## Components

| Component | Technology | Scope |
|---|---|---|
| Client | Viewer application | HTTPS |
| Edge security | CloudFront · WAF | protected entry |
| API Gateway | JWT · routing · throttle | public API |
| EKS services | Auth · catalog · playback | private subnets |
| Aurora | Users · catalog · rights | source of truth |
| ElastiCache | Sessions · hot objects | TTL cache |
| OpenSearch | Title discovery | search index |
| Amazon MSK | Domain events | async backbone |
| Event consumers | Notifications · analytics | autoscaled |
| Secrets + KMS | Keys · runtime secrets | least privilege |
| Observability | Metrics · logs · traces | operations |

## 1. Authenticate a viewer

- **What:** Authenticate a viewer and issue a short-lived application session.
- **Why:** Every downstream catalog and playback decision needs a verified account and device context.
- **How:** The request passes the protected edge and API Gateway to the EKS authentication service, which reads account policy from Aurora.

1. **Enter the protected edge:** The viewer reaches the platform through CloudFront and WAF.
2. **Validate API boundary:** API Gateway applies routing and throttling.
3. **Authenticate account:** The auth service verifies identity and device policy.
4. **Load account policy:** The service reads the account, subscription, and device state.
5. **Return account state:** The database returns only the fields needed for the decision.
6. **Issue short-lived session:** The client receives a short-lived token and refresh policy.

## 2. Search the catalog

- **What:** Search and rank titles that the viewer may access.
- **Why:** Full-text discovery has different performance and query requirements from transactional catalog storage.
- **How:** The EKS search service validates the request, queries OpenSearch, and applies entitlement and regional filtering before responding.

1. **Submit a query:** The client supplies the query and filters.
2. **Call search service:** The search service applies locale, availability, and safety filters.
3. **Query the index:** OpenSearch executes text and facet retrieval.
4. **Return ranked hits:** The index returns IDs, scores, and display metadata.
5. **Return entitled results:** The service removes unavailable results before returning them.

## 3. Publish a domain event

- **What:** Publish a domain event for independent background consumers.
- **Why:** Notifications and analytics must not increase playback-request latency or couple services together.
- **How:** An EKS service publishes a versioned event to Amazon MSK; autoscaled consumers process it and report lag and failures.

1. **Publish a domain event:** A service publishes an immutable event after committing business state.
2. **Deliver to consumers:** Independent consumers receive the event without blocking playback.
3. **Record processing outcome:** Consumers emit lag, error, and completion telemetry.

## 4. Load a runtime secret

- **What:** Load a protected runtime secret for one workload.
- **Why:** Applications need keys and credentials without embedding them in images, code, or node-wide configuration.
- **How:** The EKS workload uses its own IAM identity to request an encrypted secret governed by KMS and resource policy.

1. **Request protected secret:** The workload uses its own IAM identity rather than node-wide credentials.
2. **Return authorized value:** KMS and resource policy enforce the access decision.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
