# Communications - Permissions Catalog

This document defines all atomic permissions for the Communications platform capability.

---

## Permission Patterns

The following patterns are used consistently across all functional areas. When a new functional area is added, all permissions in each applicable pattern must be provisioned.

### Template Management Pattern

For each functional area `{fa}`, the following permissions are defined:

| Permission ID                            | Description                                                                 |
| ---------------------------------------- | --------------------------------------------------------------------------- |
| `communications.{fa}-templates.view`     | View email templates                                                        |
| `communications.{fa}-templates.create`   | Create new email templates (draft; activation requires separate permission) |
| `communications.{fa}-templates.edit`     | Edit email templates (creates new version; does not auto-activate)          |
| `communications.{fa}-templates.activate` | Activate a template version (sets it as current/active)                     |
| `communications.{fa}-templates.delete`   | Archive email templates (soft delete; remains in audit trail)               |

### Email History Pattern

For each functional area `{fa}`, the following permissions are defined:

| Permission ID                             | Description                                                                                                                              |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `communications.{fa}-emails.view-history` | View and search historical sent emails; includes filtering by recipient, date, and status                                                |
| `communications.{fa}-emails.resend`       | Resend historical emails with recipient override (requires corresponding `view-history`; only available within content retention period) |

> **Note:** Search/filtering is a capability of `view-history`, not a separate permission. Anyone who can view history can also search it.

---

## Permissions

