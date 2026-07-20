# Platform and Microservices

Inspect managed identity, private API integration, safe search, transactional outbox delivery, secrets, and observability.

[Open the interactive diagram](./02-platform-microservices.html) · [Download Draw.io](./drawio/02-platform-microservices.drawio) · [Back to suite](./index.html)

## Architecture decisions

- Primary identity is managed by Cognito or an external OIDC provider; Aurora stores subscriber and entitlement state, not passwords.
- The API path is WAF → API Gateway → VPC Link → internal load balancer → EKS; CloudFront is not an extra API hop.
- Aurora business state and the outbox record commit in one transaction; consumers are idempotent.
- OpenSearch is a rebuildable projection and never authorizes playback.

## Add only when

- Add ElastiCache only after measured Aurora or application latency identifies a hot read path.
- Replace EventBridge/SQS with MSK only when Kafka compatibility, partition ordering, replay volume, or existing Kafka operations justify it.
- Split the modular application when separate team ownership or scaling characteristics become real.

## Components

| Component | Technology | Scope |
|---|---|---|
| Client | Viewer application | HTTPS |
| Managed identity | Cognito or OIDC provider | OAuth 2.0 / OIDC |
| Protected API | WAF · API Gateway · VPC Link | JWT + throttle |
| OTT application | Internal ALB · EKS | modular deployables |
| Aurora + outbox | Accounts · catalog · rights | atomic source of truth |
| OpenSearch | Derived title index | rebuildable |
| EventBridge | Domain event routing | schema controlled |
| Consumer queues | Amazon SQS + DLQ | at-least-once |
| Background consumers | Notifications · indexing | idempotent |
| Secrets + KMS | Workload-scoped values | Pod Identity |
| Observability | Metrics · logs · traces | SLO + alarms |

## 1. Authenticate a viewer

- **What:** Authenticate a viewer and load the application policy needed for the session.
- **Why:** The platform should not own password storage or token cryptography when a managed OIDC provider can do it.
- **How:** The identity provider issues a short-lived token; API Gateway validates it before the application loads subscriber policy from Aurora.

1. **Authenticate with the identity provider:** The client completes the supported sign-in or federation journey outside the application database.
2. **Issue short-lived tokens:** The identity provider returns an access token and managed refresh policy.
3. **Call the protected API:** The client presents the access token; API Gateway validates issuer, audience, expiry, and scope.
4. **Forward verified claims:** Only verified identity claims cross the private integration.
5. **Load subscriber policy:** The application reads account, subscription, and device policy—not credentials.
6. **Return account state:** Aurora returns only the fields needed for the application decision.
7. **Return application session state:** The response contains application policy and no new identity system.

## 2. Search the entitled catalog

- **What:** Search and rank titles the viewer is allowed to discover.
- **Why:** Applying rights only after retrieval can leak counts and produce broken pagination.
- **How:** The application loads current rights, includes market and availability constraints in the OpenSearch query, then performs a final safety check.

1. **Submit a search query:** The client supplies text, locale, and filters.
2. **Call the application:** The protected API forwards verified account and market context.
3. **Load current entitlement scope:** Aurora returns the active subscription, market, and rights constraints.
4. **Return search constraints:** The application converts authoritative rights into safe query filters.
5. **Query the constrained index:** Region, rights window, maturity, and availability filters execute with the text query.
6. **Return ranked candidates:** The rebuildable index returns only candidates inside the supplied constraints.
7. **Return final entitled results:** The application performs a final rights check to tolerate stale index data.

## 3. Publish a reliable domain event

- **What:** Publish an event without losing consistency between Aurora and downstream consumers.
- **Why:** A database commit followed by a broker publish is an unsafe dual write.
- **How:** The application commits business state and an outbox row atomically; a relay publishes to EventBridge and each consumer receives its own SQS queue.

1. **Commit state and outbox atomically:** The business change and immutable event record either both commit or both roll back.
2. **Relay the committed event:** The relay reads only committed outbox rows and publishes a versioned event.
3. **Route to consumer queues:** Each independent consumer gets buffering, retries, and a dead-letter queue.
4. **Process idempotently:** The consumer records processed event IDs so duplicate delivery is harmless.
5. **Record the processing outcome:** Completion, age, retries, and DLQ depth are observable.

## 4. Load a runtime secret

- **What:** Load one protected runtime value for one workload.
- **Why:** Applications need keys and credentials without embedding them in images or node-wide configuration.
- **How:** The EKS workload uses Pod Identity to request an encrypted secret; access and failures are audited.

1. **Request the protected value:** The pod assumes its workload-scoped IAM role rather than node credentials.
2. **Return the authorized version:** Secrets Manager and KMS enforce resource and key policies.
3. **Record access outcome:** The application records success or failure without logging the value.

## Usage

Choose a scenario tab, then use **Next**, **Previous**, or **Play**. Click a node to jump to its first step. Drag nodes to refine the layout; press **R** to reset, **F** for fullscreen, and **T** to switch theme.
