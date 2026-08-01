---
name: Provision a Prophecy fabric with a connection and secret
description: Create a Prophecy fabric, add a data-source connection to it, and store the credential it needs as a secret — via the Prophecy REST API.
api: openapi/prophecyio-fabrics-openapi.json
operations: [createFabric, createSecret, createConnection, listConnectionsByFabric]
---

# Provision a Prophecy fabric with a connection and secret

Stand up an execution environment (fabric) and wire it to an external data source.

## Auth
- `Authorization: Bearer <PAT>` on every call (see
  `authentication/prophecyio-authentication.yml`); permissions follow the token owner.
- Base URL is your Prophecy environment, e.g. `https://app.prophecy.io`.

## Steps
1. **Create the fabric** — `createFabric` (`POST /api/orchestration/fabric`). Optionally
   attach a secret and connection inline. Capture the returned `fabricId`.
2. **Store the credential** — `createSecret`
   (`POST /api/orchestration/fabric/{fabricId}/secret`). Choose the secret kind that matches
   the source: `text`, `binary`, `username-password`, or `M2M OAuth`.
3. **Add the connection** — `createConnection`
   (`POST /api/orchestration/fabric/{fabricId}/connection`). Set the provider-specific
   properties (Snowflake, Databricks, BigQuery, S3, Postgres, Salesforce, …) and reference
   the secret created in step 2.
4. **Verify** — `listConnectionsByFabric`
   (`GET /api/orchestration/fabric/{fabricId}/connection`) to confirm the connection exists.

## Conventions & errors
- Resources are keyed by caller-supplied names/ids (`connectionName`, secret name), which
  gives natural upsert safety, but there is no `Idempotency-Key` header
  (`conventions/prophecyio-conventions.yml`).
- Handle `400`/`401`/`403`/`404`/`500` per `errors/prophecyio-problem-types.yml`.
