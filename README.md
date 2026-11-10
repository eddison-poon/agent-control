# Agent Lifecycle & Change Control — Functional Test Plan

**Version:** 1.1 (Draft)  
**Date:** 2026-11-10  
**Status:** For QA and Product Owner review  
**Reference:** Agent Lifecycle & Change Control, Sections 1, 3–9 (Section 2 intentionally excluded)  
**Related Jira:** ETIVAI-19638 (Agent Change Control — Deployment to Production)

## 1. Objective and scope

Validate the complete Agent Lifecycle from Draft through Published, Pilot, Production and Archived, with emphasis on controlled promotion, environment readiness, production configuration immutability, supersession, soft deletion and governance traceability.

**Coverage weighting (planning guidance, not a fixed case quota):** BS-01–03 approximately 20% of test-design effort; BS-04–11 approximately 80%. The release-gating rules in BS-02 remain mandatory despite lighter foundation coverage.

**In scope:** Agent Builder and Marketplace in non-production and production; Agent Ops API; Agent Gateway; release request and approval flows; Released-track status; monitoring/audit evidence; permissions and credential provisioning.

**Out of scope:** Phased rollout schedule (Section 2); building or enabling Patterns/Tenants, enabling MCP/Skills in production (separate processes); automatic synchronization of space structure (explicitly not supported); full agent decommissioning beyond specified Draft-v1 soft-delete; performance/load testing.

## 2. Test design and conventions

- **Business Scenario (BS):** End-to-end business behavior or control objective.
- **Test Definition (TD):** A testable condition or rule within a scenario.
- **Test Case (TC):** An executable verification with steps and expected result.
- **Priority:** P0 = release-blocking integrity/security/control; P1 = core functionality; P2 = secondary/observability.
- **Type:** Positive, Negative, Boundary, RBAC, Integration, Audit, Regression.
- **Requirement references:** Sx denotes source section; CC-1/2/3 denotes Section 7 requirement sequence; DEL denotes Section 8; GOV denotes Section 9.
- **Expected results marked `TBD`:** Clarification is required before treating the case as an acceptance gate.
- **Test data:** Use synthetic agents, tenants, spaces, patterns and credentials. Record actual Agent ID, version, UUID, release request ID and deployment instance identifiers in execution evidence.

### Lifecycle invariants

1. Agent ID remains stable across versions; each version has a distinct UUID.
2. Editing a Draft does not change its version number.
3. Publishing a Draft makes it the single non-production Published version and creates the next Draft.
4. Published/Pilot/Production configurations are immutable; reverting requires a new Draft-based release.
5. At most one non-production Published assignment and at most one production-side Pilot **or** Production assignment exist per Agent ID.
6. Superseded released versions are Archived and removed from user-facing Builder/Marketplace views, while monitoring/audit trace remains.
7. Production promotion requires an approved Published version, readiness review, controlled Pilot deployment and successful Pilot assessment.
8. Draft-v1 soft-delete is a separate Archive path from released-version supersession.

### Execution approach

- **Smoke:** BS-01–03 happy path plus one complete Draft → Published → Pilot → Production path.
- **Functional:** All cases below, with P0 first and high-risk negative paths before P2.
- **Integration:** Verify UI, API, Released-track, Gateway and Marketplace consistency after each transition.
- **Regression:** Re-run single-assignment, immutability, historical trace and delete restrictions after every deployment change.

## 3. Coverage index

| Scenario | Business Scenario | Definitions | Cases | Focus |
|---|---|---:|---:|---|
| BS-01 | Agent Creation & Draft Management | 2 | 4 | Basic |
| BS-02 | Publishing & Version Management | 3 | 7 | Basic + critical controls |
| BS-03 | Non-Production Testing | 2 | 3 | Basic |
| BS-04 | Production Readiness & Release Request | 3 | 9 | Extensive |
| BS-05 | Approval & Change Control | 3 | 7 | Extensive |
| BS-06 | Pilot Deployment & Testing | 3 | 8 | Extensive |
| BS-07 | Production Promotion & Consumption | 3 | 7 | Extensive |
| BS-08 | Version Replacement & Archiving | 3 | 8 | Extensive |
| BS-09 | Production Configuration Protection | 3 | 8 | Extensive |
| BS-10 | Draft Agent Soft Delete | 3 | 8 | Extensive |
| BS-11 | Governance Evidence & Traceability | 3 | 7 | Extensive |
| **Total** | | **31** | **76** | |

## 4. Business Scenarios → Test Definitions → Test Cases

