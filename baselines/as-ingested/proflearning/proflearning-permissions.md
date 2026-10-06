# Professional Learning - Permissions Catalog (DRAFT)

This document defines all atomic permissions for the Professional Learning domain.

---

## Permissions

| Category                  | Permission ID                            | Description                                                     | Applicable Scopes               | Notes                                               |
| ------------------------- | ---------------------------------------- | --------------------------------------------------------------- | ------------------------------- | --------------------------------------------------- |
| **Sponsor Management**    | `proflearning.sponsor.view`              | View sponsor details and program list                           | Entity (specific sponsor)       | Coordinators have this for their sponsor            |
| **Sponsor Management**    | `proflearning.sponsor.edit`              | Edit sponsor contact information and metadata                   | Entity (specific sponsor)       | Coordinators for their sponsor, admins system-wide  |
| **Sponsor Management**    | `proflearning.sponsor.approve`           | Approve new sponsor applications                                | System-wide                     | Professional Learning Admin only                    |
| **Sponsor Management**    | `proflearning.sponsor.delete`            | Delete sponsor applications or approved sponsors                | System-wide                     | Professional Learning Admin only                    |
| **Program Management**    | `proflearning.program.create`            | Submit new program applications                                 | Entity (sponsor-scoped)         | Coordinators for their sponsor                      |
| **Program Management**    | `proflearning.program.edit`              | Edit program details (triggers re-approval if approved)         | Entity (program-scoped)         | Coordinators for programs under their sponsor       |
| **Program Management**    | `proflearning.program.view`              | View program application details                                | Entity (program-scoped)         | Coordinators, admins, or public for active programs |
| **Program Management**    | `proflearning.program.approve`           | Approve program applications                                    | System-wide                     | Professional Learning Admin only                    |
| **Program Management**    | `proflearning.program.withdraw`          | Withdraw or delete programs                                     | Entity (program-scoped)         | Coordinators for their programs, admins system-wide |
| **Session Management**    | `proflearning.session.create`            | Add program sessions (dates/locations)                          | Entity (program-scoped)         | Coordinators, Professional Learning Admin           |
| **Session Management**    | `proflearning.session.edit`              | Edit session dates and locations                                | Entity (session-scoped)         | Coordinators, Professional Learning Admin           |
| **Attendance Management** | `proflearning.attendance.add`            | Add attendees to program roster                                 | Entity (session-scoped)         | Coordinators for their programs                     |
| **Attendance Management** | `proflearning.attendance.view`           | View attendance roster                                          | Entity (session-scoped)         | Coordinators for their programs, admins system-wide |
| **Attendance Management** | `proflearning.attendance.adjust`         | Adjust awarded SCECH hours before certification                 | Entity (session-scoped)         | Coordinators for their programs                     |
| **Attendance Management** | `proflearning.attendance.certify`        | Certify finalized attendance and award SCECHs                   | Entity (session-scoped)         | Coordinators for their programs                     |
| **Attendance Management** | `proflearning.attendance.override`       | Adjust SCECH hours after certification                          | System-wide                     | Professional Learning Admin only                    |
| **Evaluation Management** | `proflearning.evaluation.view`           | View submitted evaluations                                      | Entity (program-scoped)         | Coordinators, Professional Learning Admin           |
| **Evaluation Management** | `proflearning.evaluation.submit`         | Submit program evaluation                                       | Self-only                       | Attendees for programs they attended                |
| **College Course Credit** | `proflearning.collegecourse.apply`       | Apply college course for SCECH credit                           | Self-only                       | Educators for their own courses                     |
| **Admin Functions**       | `proflearning.admin.categories`          | Manage program categories and subcategories                     | System-wide                     | Professional Learning Admin                         |
| **Admin Functions**       | `proflearning.admin.questions`           | Manage evaluation question sets                                 | System-wide                     | Professional Learning Admin                         |
| **Public Access**         | `proflearning.catalog.search`            | Search public professional learning catalog                     | System-wide (unauthenticated)   | No login required                                   |
| **Public Access**         | `proflearning.catalog.bookmark`          | Bookmark programs/sponsors                                      | System-wide (authenticated)     | Requires login                                      |
| **Correction Requests**   | `proflearning.correction.submit`         | Submit SCECH correction request for certified attendance        | Entity (session-scoped)         | Coordinators for their programs                     |
| **Correction Requests**   | `proflearning.correction.view`           | View correction requests                                        | Entity (sponsor/session-scoped) | Coordinators see their requests, admins see all     |
| **Correction Requests**   | `proflearning.correction.approve`        | Approve SCECH correction requests                               | System-wide                     | Professional Learning Admin only                    |
| **Admin Functions**       | `proflearning.admin.evaluationtemplates` | Manage evaluation templates                                     | System-wide                     | Professional Learning Admin                         |
| **Reports**               | `proflearning.report.view`               | View and export all professional learning reports and analytics | System-wide                     | Includes export; Professional Learning Admin        |