| Category                      | Permission ID                                     | Description                                                            | Applicable Scopes                    | Notes                                                                             |
| ----------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------ | --------------------------------------------------------------------------------- |
| Domain-Specific: credentialing | `communications.credentialing-templates.view` | View Credentialing email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: credentialing | `communications.credentialing-templates.create` | Create new Credentialing email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: credentialing | `communications.credentialing-templates.edit` | Edit Credentialing email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: credentialing | `communications.credentialing-templates.activate` | Activate Credentialing template versions | System-wide | Sets template version as current/active |
| Domain-Specific: credentialing | `communications.credentialing-templates.delete` | Archive Credentialing email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: credentialing | `communications.credentialing-emails.view-history` | View and search historical sent Credentialing emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: credentialing | `communications.credentialing-emails.resend` | Resend historical Credentialing emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Domain-Specific: staffing | `communications.staffing-templates.view` | View Staffing email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: staffing | `communications.staffing-templates.create` | Create new Staffing email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: staffing | `communications.staffing-templates.edit` | Edit Staffing email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: staffing | `communications.staffing-templates.activate` | Activate Staffing template versions | System-wide | Sets template version as current/active |
| Domain-Specific: staffing | `communications.staffing-templates.delete` | Archive Staffing email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: staffing | `communications.staffing-emails.view-history` | View and search historical sent Staffing emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: staffing | `communications.staffing-emails.resend` | Resend historical Staffing emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Domain-Specific: proflearning | `communications.proflearning-templates.view` | View Professional Learning email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: proflearning | `communications.proflearning-templates.create` | Create new Professional Learning email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: proflearning | `communications.proflearning-templates.edit` | Edit Professional Learning email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: proflearning | `communications.proflearning-templates.activate` | Activate Professional Learning template versions | System-wide | Sets template version as current/active |
| Domain-Specific: proflearning | `communications.proflearning-templates.delete` | Archive Professional Learning email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: proflearning | `communications.proflearning-emails.view-history` | View and search historical sent Professional Learning emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: proflearning | `communications.proflearning-emails.resend` | Resend historical Professional Learning emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Domain-Specific: epp | `communications.epp-templates.view` | View EPP email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: epp | `communications.epp-templates.create` | Create new EPP email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: epp | `communications.epp-templates.edit` | Edit EPP email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: epp | `communications.epp-templates.activate` | Activate EPP template versions | System-wide | Sets template version as current/active |
| Domain-Specific: epp | `communications.epp-templates.delete` | Archive EPP email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: epp | `communications.epp-emails.view-history` | View and search historical sent EPP emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: epp | `communications.epp-emails.resend` | Resend historical EPP emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Domain-Specific: profpractice | `communications.profpractice-templates.view` | View Professional Practice Review email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: profpractice | `communications.profpractice-templates.create` | Create new Professional Practice Review email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: profpractice | `communications.profpractice-templates.edit` | Edit Professional Practice Review email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: profpractice | `communications.profpractice-templates.activate` | Activate Professional Practice Review template versions | System-wide | Sets template version as current/active |
| Domain-Specific: profpractice | `communications.profpractice-templates.delete` | Archive Professional Practice Review email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: profpractice | `communications.profpractice-emails.view-history` | View and search historical sent Professional Practice Review emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: profpractice | `communications.profpractice-emails.resend` | Resend historical Professional Practice Review emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Domain-Specific: payments | `communications.payments-templates.view` | View Payments email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: payments | `communications.payments-templates.create` | Create new Payments email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: payments | `communications.payments-templates.edit` | Edit Payments email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: payments | `communications.payments-templates.activate` | Activate Payments template versions | System-wide | Sets template version as current/active |
| Domain-Specific: payments | `communications.payments-templates.delete` | Archive Payments email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: payments | `communications.payments-emails.view-history` | View and search historical sent Payments emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: payments | `communications.payments-emails.resend` | Resend historical Payments emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Domain-Specific: iam | `communications.iam-templates.view` | View Identity and Access Management email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: iam | `communications.iam-templates.create` | Create new Identity and Access Management email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: iam | `communications.iam-templates.edit` | Edit Identity and Access Management email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: iam | `communications.iam-templates.activate` | Activate Identity and Access Management template versions | System-wide | Sets template version as current/active |
| Domain-Specific: iam | `communications.iam-templates.delete` | Archive Identity and Access Management email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: iam | `communications.iam-emails.view-history` | View and search historical sent Identity and Access Management emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: iam | `communications.iam-emails.resend` | Resend historical Identity and Access Management emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Domain-Specific: documents | `communications.documents-templates.view` | View Documents email templates | System-wide | Read-only access to template list and versions |
| Domain-Specific: documents | `communications.documents-templates.create` | Create new Documents email templates | System-wide | Can create drafts; activation requires separate permission |
| Domain-Specific: documents | `communications.documents-templates.edit` | Edit Documents email templates | System-wide | Creates new template version; does not auto-activate |
| Domain-Specific: documents | `communications.documents-templates.activate` | Activate Documents template versions | System-wide | Sets template version as current/active |
| Domain-Specific: documents | `communications.documents-templates.delete` | Archive Documents email templates | System-wide | Soft delete; templates remain in audit trail |
| Domain-Specific: documents | `communications.documents-emails.view-history` | View and search historical sent Documents emails | System-wide | Includes filtering by recipient, date, status; typically for help desk and admins |
| Domain-Specific: documents | `communications.documents-emails.resend` | Resend historical Documents emails with recipient override | System-wide | Only available during content retention period; requires view-history |
| Template Variables            | `communications.template-variables.view`          | View template variable definitions                                     | System-wide                          | See available variables and their data sources                                    |
| Email Sending                 | `communications.email.send-manual`                | Manually compose and send emails using templates                       | System-wide                          | Typically for help desk staff sending one-off emails on behalf of users           |
| Email Sending                 | `communications.email.send-adhoc`                 | Send ad-hoc emails without using templates                             | System-wide                          | System Admin only; bypasses template framework                                    |
| Email Sending                 | `communications.email.customize`                  | Customize template content for one-time send                           | System-wide                          | Modify template content without altering base template                            |
| Email History - View          | `communications.email.view-all-history`           | View all historical sent emails across all functional areas            | System-wide                          | System Admin only; bypasses functional area filtering                             |
| Email History - Details       | `communications.email.view-details`               | View full email details including content, metadata, delivery events   | System-wide                          | Requires corresponding view-history permission                                    |
| Alert Management              | `communications.alerts.view`                      | View alerts for functional area                                        | System-wide                          | See alert templates and configuration                                             |
| Alert Management              | `communications.alerts.create`                    | Create new alert templates                                             | System-wide                          | Define alert content, targeting, triggers                                         |
| Alert Management              | `communications.alerts.edit`                      | Edit alert templates                                                   | System-wide                          |                                                                                   |
| Alert Management              | `communications.alerts.delete`                    | Archive alert templates                                                | System-wide                          |                                                                                   |
| Comment Management            | `communications.comments.view-internal`           | View internal comments on process items                                | Building, District, ISD (transitive) | Internal staff only; scoped to their org level                                    |
| Comment Management            | `communications.comments.create-internal`         | Create internal comments                                               | Building, District, ISD (transitive) | Comments not visible to external users; scoped to their org level                 |
| Comment Management            | `communications.comments.create-external`         | Create external comments                                               | Building, District, ISD (transitive) | Comments may be shared with citizens/applicants; scoped to their org level        |
| Comment Management            | `communications.comments.edit`                    | Edit comments (before email send)                                      | Building, District, ISD (transitive) | Cannot edit after included in sent email; scoped to their org level               |
| Comment Management            | `communications.comments.delete`                  | Delete comments                                                        | Building, District, ISD (transitive) | Soft delete; remains in audit trail; scoped to their org level                    |
| Predefined Comments           | `communications.predefined-comments.view`         | View predefined comment library                                        | System-wide                          |                                                                                   |
| Predefined Comments           | `communications.predefined-comments.manage`       | Create/edit predefined comments                                        | System-wide                          | System Admin or Functional Area Lead                                              |
| Reporting                     | `communications.reports.view`                     | View template usage analytics                                          | System-wide                          | Send counts, frequency, trigger analysis                                          |