### BS-01 — Agent Creation & Draft Management
**Business objective:** Creators can establish and refine a working Agent without altering released functionality. **References:** S3, S4, S5, S6.

#### TD-01-01 — Create and identify a Draft

**TC-01-01-01 — Create a new Agent** | P1 | Positive  
Preconditions: Authorized Creator has access to non-production Agent Builder.  
Steps: (1) Create an Agent with valid configuration. (2) Save. (3) Inspect Agent Builder and metadata.  
Expected: Agent appears as Draft v1 with a stable Agent ID and version-specific UUID; it is not consumer-facing. [S5, S6]

**TC-01-01-02 — Verify Draft visibility** | P1 | RBAC  
Preconditions: Draft v1 exists; Creator and Consumer accounts available.  
Steps: (1) Open non-production Builder as Creator. (2) Search non-production Marketplace as Consumer.  
Expected: Draft is available in Builder but not consumer-facing Marketplace. [S5]

#### TD-01-02 — Refine Draft without release impact

**TC-01-02-01 — Update Draft configuration** | P1 | Positive  
Preconditions: Draft v1 exists.  
Steps: (1) Change instructions/tools. (2) Save. (3) Reopen version metadata.  
Expected: Changes persist in Draft; version remains v1. [S6]

**TC-01-02-02 — Modify a later Draft while release remains active** | P1 | Regression  
Preconditions: Published v1 and Draft v2 exist.  
Steps: (1) Update Draft v2. (2) Run/read Published v1.  
Expected: Published v1 configuration and behavior are unchanged. [S1, S4]

### BS-02 — Publishing & Version Management
**Business objective:** Publishing creates immutable, uniquely identified release versions and advances Draft iteration. **References:** S1, S3–S6.

#### TD-02-01 — Publish a validated Draft

**TC-02-01-01 — Publish Draft v1** | P0 | Positive  
Preconditions: Draft v1 exists and required unit tests have passed.  
Steps: (1) Publish Draft. (2) Inspect non-production Marketplace and version metadata.  
Expected: v1 becomes Published and is the current non-production Marketplace release. [S3, S5]

**TC-02-01-02 — Auto-create next Draft** | P1 | Positive  
Preconditions: TC-02-01-01 completed.  
Steps: (1) Open Agent Builder. (2) Inspect current working version.  
Expected: New Draft v2 exists under the same Agent ID with a different UUID. [S4, S5]

**TC-02-01-03 — Enforce publish testing gate** | P0 | Negative  
Preconditions: Draft fails mandatory unit-test gate.  
Steps: Attempt Publish.  
Expected: Publish is not completed; Draft remains editable. Exact failure UI/API response TBD. [S3]

#### TD-02-02 — Ensure version identity and immutability

**TC-02-02-01 — Validate identifiers across versions** | P0 | Integration  
Preconditions: v1 Published and v2 Draft exist.  
Steps: Compare Agent ID, version and UUID metadata.  
Expected: Same Agent ID, distinct version numbers and distinct UUIDs. [S4, S5]

**TC-02-02-02 — Reject modification to Published version** | P0 | Negative  
Preconditions: Published v1 exists.  
Steps: Attempt to edit published configuration via available UI and authorized API.  
Expected: Published configuration cannot be changed. [S4, S5]

#### TD-02-03 — Publish successor version

**TC-02-03-01 — Publish v2 and supersede v1** | P0 | Integration  
Preconditions: Published v1 and tested Draft v2 exist.  
Steps: Publish v2; inspect Marketplace and historical records.  
Expected: v2 is the sole current non-production Published version; former Published assignment v1 is Archived and not overwritten; Draft v3 is created. [S4]

**TC-02-03-02 — Revert through a new release** | P1 | Negative/Regression  
Preconditions: A published configuration needs to be restored.  
Steps: Create/update a Draft with the desired earlier configuration and publish through normal gates.  
Expected: Earlier immutable version is not directly edited or reactivated; a new version is released. [S4]

### BS-03 — Non-Production Testing
**Business objective:** Consumers validate a Published release before production request. **References:** S3, S5, S6.

#### TD-03-01 — Access and execute Published version

**TC-03-01-01 — Consumer accesses Published Agent** | P1 | Positive  
Preconditions: Published v1 available in non-production Marketplace.  
Steps: Sign in as authorized non-production Consumer; locate and execute Agent.  
Expected: Consumer can access and test the Published release. [S5, S6]

