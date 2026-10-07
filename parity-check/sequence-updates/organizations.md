# Organizations sequences: change log

File edited: design/working-docs/solution-areas/organizations/organizations-sequences.md

Diagrams before: 6. After: 6. Splits: 0. Missing operations: 0.

## Conventions block
Actor is now humans only; participant covers services, components and external systems. Added the API-kind tag rule (APP, SVC, EXT, OUT), the standing sentence on IAM authorization, and the box grouping note.

## Sections changed
- CEPI Nightly Sync: Organizations Database participant removed; ETL writes shown as Synapse self arrows and one Note (self arrows cut from 7 to 4). Participants declared in boxes (Azure Platform, MiEdWorkforce (AKS), External). Label "CEPI CEDS JSON-LD API" shortened to "CEPI". Arrows tagged: OUT for the OAuth2 token request and GET /organizations, SVC for POST /admin/jobs/sync/complete. Token response and the sync-complete notification wording trimmed. Who line gained a Permission line (organizations.sync.trigger, manual trigger only). Prose unchanged.
- Organization Search: actor User kept, UI participant renamed "UI", database removed (self arrow), tag APP GET /organizations/search (query string dropped), response arrows merged so the two alt branches share one return to the UI. Permission line added (organizations.organization.view). The Redis cache stays because caching is the subject.
- Organization Detail Lookup: generic "Caller" replaced by UI with APP tag, plus a Note that internal services call the same operation as SVC. Path now uses {organizationCode} as in the spec. Database removed. Permission line added.
- Hierarchy Resolution: participants renamed to IamApi and OrgApi, database removed, arrow tagged SVC. Permission line added (organizations.hierarchy.view).
- Lead Admin Lookup: same treatment, SVC GET /organizations/by-lead-admin (email query parameter dropped). Permission line added (organizations.lead-admin.view).
- Degraded Operation (Sync Failure): database removed (self arrows), participants boxed, consumer call tagged SVC GET /organizations/{organizationCode}. No permission line (system process, not gated).

## Splits
None. Section 3.1 to 3.3 list no organizations rows; metrics are all within limits.

## Sections left alone
None unchanged. Key Decisions, State Changes, Events Published and Error Scenarios text blocks in all six sections are untouched. Events drawn once each and consumer note kept; no email sends.

## Missing operations
None. Every arrow matches an operationId in organizations-api.yml: searchOrganizations, getOrganization, getOrganizationHierarchy, getOrganizationsByLeadAdmin, completeSyncJob. CEPI calls are outbound to an external system and have no operation here.

Not drawn although in the spec: triggerOrgSync (Synapse also calls POST /admin/jobs/sync on scheduled runs, and the System Admin manual trigger); the doc text mentions it but no diagram draws it. getSyncLog and listOrganizationTypes have no sequence.

## API kind observations
- getOrganizationHierarchy and getOrganizationsByLeadAdmin carry x-access "internal-user, internal-service", but the sequences, the spec descriptions, the permissions doc and the Key Decisions say IAM service account only (Istio restricted). Sequences draw only SVC use. Suggest x-access internal-service.
- getOrganization and searchOrganizations: used by the UI (APP) and by services (SVC), consistent with "internal-user, internal-service". Detail diagram shows APP and notes the SVC use.
- triggerOrgSync and completeSyncJob have no x-access in the spec. The sequence uses completeSyncJob as SVC (Synapse managed identity, Istio restricted), which does not fit the Service API definition cleanly because Synapse runs outside the cluster. Suggest declaring x-access and confirming the mechanism.
- No operation is currently marked both external and application.

## Missing operations list (arrows with no operationId)
None.

## Questions
1. Source of org data: the sync diagram says "CEPI CEDS JSON-LD API"; the catalog treats EEM as the source (style guide 3.4 item 12). The capability doc says CEPI reflects EEM. Kept the label "CEPI"; confirm whether the External box should show EEM, CEPI or both.
2. Authentication to CEPI is drawn as a token request arrow with no response; confirm that the OUT token call should stay in the diagram.
3. The Synapse pipeline initiates the sync inside the API via POST /admin/jobs/sync per the spec, but the diagram starts with Synapse writing to the database. Should the opening call be drawn?
4. Azure Synapse and Azure Monitor are grouped in an extra box "Azure Platform" (the style guide defines only Browser, MiEdWorkforce (AKS), External). The Synapse to Monitor alert arrow is untagged because it fits none of APP, SVC, EXT, OUT. Confirm the box and tagging.
5. Redis cache arrows (GET, SET) are untagged because they are not API calls; the cache is kept as a participant because caching is the subject, per the style guide.
6. Lead Admin Lookup checks a cache keyed by organization code but the call is by email; the cache-hit path cannot be reached for an email lookup. Pre-existing; kept as is.
7. Degraded Operation is mostly a behavior description; consider whether it belongs under a sequence heading.
