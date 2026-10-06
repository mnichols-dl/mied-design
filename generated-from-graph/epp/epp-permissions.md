# Education Preparation Provider (EPP) - Permissions Catalog

This document defines all atomic permissions for the Education Preparation Provider (EPP) domain.

---

## Permissions

| Category                          | Permission ID                    | Description                                       | Applicable Scopes         | Notes                                                             |
| --------------------------------- | -------------------------------- | ------------------------------------------------- | ------------------------- | ----------------------------------------------------------------- |
| EPP Provider Management           | `epp.provider.view`              | View EPP provider details and configuration       | System-wide, EPP-specific | Allows viewing EPP master data, program approvals, contacts       |
| EPP Provider Management           | `epp.provider.create`            | Designate EEM organization as EPP                 | System-wide               | Requires organization to exist in EEM first                       |
| EPP Provider Management           | `epp.provider.edit`              | Update EPP-specific configuration                 | System-wide, EPP-specific | Cannot modify core org data (name, address) - those come from EEM |
| EPP Provider Management           | `epp.provider.delete`            | Close/deactivate EPP provider                     | System-wide               | Typically restricted to prevent accidental deletion               |
| Certificate Category Management   | `epp.certificatecategory.create` | Add certificate type/pathway approval to EPP      | System-wide, EPP-specific | Grants authority to offer new program types                       |
| Certificate Category Management   | `epp.certificatecategory.edit`   | Modify certificate category approval dates        | System-wide, EPP-specific | Update MDE approval, enrollment close, recommend close dates      |
| Certificate Category Management   | `epp.certificatecategory.delete` | Remove certificate category approval              | System-wide, EPP-specific | Blocked if active candidate enrollments exist                     |
| Endorsement Management            | `epp.endorsement.create`         | Add approved endorsement to EPP                   | System-wide, EPP-specific | Must have approved certificate type first                         |
| Endorsement Management            | `epp.endorsement.edit`           | Modify endorsement approval dates                 | System-wide, EPP-specific | Update MDE approval, enrollment close, recommend close dates      |
| Endorsement Management            | `epp.endorsement.delete`         | Remove approved endorsement                       | System-wide, EPP-specific | Blocked if active candidates pursuing endorsement                 |
| Pro Prep Management               | `epp.proprepconfig.manage`       | Configure Pro Prep catalog visibility             | System-wide, EPP-specific | Toggle program visibility, manage page content                    |
| Candidate Enrollment Verification | `epp.enrollment.verify`          | Accept or reject candidate enrollment             | EPP-specific              | Core EPP coordinator function                                     |
| Candidate Enrollment Management   | `epp.enrollment.view`            | View candidate enrollment records                 | EPP-specific, Self-only   | Coordinators see their EPP's candidates only                      |
| Candidate Enrollment Management   | `epp.enrollment.edit`            | Update candidate enrollment status and programs   | EPP-specific              | Modify status, add/remove programs, update details                |
| Candidate Enrollment Management   | `epp.enrollment.bulk-upload`     | Upload bulk candidate tracking file               | EPP-specific              | Batch enrollment updates from institutional systems               |
| Credential Application Review     | `epp.applications.view`          | View credential applications requiring EPP review | EPP-specific              | See applications requiring EPP review                             |
| Credential Application Review     | `epp.applications.review`        | Review and take action on credential applications | EPP-specific              | Hold, Deny, Cancel, Recommend actions                             |
| Credential Application Review     | `epp.applications.recommend`     | Recommend candidates for credentials              | EPP-specific              | May be separated from general review for dual approval            |
| Approval Application Review       | `epp.approvals.view`             | View approval applications requiring EPP review   | EPP-specific              | See alternative route approval applications                       |
| Approval Application Review       | `epp.approvals.review`           | Review and take action on approval applications   | EPP-specific              | Recommend or Deny approval applications                           |
| Reporting                         | `epp.reports.view`               | View EPP-specific reports                         | EPP-specific, System-wide | Access to enrollment metrics, application statistics              |
| Administration                    | `epp.admin.system`               | Full administrative access to EPP domain          | System-wide               | Super admin - all EPP functions across all providers              |

---

## Notes

- **EPP Scope Enforcement:** All EPP-scoped permissions are enforced at the API level by filtering queries based on the user's assigned EPP code(s) from IAM authorization context
- **"Worklist" Framing:** What stakeholders call a "worklist" is simply the filtered, sorted list view each screen already presents over candidate enrollments and applications. It is not a separate permission or capability — access is governed entirely by the existing `epp.enrollment.view`, `epp.applications.view`, and `epp.approvals.view` permissions, scoped to the user's EPP
- **Cross-Domain Coordination:** EPP domain frequently calls Credentialing and IAM APIs. Users may need permissions in those domains for full functionality (e.g., `credentialing.applications.view` to see full application details)
- **Bulk Upload Permission:** `epp.enrollment.bulk-upload` is separate from `epp.enrollment.edit` to allow for finer-grained control and audit trail of bulk vs. individual updates
- **Recommendation Separation:** Some institutions may separate `epp.applications.review` (Hold, Deny, Cancel) from `epp.applications.recommend` for dual approval workflows, though this is not required