**TC-03-01-02 — Verify testing does not mutate release** | P1 | Regression  
Preconditions: Published version accessible.  
Steps: Run test requests; compare status/version/configuration before and after.  
Expected: Status, version and released configuration are unchanged. [S6]

#### TD-03-02 — Handle unsuccessful consumer testing

**TC-03-02-01 — Remediate failed non-production testing** | P1 | Negative  
Preconditions: Consumer reports defect in Published v1; Draft v2 exists.  
Steps: Record failure; update Draft v2; retest and publish when eligible.  
Expected: Remediation occurs in Draft, not by editing Published v1; subsequent release follows normal gates. [S5, S6]

### BS-04 — Production Readiness & Release Request
**Business objective:** Release requests expose the real production availability of Pattern, Tenant and Space and prevent approval when dependencies are absent. **References:** S6, S7 CC-2, S7 assumptions.

#### TD-04-01 — Display dependency readiness

**TC-04-01-01 — All dependencies available** | P0 | Positive  
Preconditions: Required Pattern, Tenant and Space exist in production.  
Steps: Open Release Request readiness tab.  
Expected: All three dependencies show available; readiness does not block approval. [CC-2]

**TC-04-01-02 — Pattern missing** | P0 | Negative  
Preconditions: Pattern absent from production.  
Steps: Open readiness tab and attempt approval.  
Expected: Pattern unavailable is shown; approval is not allowed; Tenant Owner is reminded to contact Pattern Owner. [CC-2]

**TC-04-01-03 — Tenant missing** | P0 | Negative  
Preconditions: Tenant absent from production.  
Steps: Open readiness tab and attempt approval.  
Expected: Tenant unavailable is shown; approval is blocked; link to production Tenant Creation Request is presented. [CC-2]

**TC-04-01-04 — Only Space missing** | P0 | Negative  
Preconditions: Pattern and Tenant exist; Space absent.  
Steps: Open readiness tab and attempt approval.  
Expected: Approval blocked; link to create Space under production Tenant is presented. [CC-2]

**TC-04-01-05 — Multiple dependencies missing** | P1 | Boundary  
Preconditions: Pattern, Tenant and/or Space missing in multiple combinations.  
Steps: Open readiness and inspect each dependency.  
Expected: Every missing dependency is identified; approval blocked. Exact combination-specific remediation precedence TBD. [CC-2]

#### TD-04-02 — Refresh and API consistency

**TC-04-02-01 — Refresh after dependency becomes available** | P1 | Positive  
Preconditions: Initial readiness shows missing Space; Space subsequently created by separate process.  
Steps: Click Refresh.  
Expected: Readiness updates to latest production availability. [CC-2]

**TC-04-02-02 — Refresh after dependency becomes unavailable** | P0 | Negative  
Preconditions: Initial readiness all available; a dependency becomes unavailable.  
Steps: Click Refresh.  
Expected: Readiness reflects missing dependency and approval is blocked. [CC-2]

**TC-04-02-03 — UI and Agent Ops API readiness agree** | P1 | Integration  
Preconditions: Known production dependency state.  
Steps: Compare readiness tab with Agent Ops API check using Pattern/Tenant/Space names.  
Expected: Same availability conclusions for each dependency. [CC-2]

#### TD-04-03 — Request without status mutation

**TC-04-03-01 — Submit deployment request** | P0 | Positive  
Preconditions: Tested Published version exists.  
Steps: Creator submits production deployment request; inspect version metadata.  
Expected: Request is recorded; Published status and version number do not change merely because of request submission. [S6]

### BS-05 — Approval & Change Control
**Business objective:** Tenant Owner approval and policy/readiness gates control eligibility for production Pilot deployment. **References:** S3, S6, S7 CC-1/2.

#### TD-05-01 — Approval authorization and evidence

**TC-05-01-01 — Tenant Owner approves eligible request** | P0 | Positive  
Preconditions: Published version, successful non-production tests, completed policy/control evidence and ready production dependencies.  
Steps: Tenant Owner reviews and approves request.  
Expected: Approved Published version becomes eligible for Pilot deployment; version/status do not change at approval. [S6]

**TC-05-01-02 — Reject request with failed policy review** | P0 | Negative  
Preconditions: Policy/control review not passed.  
Steps: Attempt to approve/deploy.  
Expected: Pilot deployment does not proceed; remediation returns to Creator via Draft. Exact rejection status TBD. [S3, S5, S6]

**TC-05-01-03 — Unauthorized user attempts approval** | P0 | RBAC  
Preconditions: Non-Tenant-Owner account.  
Steps: Attempt Tenant Owner approval action through UI/API.  
Expected: User cannot approve request. [S6 role responsibility]