---

## Scope Definitions

**System-wide:** Permission applies across entire system, no scoping restrictions. Typically granted to System Admins or Functional Area Leads.

**Building, District, ISD (transitive):** Permission is scoped to a specific organizational level.
- **Building** = School building
- **District** = Local school district (contains multiple buildings)
- **ISD** = Intermediate School District (contains multiple districts)

**Transitive:** If granted at ISD level, permission automatically applies to all constituent districts and buildings within that ISD. If granted at District level, applies to all buildings within that district.

**Functional Area:** Permission is scoped by functional area (Credentialing, Staffing, etc.) but applies system-wide within that functional area. Most communications permissions are not scoped to organizational hierarchy since email management is an administrative function.

**Example:**
- User has `communications.credentials-emails.view-history` (system-wide, functional area scoped)
- User can view all credentialing emails across the entire system
- User CANNOT view staffing emails (different functional area)

**Note:** In practice, most communications permissions are granted to system administrators and functional area administrators who need system-wide visibility for help desk support, troubleshooting delivery issues, and responding to citizen inquiries.

---

## Permission Combinations

### Email History Viewing

To view email history, a user needs the appropriate functional area permission:
- `communications.{functional-area}-emails.view-history` - View emails for specific functional area (Credentialing, Staffing, etc.)
- `communications.email.view-all-history` - View all emails across all functional areas (System Admin only)

**Typical Roles:**
- **Credentialing Administrator:** Has `credentials-emails.view-history` - can see all credentialing emails system-wide
- **Staffing Administrator:** Has `staffing-emails.view-history` - can see all staffing emails system-wide  
- **Help Desk Staff:** May have multiple functional area permissions depending on support responsibilities
- **System Administrator:** Has `email.view-all-history` - can see everything

**Note:** These permissions are system-wide because administrators need full visibility for troubleshooting delivery issues, responding to citizen inquiries, and providing help desk support regardless of which district or building the email relates to.

---

### Email Resending

To resend an email, a user needs:
1. **View Permission:** Corresponding `view-history` permission for that functional area
2. **Resend Permission:** `communications.{functional-area}-emails.resend`
3. **Content Availability:** Email content must still exist in hot storage (within retention period)

**Constraints:**
- Resend disabled if email content has been archived
- Resend creates new EmailInstance referencing original
- Recipient override allowed (can change email address)
- User notified if recipient's email address has been updated since original send

**Typical Use Cases:**
- Email bounced due to invalid address, help desk resends to corrected address
- Citizen claims they never received email, administrator resends to confirm delivery
- Original email had incorrect content, administrator resends corrected version

---

### Template Management

To fully manage templates in a functional area:
1. `communications.{functional-area}-templates.create` - Create new templates
2. `communications.{functional-area}-templates.edit` - Create new versions
3. `communications.{functional-area}-templates.activate` - Set version as active
4. `communications.{functional-area}-templates.delete` - Archive templates

**Typical Roles:**
- **Functional Area Administrator** (e.g., Credentialing Administrator) - Has all template permissions for their functional area
- **System Administrator** - Has template permissions across all functional areas

**Note:** Template management is strictly a system-wide administrative function. End users (district staff, building administrators) never manage templates directly.

---

### Comment Management (Organizational Scoping)

Comments are one of the few permissions that ARE scoped to organizational hierarchy because they relate to specific process items (credential applications, staffing records) that belong to districts/buildings:

**Example:**
- District XYZ staff member has `communications.comments.create-external` at District level
- Can create external comments on credential applications for applicants in District XYZ
- Cannot create comments on applications for District ABC

**Rationale:** Comments are tied to business processes that have organizational ownership, unlike email templates which are centrally managed infrastructure.

---

## Implementation Notes

### Scoping Considerations

Unlike most domain permissions, email history permissions are intentionally NOT scoped to organizational hierarchy:

**Rationale:**
- Email management is a centralized administrative function
- Help desk staff need system-wide visibility to troubleshoot delivery issues
- Administrators respond to citizen inquiries regardless of which district sent the email
- Functional area scoping provides sufficient access control (Credentialing Admin sees credentialing emails, not staffing emails)

Comments ARE scoped to organizational hierarchy because they're attached to business process items.

### Resend Availability Logic

When displaying email details or history, the UI should check:
1. Does user have corresponding `resend` permission?
2. Is email content still in hot storage (`content_status = 'Active'`)?
3. If both true -> enable "Resend" button
4. If content archived -> disable button with tooltip "Email content no longer available (archived)"
