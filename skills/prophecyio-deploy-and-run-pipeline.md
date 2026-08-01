---
name: Deploy a Prophecy project and run its pipeline
description: Deploy a Prophecy project to a fabric, trigger a pipeline run, poll its status, and run data tests — via the Prophecy REST API.
api: openapi/prophecyio-projects-openapi.json
operations: [deployProject, triggerPipelineRun, getPipelineRunStatus, runDataTests]
---

# Deploy a Prophecy project and run its pipeline

Use the Prophecy REST API to promote a project into an environment and execute it,
without relying on Prophecy's built-in scheduling. Good for external CI/CD.

## Auth
- Authenticate every request with a per-user **Personal Access Token (PAT)** as
  `Authorization: Bearer <token>` (see `authentication/prophecyio-authentication.yml`).
- Permissions are scoped to the token owner. Generate the token in the Prophecy UI:
  Settings > Access Tokens > Generate Token.
- Base URL is your Prophecy environment, e.g. `https://app.prophecy.io` for SaaS.

## Steps
1. **Deploy the project** — `deployProject` (`POST /api/deploy/project`). Supply the
   project id, target `fabricId`, and any per-environment pipeline/project configuration
   values. Use this to deploy the same project to dev vs prod with different config.
2. **Trigger a run** — `triggerPipelineRun` (`POST /api/trigger/pipeline`) against the
   deployed project's pipeline. Capture the returned `runId`.
3. **Poll status** — `getPipelineRunStatus` (`GET /api/trigger/pipeline/{runId}`) until the
   run reaches a terminal state. On failure the response includes an error message.
4. **Run data tests** — `runDataTests` (`POST /api/orchestration/tests/run`) to execute the
   project's existing data tests as part of the promotion gate.

## Conventions & errors
- Requests/responses are `application/json`; there is no idempotency-key contract, so make
  create/deploy calls once and check status rather than blindly retrying
  (`conventions/prophecyio-conventions.yml`).
- Handle `400` (bad config), `401` (bad/missing token), `403` (insufficient permission),
  `404` (unknown project/fabric/run), `500` (server error) — see
  `errors/prophecyio-problem-types.yml`.