#### TD-05-02 — Block bypass of approval gates

**TC-05-02-01 — Attempt Pilot deployment without approval** | P0 | Negative/Integration  
Preconditions: Published version without approved deployment request.  
Steps: Invoke deployment path.  
Expected: Pilot promotion not permitted. [S3, S4, S6]

**TC-05-02-02 — Attempt approval with missing dependency** | P0 | Negative  
Preconditions: Any required production Pattern/Tenant/Space unavailable.  
Steps: Tenant Owner attempts approval.  
Expected: Approval prevented. [CC-2]

#### TD-05-03 — Preserve release request integrity

**TC-05-03-01 — Approved request targets intended version** | P0 | Integration  
Preconditions: Approved request for Published v1; Draft v2 exists.  
Steps: Inspect approval record and deploy approved candidate.  
Expected: Approval/deployment refers to intended Published version, not latest Draft; Agent ID/version trace is consistent. [S4, S9]

**TC-05-03-02 — Recheck stale readiness at final approval** | P1 | Proposed/TBD  
Preconditions: Dependencies available when form loaded but absent by approval time.  
Steps: Attempt approval without Refresh.  
Expected: **TBD:** Requirements specify Refresh and blocking when unavailable, but do not explicitly define server-side revalidation timing. Confirm required behavior. [CC-2]

### BS-06 — Pilot Deployment & Testing
**Business objective:** Approved Published version is deployed into production for limited Pilot validation. **References:** S3–S7 CC-1.

#### TD-06-01 — Deploy approved release to Pilot

**TC-06-01-01 — Deploy Published version to production Pilot** | P0 | Positive/Integration  
Preconditions: Approved deployment request; required dependencies ready.  
Steps: AH Engineer triggers Agent Ops deployment.  
Expected: Approved Published release is cloned/deployed to production, Released-track becomes Pilot, and Agent appears in production Marketplace for its Space. [CC-1, S6]

**TC-06-01-02 — Preserve Agent ID and version on deployment** | P0 | Integration  
Preconditions: Pilot deployment completed.  
Steps: Compare non-production source with production release metadata.  
Expected: Agent ID and version number are unchanged; deployment UUID semantics require clarification (see open questions). [CC-1]

**TC-06-01-03 — Verify Pilot audience restriction** | P0 | RBAC  
Preconditions: Pilot release deployed; pilot-authorized and general consumer accounts.  
Steps: Attempt to locate and execute Pilot as each account.  
Expected: Designated Pilot users can use Pilot; normal production consumers without Pilot entitlement cannot use it. [S5]

#### TD-06-02 — Pilot execution and control

**TC-06-02-01 — Execute Pilot test without changing status** | P1 | Positive  
Preconditions: Pilot deployed and accessible to pilot-authorized Consumer.  
Steps: Execute Pilot tests; inspect status/version afterward.  
Expected: Pilot remains Pilot while evidence is collected; version unchanged. [S6]

**TC-06-02-02 — Block direct promotion without Pilot success** | P0 | Negative  
Preconditions: Pilot testing not passed.  
Steps: Attempt Production status assignment.  
Expected: Production status not assigned. [S3, S5, S6]

**TC-06-02-03 — Remediate failed Pilot through Draft** | P0 | Negative  
Preconditions: Pilot fails acceptance.  
Steps: Record failure; create/update non-production Draft and initiate new release cycle.  
Expected: Failed Pilot version is not edited; remediation uses a new Draft version. Handling of failed Pilot assignment before replacement TBD. [S5, S6]

#### TD-06-03 — Handle deployment errors

**TC-06-03-01 — Deployment API failure does not create false Pilot success** | P0 | Proposed/TBD  
Preconditions: Controlled Agent Ops deployment failure.  
Steps: Trigger deployment failure; inspect Released-track, Marketplace and records.  
Expected: **TBD:** No inconsistent successful Pilot assignment should be reported; rollback/compensation semantics not specified. [CC-1]

**TC-06-03-02 — Retry after interrupted deployment** | P1 | Proposed/TBD  
Preconditions: Interrupted deployment with uncertain completion.  
Steps: Retry or reconcile deployment; inspect instances and assignments.  
Expected: **TBD:** Define idempotency and recovery behavior; at most one production-side assignment remains an invariant. [S4, CC-1]

### BS-07 — Production Promotion & Consumption
**Business objective:** Successful Pilot becomes the single normal Production release only after controlled decision. **References:** S3–S6.

