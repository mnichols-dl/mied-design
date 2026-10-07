# Reporting sequences: change log

File edited: design/working-docs/solution-areas/reporting/reporting-sequences.md. Diagrams were not re-rendered.

Counts: diagrams 6 before, 6 after. Splits 0. Missing operations 2, both outbound calls to Power BI.

## Sections changed

- **Conventions:** Actor is humans only, and participants include UI, services and external systems. Added the box grouping line, the APP / SVC / EXT / OUT tag line, and the standing authorization sentence.
- **All 6 diagrams:** Portal renamed to UI. Aliases standardized: ReportingApi, IamApi, OrgApi, PowerBi. Boxes added: Browser, MiEdWorkforce (AKS), External. Every request arrow tagged APP, SVC or OUT.
- **Embed token paths:** `{reportId}` changed to `{reportDefinitionId}`.
- **Organizations call:** now `SVC GET /organizations/{organizationCode}/hierarchy`, with the response shortened to "Ancestor chain".
- **Power BI:** now labeled "Power BI" (was "Power BI Service API").
- **IAM calls kept** as `SVC POST /permissions/check` in Browse, Happy Path, Access Denied and Parameter Resolution Failure. In each, the IAM result drives the flow.
- **Plain responses dropped:** "Report metadata confirmed" and the two "200 OK" returns in Register and Deactivate.
- **Who lines:** Browse: reporting.catalog.view. Embed token: reporting.embed-token.generate. Register and Deactivate: reporting.catalog.manage.

## Splits

None. Reporting has no row in sections 3.1, 3.2, 3.3 or 3.4, and the embed token flow was already split.

## Left alone

All text blocks, the Recovery text, and the self arrows with notes. No diagram has an alt or exceeds a size limit.

## Missing operations

| Area | Tag | Verb and path | Section |
| --- | --- | --- | --- |
| Power BI | OUT | POST /GenerateToken | Embed Token, Happy Path |
| Power BI | OUT | GET /reports/{reportId} | Register Report in Catalog |

All APP and SVC arrows match a spec operationId.

## API kind observations

- `getOrganizationHierarchy` is marked `x-access: internal-user, internal-service`, so it is both kinds. Its description says it is IAM only, restricted by Istio, and not for user-facing calls. The Happy Path has Reporting API calling it as SVC, which contradicts that.
- `checkPermissions` is internal-service and is drawn as SVC, which is consistent.
- All Reporting operations used are internal-user and drawn as APP, which is consistent.

## Questions

- Is Reporting API allowed to call `getOrganizationHierarchy`?
- What are the real Power BI paths? Real Power BI REST paths include the workspace id.
- Does embed-token return 403 or 404 for an unknown or unauthorized report? The doc says 403; the spec says 404.
- Should there be a ReportInactive variant? The spec denial enum has ReportInactive, but no diagram covers it.
- Does parameter resolution failure return 400 only, as the spec says, or 400/403, as Key Decisions says?
- Are getReport, adminListReports, adminGetReport, adminUpdateReport and adminListEmbedAudit intentionally not drawn?
- The optional `orgContext` override is not shown in any diagram.
- Should the 422 "report not found" path in Register be an alt? It is only in Error Scenarios.