#### TD-07-01 — Request and approve Pilot-to-Production

**TC-07-01-01 — Creator requests Production status** | P1 | Positive  
Preconditions: Pilot deployed and tested.  
Steps: Creator submits Production status request.  
Expected: Request is recorded; Pilot status/version remain unchanged pending decision. [S6]

**TC-07-01-02 — Tenant Owner confirms successful Pilot** | P0 | Positive  
Preconditions: Successful Pilot evidence available.  
Steps: Tenant Owner confirms outcome.  
Expected: Selected Pilot release is assigned Production; same Agent version. [S5, S6]

**TC-07-01-03 — Reject Production promotion after unsuccessful Pilot** | P0 | Negative  
Preconditions: Pilot evidence shows failure.  
Steps: Attempt Production confirmation.  
Expected: No Production assignment; Creator must remediate through new Draft. [S5, S6]

#### TD-07-02 — Enforce Production access and single assignment

**TC-07-02-01 — Normal Consumer uses Production release** | P0 | Positive  
Preconditions: Production assignment active.  
Steps: Sign in as authorized normal production Consumer; execute Agent.  
Expected: Production Agent is visible/usable in production Marketplace. [S5, S6]

**TC-07-02-02 — Enforce only one Pilot or Production assignment** | P0 | Negative/Integration  
Preconditions: Production assignment exists.  
Steps: Attempt second simultaneous production-side assignment outside controlled replacement.  
Expected: No two concurrent Pilot/Production assignments for the same Agent ID. [S4, S5]

#### TD-07-03 — Preserve version integrity during promotion

**TC-07-03-01 — Promotion does not edit or increment version** | P0 | Integration  
Preconditions: Pilot v1 with known configuration.  
Steps: Promote to Production; compare version/configuration.  
Expected: Version number and configuration unchanged. [S4–S6]

**TC-07-03-02 — Unauthorized Production confirmation** | P0 | RBAC  
Preconditions: Non-Tenant-Owner user.  
Steps: Attempt to confirm Pilot outcome via UI/API.  
Expected: User cannot perform Tenant Owner approval action. [S6]

### BS-08 — Version Replacement & Archiving
**Business objective:** Successor versions replace active assignments without overwriting historical releases. **References:** S1, S3–S5, S7 CC-1.

#### TD-08-01 — Supersede Published version

**TC-08-01-01 — Archive previous Published assignment** | P0 | Integration  
Preconditions: Published v1 and tested Draft v2.  
Steps: Publish v2; inspect previous Published record.  
Expected: v1 released assignment becomes Archived, v2 becomes sole non-production Published assignment. [S4]

**TC-08-01-02 — Keep production release independent of Draft updates** | P0 | Regression  
Preconditions: Production v1; non-production Draft v2.  
Steps: Change Draft v2; execute Production v1.  
Expected: Production v1 remains unchanged and usable. [S1, S4]

#### TD-08-02 — Replace production-side assignment

**TC-08-02-01 — Promote newer approved version over existing Production** | P0 | Integration  
Preconditions: Production v1, Published v2 with approved Pilot deployment request.  
Steps: Deploy v2 to Pilot via controlled promotion.  
Expected: Existing production-side assignment v1 is Archived and v2 becomes sole Pilot assignment. [S4, CC-1]

**TC-08-02-02 — Prevent two production-side assignments during replacement** | P0 | Negative  
Preconditions: Replacement underway.  
Steps: Inspect Released-track and Marketplace assignments across transition.  
Expected: No simultaneously active Pilot and Production assignments for the Agent ID. [S4, S5]

**TC-08-02-03 — Archive replaced Pilot** | P0 | Integration  
Preconditions: Pilot v1 exists; approved successor v2 ready.  
Steps: Deploy successor Pilot.  
Expected: Previous Pilot released assignment is Archived, successor becomes Pilot. [S4, CC-1]

#### TD-08-03 — Retain history and restrict restoration

**TC-08-03-01 — Hide Archived released versions** | P1 | RBAC  
Preconditions: Superseded version Archived.  
Steps: Search Agent Builder and both Marketplaces.  
Expected: Archived released version is not user-facing. [S4, S5]

**TC-08-03-02 — Retain historical UUID and traces** | P0 | Audit  
Preconditions: Archived version with historical executions.  
Steps: Inspect monitoring/audit records.  
Expected: Historical version UUID and execution traces remain accessible to authorized monitoring/audit users. [S1, S4, S5]

**TC-08-03-03 — Prevent direct restoration of Archived release** | P0 | Negative  
Preconditions: Archived released version.  
Steps: Attempt to assign it directly as current Published/Pilot/Production.  
Expected: Cannot restore without a new controlled release. [S4, S5]

### BS-09 — Production Configuration Protection
**Business objective:** No user may edit Pilot/Production configuration; permitted credential provisioning remains available. **References:** S4, S5, S7 CC-3.

#### TD-09-01 — Lock production configuration

**TC-09-01-01 — Block Agent information changes** | P0 | Negative/RBAC  
Preconditions: Pilot and Production releases.  
Steps: As Creator, Tenant Owner and Consumer, attempt to edit Agent information in UI/API.  
Expected: No Pilot/Production Agent information is modified. [CC-3]

**TC-09-01-02 — Block instruction changes** | P0 | Negative  
Preconditions: Pilot/Production release.  
Steps: Attempt to change instructions.  
Expected: Configuration unchanged. [CC-3]

**TC-09-01-03 — Block advanced setting changes** | P0 | Negative  
Preconditions: Pilot/Production release.  
Steps: Attempt to change advanced settings.  
Expected: Configuration unchanged. [CC-3]

#### TD-09-02 — Lock attached capabilities

**TC-09-02-01 — Block add/modify/remove Skills** | P0 | Negative  
Preconditions: Pilot/Production release.  
Steps: Attempt each Skill mutation.  
Expected: All mutations denied; existing Skills unchanged. [CC-3]

**TC-09-02-02 — Block add/modify/remove Connectors** | P0 | Negative  
Preconditions: Pilot/Production release.  
Steps: Attempt each Connector mutation.  
Expected: All mutations denied; existing Connectors unchanged. [CC-3]

**TC-09-02-03 — Block add/modify/remove Knowledge** | P0 | Negative  
Preconditions: Pilot/Production release.  
Steps: Attempt each Knowledge mutation.  
Expected: All mutations denied; existing Knowledge unchanged. [CC-3]

#### TD-09-03 — Permit credentials without configuration edits

**TC-09-03-01 — Creator provides runtime credentials** | P1 | Positive  
Preconditions: Pilot/Production release requires credentials.  
Steps: Creator provides credentials through supported flow; execute Agent.  
Expected: Credential provisioning is allowed; immutable Agent configuration unchanged. [CC-3]

**TC-09-03-02 — Space Consumer provides runtime credentials** | P1 | Positive/RBAC  
Preconditions: Authorized Space Consumer and credential-requiring Pilot/Production Agent.  
Steps: Consumer provides credentials; execute Agent.  
Expected: Credential provisioning allowed for eligible Consumer without modifying released configuration. [CC-3]

### BS-10 — Draft Agent Soft Delete
**Business objective:** Mistakenly created Draft-v1 Agents may be soft-deleted without removing monitoring history or allowing Gateway use. **References:** S8.

#### TD-10-01 — Enforce delete eligibility

**TC-10-01-01 — Soft-delete Draft v1** | P0 | Positive  
Preconditions: Agent has only Draft v1 and no Published history.  
Steps: Use Delete in Agent Studio/Builder.  
Expected: Released-track becomes Archived and Agent disappears from Agent Builder. [DEL]

**TC-10-01-02 — Disable Delete after publication** | P0 | Negative  
Preconditions: Agent has Published v1 and Draft v2.  
Steps: Inspect Delete action for Agent.  
Expected: Delete button disabled because Agent has a Published version. [DEL]

**TC-10-01-03 — Reject delete of non-Draft Agent** | P0 | Negative  
Preconditions: Published, Pilot or Production Agent.  
Steps: Attempt Draft-delete operation via UI/API.  
Expected: Draft-delete operation not allowed; normal supersession archiving remains separate. [DEL]

#### TD-10-02 — Preserve monitoring, block runtime use

**TC-10-02-01 — Hide deleted Agent from Builder** | P1 | Positive  
Preconditions: Draft v1 soft-deleted.  
Steps: Search Agent Builder.  
Expected: Deleted Agent not shown. [DEL]

**TC-10-02-02 — Retain deleted Agent in Monitoring** | P1 | Audit  
Preconditions: Draft v1 soft-deleted.  
Steps: Search Monitoring Dashboard.  
Expected: Deleted Agent record remains discoverable. [DEL]

**TC-10-02-03 — Block Agent Gateway access to Archived Agent** | P0 | Integration  
Preconditions: Released-track is Archived following Draft delete.  
Steps: Attempt Agent invocation through Agent Gateway.  
Expected: Archived Agent cannot be reached via Gateway. [DEL; proposed API/Gateway enhancement]

#### TD-10-03 — Enforce transition constraints

**TC-10-03-01 — Reject unauthorized Draft-to-Archived transition** | P0 | Negative  
Preconditions: Agent status is not Draft.  
Steps: Attempt to use Draft-delete API path to set Released-track to Archived.  
Expected: Draft-delete transition rejected; supersession/archive process is unaffected. [DEL; proposed enhancement]

**TC-10-03-02 — Prevent use of deleted Agent after UI refresh** | P1 | Regression  
Preconditions: Draft v1 deleted.  
Steps: Refresh Builder, search and attempt previously saved direct Agent URL.  
Expected: Deleted Agent is not available for ordinary Builder use. Exact direct-link response TBD. [DEL]

### BS-11 — Governance Evidence & Traceability
**Business objective:** Each promotion is auditable from request and approvals through the exact deployed version and production execution. **References:** S1, S4, S9.

#### TD-11-01 — Retain promotion decisions and evidence

**TC-11-01-01 — Retain release request and approval** | P0 | Audit  
Preconditions: Completed Pilot deployment.  
Steps: Retrieve release record.  
Expected: Release request and approval decision retained and associated with the promotion. [GOV]

**TC-11-01-02 — Retain policy and non-production testing evidence** | P0 | Audit  
Preconditions: Completed Pilot deployment.  
Steps: Retrieve control review and non-production testing records.  
Expected: Policy/control review outcome and non-production test evidence retained. [GOV]

**TC-11-01-03 — Retain Pilot evidence and production decision** | P0 | Audit  
Preconditions: Pilot promoted to Production.  
Steps: Retrieve Pilot test evidence and final production decision.  
Expected: Pilot evidence and production decision retained. [GOV]

#### TD-11-02 — Trace exact release identity

**TC-11-02-01 — Match Agent ID, version and UUID across records** | P0 | Integration/Audit  
Preconditions: Promoted version and monitoring records exist.  
Steps: Compare release, deployment and monitoring records.  
Expected: Agent ID, version and version UUID consistently identify the intended release. [GOV]

**TC-11-02-02 — Trace production execution to deployed version** | P0 | Audit  
Preconditions: At least two historical versions with execution traces.  
Steps: Select a production trace and resolve associated version.  
Expected: Trace maps to exact deployed version/UUID, not merely Agent ID. [S1, GOV]

#### TD-11-03 — Preserve history across lifecycle transitions

**TC-11-03-01 — Historical evidence survives supersession** | P0 | Audit/Regression  
Preconditions: v1 released and executed; v2 replaces v1.  
Steps: Retrieve v1 request, approval, execution and audit records.  
Expected: Evidence remains accessible after v1 Archived. [S1, S4, GOV]

**TC-11-03-02 — Identify missing required evidence** | P1 | Proposed/TBD  
Preconditions: Promotion record intentionally missing an evidence item.  
Steps: Inspect release governance controls and attempt advancement.  
Expected: **TBD:** Evidence must be retained, but whether missing evidence automatically blocks advancement is not specified for every evidence type. [GOV]

## 5. End-to-end integration journeys

| Journey | Sequence | Core assertions |
|---|---|---|
| E2E-01 — First production release | BS-01 → 02 → 03 → 04 → 05 → 06 → 07 → 11 | Correct status transitions, gates, audience, immutable identity, evidence |
| E2E-02 — Subsequent release | Production v1 + Draft v2 → Publish v2 → Approve → Pilot v2 → Production v2 → Archive v1 | Single production-side assignment, no overwrite, trace retained |
| E2E-03 — Failed non-prod testing | Published v1 fails → update Draft v2 → publish successor | Released version unchanged until controlled successor publication |
| E2E-04 — Failed Pilot | Pilot fails → no Production promotion → new Draft remediation | Failed release cannot bypass gates |
| E2E-05 — Delete mistaken Agent | Draft v1 → Delete → Archived → Gateway denied + Monitoring retained | Soft-delete integrity and retention |

## 6. Test data and environment prerequisites

| Dataset | Purpose |
|---|---|
| AG-A: New Agent, Draft v1 only | Create/edit/delete eligibility |
| AG-B: Published v1 + Draft v2 | Publish, immutability, non-prod testing |
| AG-C: Production v1 + successor Published v2 | Replacement, archive, regression |
| AG-D: Pilot version with failed testing | Failed Pilot handling |
| ENV-R1: Pattern/Tenant/Space all available | Positive readiness |
| ENV-R2: Pattern absent | Pattern block/remediation |
| ENV-R3: Tenant absent | Tenant block/remediation |
| ENV-R4: Space absent | Space block/remediation |
| ENV-R5: Multiple dependencies absent | Combined error handling |
| Roles: Creator, Non-Prod Consumer, Pilot Consumer, General Prod Consumer, Tenant Owner, AH Engineer | Access and approval boundaries |

Required setup: access to non-production Agent Builder and Marketplace, production Marketplace, release request UI, Agent Ops API, Agent Gateway and monitoring/audit data. Confirm availability of a safe isolated production-like validation space before destructive/replacement tests. Use non-sensitive synthetic credentials.

## 7. Entry and exit criteria

**Entry:** Requirements reviewed; test users/roles provisioned; environments and dependencies available; release/deployment APIs reachable; synthetic Agent test data prepared; outstanding TBD behaviors classified; logging/monitoring access granted.

**Exit recommendation:** All P0 cases executed and passed; no unresolved Critical/High defect affecting production promotion, configuration integrity, access or traceability; P1 exceptions reviewed and accepted by accountable owner; evidence linked to executed cases; unresolved requirement ambiguities documented. Numeric pass-rate thresholds are not specified by source and must be agreed before release gating.

## 8. Risks, assumptions and open questions

| ID | Question / gap | Why it matters | Proposed owner |
|---|---|---|---|
| Q-01 | Does production cloning create a new deployment UUID while retaining the version UUID, or retain the exact same UUID? | Traceability and identifier assertions | Architecture / Agent Ops |
| Q-02 | When Published v2 replaces Published v1 in non-production while v1 is Production, is the v1 Production release unaffected (as expected)? How are separate release assignments represented? | Prevent false Archive interpretation | Product Owner / Agent Ops |
| Q-03 | When deploying successor Pilot over existing Production, is old Production archived immediately? Is any no-downtime handover/rollback guarantee required? | Release availability risk | Product Owner / Operations |
| Q-04 | What happens to a failed Pilot assignment before a replacement is deployed? | Status and remediation test oracle | Product Owner |
| Q-05 | Must readiness be revalidated server-side at approval/deployment time, beyond the Refresh button? | Time-of-check/time-of-use gap | Product Owner / API |
| Q-06 | What is the exact approval and evidence record lifecycle (request statuses, rejection, resubmission)? | Precise negative-path expectations | Product Owner |
| Q-07 | Which Pilot user entitlement model governs Marketplace visibility and Gateway execution? | RBAC test setup | Security / Product Owner |
| Q-08 | Are Draft-delete Agent Ops/Gateway enhancements in current release scope? | Proposed vs committed functionality | Product Owner |
| Q-09 | What is the expected response to deployment partial failure, retry and duplicate requests? | Consistency/recovery assertions | Agent Ops |
| Q-10 | Does the immutable configuration rule cover all write APIs and indirect changes? How are credentials stored/scoped? | Security and mutation coverage | Security / Agent Ops |
| Q-11 | Which evidence fields are mandatory and which missing records block a promotion? | Governance gate design | Governance / Product Owner |
| Q-12 | What are the explicit publish unit-test and non-prod testing pass criteria? | Deterministic publish/deploy gates | QA / Product Owner |

**Source assumptions (Section 7):** Pattern, Tenant and Space must already exist in production; Pattern/Tenant enablement and MCP/Skills enablement use separate processes; space structures are not automatically synchronized across environments.

**Archiving Rules**

Agent versions can be archived through two different operations:

1. **Draft Deletion (BS-10):** A user can delete an agent only when it is in Draft status and has never been published (Version 1).
2. **Version Replacement (BS-08):** The system automatically archives an existing Published, Pilot, or Production version when it is superseded by a newer version.

These operations must be tested separately because they have different business rules and triggers.

## 9. Execution record template

For execution tracking, capture: Test Case ID; environment; build; Agent ID; version; UUID; request ID; executor; execution date; status (Not Run/Passed/Failed/Blocked); actual result; evidence link; defect ID; comments. Keep test definitions stable in Git and execution results in a separate tracker or test-management tool.

## 10. Repository layout recommendation

```text
agent-lifecycle-change-control/
  README.md                       # This test plan, scope and coverage
  docs/
    requirement-clarifications.md # Resolved questions and decisions
  execution/
    README.md                     # Execution evidence/linking convention
```

The initial v1.1 can be committed as a **single Markdown document** to minimize maintenance. Split into per-scenario Markdown files only if reviews or test ownership require it.
