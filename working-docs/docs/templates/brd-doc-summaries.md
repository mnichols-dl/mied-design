# Feature Overviews

Generated: 2025-11-17 19:17:36

---

## 1.1 MILogin Citizen and Business for Third party

The MiEdWorkforce system must implement authentication via MiLogin, implementing its own authorization framework to support business functionality, data restriction requirements, and State standards. The authorization process and workflow are unique to the MiLogin type: MiLogin for Citizens uses an auto-approval mechanism, while MiLogin for Business directs users to security access forms. Users are required to login to MiLogin each time MiEdWorkforce is accessed.

---

## 1.2 MiLogin Common

MiEdWorkforce will integrate with MiLogin to perform User Authentication for all user types (Citizens, Business, and Workers). MiLogin defines the underlying user authentication process. It is mandated that users log in to MiLogin each time MiEdWorkforce is accessed. OpenID Connect is the strongly preferred technical solution for this integration.

---

## 1.3 User Group Management

The system will provide an administrative screen for MiEdWorkforce System Administrators to view, add, and edit user groups based on MiLogin type. Defined User Groups include System Administrator, Credential, Staffing, EPP, Professional Learning, Data Quality, and Citizen. Users may be assigned to multiple groups, but only if all selected groups belong to the same MiLogin type.

---

## 1.4 User search

A dedicated page will be available for all System Administrators to search for User Accounts. The search list must include users who have authorized roles established, as well as those who have requested access and are currently pending the assignment of groups/roles. The search function supports filters such as MiLoginID, Name, Email, and MiLogin Type. Furthermore, Admins can view, edit, and remove user permissions directly through this page without needing the standard User Removal Request form.

---

## 1.5 User Roles

The system requires an administrative screen that allows the MiEdWorkforce System Administrator to view, add, and edit User Roles. The definition and permissions associated with these roles are based on established User Groups and MiLogin type, as defined in the external document MiEdWorkforce Role Matrix.xlsx.

---

## 1.6 MiEdWorkforce Account Registration Requirements

After a MiLogin for Citizens user is auto-approved for access, they are routed to the “Account Creation Landing Page”. This process is critical for establishing the user’s identity, requiring them to complete demographic data elements so that MiEdWorkforce can use Mi-Key services for identity matching and assign a Unique ID. The system enforces that the Educational Staff Dashboard cannot be accessed until this Unique ID is assigned.

---

## 1.7 User Authorization reports

The system requires standard, custom, and ad-hoc reports related to User Authorization. This functionality leverages the framework developed in Epic 2 (Reporting). The MiEdWorkforce System Administrator will be able to define and maintain the standard reports, which include specific metrics such as counts of new user accounts by MiLogin Type, entities with missing groups/roles, counts of authorized user requests denied by the Lead Administrator, and audit summary reports.

---

## 1.8 Emulate system for specific users and entities

The system supports Admin access to view the system as any user (User Impersonation) for troubleshooting and support purposes. This feature permits the Admin to search for a Citizen user by Unique ID, or search for a Business user/entity by entity code/name. During impersonation, the Admin can view data like in-process applications, dashboards, rosters, and reports, including specific error messages received by the user. Crucially, the Admin is strictly prohibited from performing any transactions while impersonating.

---

## 1.61 Business User Account Creation_User Management

This feature establishes a formal, semi-automated process for business users to request role-based access to the MiEdWorkforce application. After authenticating via MiLogin, a user is directed to MiEdWorkforce where they select an Access Group (EPP, School District, or SCECH). The system then presents an access-group-specific security form (an embedded Google Form) for the user to complete.

Upon submission, the request is routed via email to the designated Lead Administrator(s) of the selected educational entity for review. The Lead Admin can approve, modify and approve, or deny the request. The system then updates the user's permissions in the MiEdWorkforce database accordingly and notifies the user of the outcome. The entire workflow includes automated notifications and expiration handling for pending requests.

3. Scope

---

## 2.1 Reporting Framework Implementation

This document defines the base functionality needed to implement reporting within MiEdWorkforce. The system will establish Power BI as the core reporting framework, aligning with the Technology Standards. This framework is foundational and applied globally to all reporting needs stemming from various modules, including User Authorization, Data Quality, and Staffing Compliance. Reports must be embedded within the application interface and generated based on the user's specific access and organizational data. Authorized administrators are required to define, manage, and maintain report availability, viewability, and associated audit controls. Ad-Hoc report creation must be restricted to the Administrator level due to licensing and technical constraints, and will take place primarily within the Power BI workspace/development environment.

---

## 3.1 Dashboard - School District

The School District Dashboard provides users with dynamic and customizable tiles displaying an initial summary of tasks, status monitoring, alerts, notifications, and progress tracking related primarily to data collection and compliance activities (such as Roster of Positions and employee data submissions) for their selected entity. The dashboard is intended to serve as the user's home page and aid navigation.

---

## 3.2 Dashboard - Educational Staff

The Educational Staff Dashboard provides Citizen Users with dynamic and customizable tiles displaying an initial summary of personal tasks, alerts, notifications, and progress tracking, primarily concerning individual credentials, certificates, and professional learning activities. The content seen upon logging in using My Login for citizens is distinct from that of business user types.

---

## 3.3 Dashboard - ISD Auditor

The ISD/Auditor Dashboard provides users with dynamic and customizable tiles focused on audit tracking, submission status monitoring (e.g., Ingham ISD - audit tracking), data quality review, and overall compliance across multiple districts or entities within their purview.

---

## 3.4 Dashboard - Educator Preparation Providers

The EPP Dashboard provides EPP users with dynamic and customizable tiles displaying an initial summary of EPP processes, candidate tracking, worklists, and credential recommendation statuses. EPP Admin roles have additional configuration capabilities for defining tile availability.

---

## 3.5 Dashboard - Professional Learning Coordinator

The PLC Admin Dashboard provides users with dynamic tiles focused on managing and tracking professional learning (PL) activities, including credit hours, application status, and review status. It also includes status monitoring for Professional Practices Review.

---

## 3.6 Dashboard - Credential Admin

The Credential Admin Dashboard provides users with dynamic tiles focused on monitoring the status of credential applications, certificate processing, managing credential-related alerts, and defining system parameters, such as the specified timeframe for unread messaging aging.

---

## 3.7 Dashboard - Staffing Admin

The Staffing Admin Dashboard provides users with dynamic tiles focused on monitoring the collection report status, staffing and employment data, open positions, spending, placements, and defining system parameters, such as the specified timeframe for unread messaging aging.

---

## 4.1 Question sets

The Question Sets feature is an administrative tool that enables functional area administrators to author, modify, and version validation logic without developer intervention. The system uses decision tree logic to route users through specific question paths based on their previous answers (e.g., yes/no branching). This feature manages all components of the question sets, including question text, associated error messages, effective dates, and versioning. The primary goal is to provide the administration team with the flexibility to quickly adapt application requirements (e.g., changing required credit hours).

---

## 4.2 Business Logic Rules

The Business Logic/Rules feature establishes a robust business rule engine to validate data input, supporting user workflows and overall system functionality. The system will allow Admins, segregated by functional area, to manage and maintain these rules by setting validation conditions, severity status (Error/Warning), and justification options (Required/Optional). This feature addresses the historical challenge of having hard-coded rules requiring developer intervention, allowing admins to manage changes like district exemptions or updated criteria. The engine must also connect to external data sources like the Data Warehouse and EEM for comprehensive validation.

---

## 5.1 Managing Certificate Permit Types

The feature enables the System Administrator – Credentialing role (also referred to as Credential Admin role) to search for existing Certificate or Permit types by category, view their details, and update fields such as validity, fees, and effective dates. Upon saving changes, the system will version the record based on the Effective From date field.

---

## 5.2 Managing Endorsements (CIP Codes included)

The Credential Admin can view, edit, or delete endorsement classifications (e.g., Class Name, Description). The Admin can search for existing endorsements using various filters, including Cert Type (Teacher/School Admin only), Endorsement/CIP Code, and Subject Area. When editing an endorsement, the Admin can manage associated tables for Endorsement Codes (specifying validity parameters) and Test Codes (defining MTTC requirements). Adding a new endorsement code triggers an action item (notification) for the MiEdWorkforce System Admin to define assignment mappings.

---

## 5.3 Managing Global Parameters

The Credential Admin can search for a parameter using its description. The search results table displays the Description, Value, Format, and audit details. The Admin is authorized to update the Description or the Value of these parameters, which are often used to manage system-wide or role-specific communications (e.g., system downtime messages or landing page text). The system captures audit details upon saving changes. Note that while currently named "Global Parameters," the functionality is intended for credential/staffing-level messaging, not general system administration.

---

## 5.4 Run & View Batch Jobs

The Credential Admin can search for existing batch jobs using filter options such as Run Date and Batch Job name. The search results display the Batch Job, Run Status, Run Start Date, Run End Date, and Run Status Description. The admin can manually select and execute a predefined batch job, and the system automatically captures audit details for these manual runs. The run status description is expected to be auto-generated by the system.

---

## 5.5 Manage Worklist Assignment

The Credential Admin searches for users using filters including User Type (EPP, School District, Applicant), First Name, Last Name, and Active status. Depending on the User Type selected, specific search filters (e.g., SSN for Applicant, School Name for School District) are displayed. Once a user is selected from the results table, the Admin can view and modify the user's assigned worklists and report access. All modifications trigger the capture of audit details (Modified By, Modified On). Assignment is expected to be managed at the individual user level.

---

## 5.6 Reports

The Credential Admin manages report settings, including allowing for immediate or future retiring of report availability. The Admin can define report access based on authorized user roles. This administrative page also allows the Admin to search existing reports, create ad-hoc and custom reports, manage export/download options, and manage report resources such as page text and links. Audit details must be stored for all modifications.

---

## 6.1 Email Notification implementation

This feature defines the underlying system support required for sending emails, including the ability to address communications to multiple recipients (To: and CC:), send emails internally or externally, and execute mass email sending based on selected criteria or user types. These communications are triggered by processes such as credential application status changes, payment reminders, or date-dependent notifications.

---

## 6.2 Email Notification - Coded Inserts (variables)

This feature ensures that personalized and context-specific data, such as applicant name, fees, or reason codes, is automatically merged from existing database fields into email templates. The system must merge this information in a defined, appropriate order set by the System Administrator. Coded inserts are also referred to as "variables" or "tokens".

---

## 6.3 Edit Email Templates

This feature provides System Administrators with tools to manage all email templates, supporting functionality like rich text, HTML, image inserts (such as MDE/CEPI letterhead), and automated template versioning. Management functions include adding new templates and searching, modifying, and deleting existing ones.

---

## 6.4 Email external comments for credential and staffing processing to the applicant or data submitter

This feature enables system users (internal SOM staff) to record remarks about a process flow item. Comments are categorized as either Internal (visible only to internal SOM users) or External (which can be selected by the user to be included in an external email communication).

---

## 6.5 Reports

This feature ensures that data regarding all alerts, emails, and communications is available within the central reporting framework. This integration supports audit needs, business intelligence analysis, and the ability to review historical communication metrics, such as counts of emails sent and reminders issued.

---

## 7.1 CEPAS Interface (Payments and Refunds)

MiEdWorkforce will integrate with CEPAS using API capabilities to process payments for credential applications and permits. Users requiring payment will be redirected to the secure CEPAS site to complete the transaction, as MiEdWorkforce does not process or store sensitive payment information. The system will receive immediate confirmation (Success/Error) upon return. Additionally, the system supports real-time refund processing initiated by a Credential Administrator.

---

## 7.2 CEPAS Reconciliation

MiEdWorkforce receives a daily CEPAS post file containing information about single payments, bulk payments, single refunds, and bulk refunds. A daily batch job processes this file to check for any payments that were processed successfully by CEPAS but missed by the MiEdWorkforce system during the real-time return, thus serving as the source of truth for payment status. This data populates audit tables available to the Credential Administrator for review and filtering.

---

## 7.3 Batch processing for payments and refunds

The MiEdWorkforce system must support the ability for authorized users, typically District Staff (Business Users), to select multiple credential applications (such as temporary credentials) with outstanding fees and submit a single, bulk payment to CEPAS. Furthermore, the system must enable the MiEdWorkforce Credential Administrator to initiate bulk refunds via the CEPAS API.

---

## 8.1 Receive - EEM Interface

The EEM Interface allows MiEdWorkforce to receive official educational entity data from the Educational Entity Master (EEM) system. EEM serves as the official source of truth for all Michigan education entities. The data flow is one-way, with MiEdWorkforce receiving EEM data. This entity information is crucial for determining which entity a user belongs to upon login, driving core system functionality based on entity type and settings, and providing necessary validation for various data submissions. The EEM data is initially retrieved via a high-frequency REST API connection, but since the returned data is not in CEDS standard format, it must be processed through an ETL pipeline (known as the Entity Directory Data Population). This process maps the data into CEDS aligned tables, ensuring MiEdWorkforce stores a local copy and removing dependency on live EEM calls.

---

## 8.2 Receive - NASDTEC Interface

The MiEdWorkforce system is expected to connect to the NASDTEC Ed ID Clearinghouse (National Clearing House of educator disciplinary actions). This system is a secure, searchable national database that provides immediate alerts regarding individuals who have had their professional educator license or certificate annulled, denied, suspended, revoked, or otherwise invalidated. The integration will utilize a RESTful API to pull data. The connection ensures that Michigan can or will only issue credentials to educators who are in good standing. The biggest unknowns for this integration are when to call the API, how often, and what specific actions to take with the results, which are pending further business decisions. The results will ultimately help flag potential issues before an application is approved.

---

## 8.3 Receive - STARR interface

The MiEdWorkforce system will integrate with the CEPI STARR Collection to receive individual level data on post-secondary awards and course information. The system will pull this data from STAR to be used primarily in the credential application processes. Award data received will be used to pre-populate required fields in the credential application. Additionally, course level data will be associated with the individual/citizen account and can be electively used to meet professional learning requirements. This integration is a new connection, as neither REP nor MOECS currently interface with STARR data. The process relies on aligning the Unique ID associated with the individual’s citizen account in MiEdWorkforce and the Unique ID (UIC) submitted within the STARR data collection.

---

## 8.4 Receive_Send - Michigan Data Hub

The Michigan Data Hub (MiDH) interface establishes a new connection between MiEdWorkforce and internal school district systems. This initiative aims to create an ecosystem for information exchange based on pre-defined standards, supporting state and federal reporting requirements. The feature includes capabilities for MiEdWorkforce to send staffing data to MiDH and receive staffing data submitted by districts via MiDH. A critical component of this interface is the mapping of data standards, as MiDH uses the Ed-Fi Data Standard, while MiEdWorkforce utilizes the Common Education Data Standard (CEDS). Incoming data must be mapped from Ed-Fi to CEDS, and outgoing data must be mapped from CEDS to Ed-Fi. Data transmission relies on CEPI Public Services endpoints.

---

## 8.5 Send - Career and Technical Education Information System (CTEIS)

The CTEIS interface is designed to provide MiEdWorkforce’s up-to-date staff roster data to the CTEIS system. CTEIS is the state system used for Career Technical Education (CTE) student course enrollment and outcomes. Currently, districts submit associated staff records redundantly in both the legacy REP system and CTEIS. The new interface aims to eliminate this duplicate data submission by ensuring MiEdWorkforce supplies the identity and employment data required by CTEIS for instructor validation.

The data flow is one-way (send) from MiEdWorkforce to CTEIS and operates daily. The services utilize common service endpoints based on existing REP services, shared across multiple integrations like MSDS/TSDL and NexSys.

---

## 8.6 Send - MDE Grant System (NexSys)

The MiEdWorkforce project requires an interface with NexSys to fulfill requirements for Title I Part A Comparability reporting. NexSys is Michigan's system utilized for federal program reporting. This interface must send the list of all staff members, along with their assignment details, categorized by entity, using the staffing data submissions in MiEdWorkforce. This is designed as a one-way service, where MiEdWorkforce sends data to NexSys, but does not receive data back. The data provided will be impacted by changes to employment and identity reporting, ensuring alignment with the CEDS model. The data flow is one-way (send) from MiEdWorkforce to CTEIS and operates daily. The services utilize common service endpoints based on existing REP services, shared across multiple integrations like MSDS/TSDL and NexSys.

---

## 8.7 Receive_Send - Teacher Student Data Link (TSDL)

The MSDS TSDL is used to connect student courses to teacher records. The TSDL interface requires MiEdWorkforce to validate the Unique ID (currently relying on the PIC) of the Teacher of Record to ensure they have active employment records within MiEdWorkforce. MiEdWorkforce will employ data model changes to support increased quality of student course data collected. The Unique ID used for TSDL submissions must be valid and linked to active employment records in MiEdWorkforce. Student and educator courses will be submitted using the SCED (School Codes for the Exchange of Data) reflecting course content, which should be aligned with assignment reporting in MiEdWorkforce. The interface utilizes a REST API for data transfer, operating on a Daily+ frequency. The main goal of this integration is to align data reporting with the CEDS (Common Education Data Standards) model and reduce data reporting burden in the long term.

---

## 8.8 Send - MDE Secure Site

Secure Site is the application utilized by the Michigan Department of Education (MDE) to register students for state assessments. Previously, Secure Site used a connection to the legacy REP system to retrieve a list of individuals by school and district, including limited assignment details, to facilitate the Online Sessions feature (e.g., assigning Testing Proctors). Starting in 2023, MDE retired this connection. This feature ensures that MiEdWorkforce maintains the technical ability to share the required Secure Site data for future assessment processing with limited development effort. The long-term goal is to have such robust and clean data in MiEdWorkforce that the Secure Site team will be incentivized to connect to this interface for seamless updates, reducing their internal maintenance burden. This interface capability is also intended to supply similar employee roster data to other MDE offices and systems, such as Nexus.

---

## 8.9 Receive_Send - CEPI Data Warehouse

The CEPI Data Warehouse is a grant-funded initiative designed to modernize Michigan Longitudinal Data Storage for the long-term storage of system data used in the production of state and federal reports. The Data Warehouse (DW) is CEDS standards aligned and utilizes the Azure Architecture platform. This feature covers the bi-directional interface between MiEdWorkforce and the CEPI Data Warehouse:

1. Sending (Consumption): MiEdWorkforce makes its current transactional data available for consumption by the DW (specifically, the SLDS project pulls data from MiEdWorkforce).

2. Receiving (Retrieval): MiEdWorkforce pulls historical or longitudinal data from the DW/Semantic Layer for display in application screens or reports, especially since the transactional layer is planned to hold only current data.

The Semantic Layer, which is part of the DW architecture, is intended to be the source for all reporting outputs, including data quality dashboards and external reports.

---

## 9.1 Disclosure-IndividualUserDisclosure

This feature provides the citizen users with the ability to self-disclose incidents related to Professional Practices Review (PPR). Through an online disclosure form, users can report new misdemeanors or felonies, supply descriptive details, and upload supporting documentation. This feature will also allow the system, at regular intervals, to remind the Citizen user that they need to update their Professional Practice Disclosure.

---

## 9.2 Disclosure-AdminUser

This feature provides the Credentialing Admin users with the ability to oversee the disclosure process by reviewing citizen-submitted records, updating application statuses, and applying or removing flags that control credential workflows. They configure disclosure questions and branching logic, manage communication templates and reminder schedules, and monitor disclosures through the PPR worklist. In addition, they perform searches, generate compliance and audit reports, and access external data sources such as CHRISS/MSP, Rapback, and NASDTEC for verification. All actions are logged to ensure transparency, with audit records retained for seven years in compliance with state policy.

---

## 9.3 DisclosureProcessing

This feature gives the user the ability to manage the workflow initiated when a citizen self-discloses a criminal history event in MiEdWorkforce. The system routes the submission to the OEE worklist and immediately sets any related credential application to 'Under PPR Review', halting its progression. A PPR System Admin/Review User then reviews the disclosure, utilizing external data from CHRISS/Rapbacks and NASDTEC, and manages the application status with holds (e.g., 'PPR Hold') or final review decisions. The finalized status triggers standardized messages for district users regarding the individual's employment and credentialing eligibility.

---

## 9.4 Disclosure-StopChecks

This feature ensures…

---

## 9.5 ProfessionalPracticeReporting

The Credential Admin manages report settings, including allowing for immediate or future retirement of report availability. The Admin can define report access based on authorized user roles. This administrative page also allows the Admin to search existing reports, create ad-hoc and custom reports, manage export/download options, and manage report resources such as page text and links. Audit details must be stored for all modifications.

---

## 9.6 DocumentUpload

This feature ensures the MiEdWorkforce system can receive and parse data submitted via both file uploads and API payloads for PPR disclosures, allowing for robust data capture. It enables the PPR System Admin to download and extract this submitted data for administrative review and archival purposes. Crucially, the feature also ensures that the Citizen Accountholder maintains a record of their own submission by allowing them to download and extract the data they provided.

---

## 10.1 User Access and Role Management

This feature ensures that the MiEdWorkforce system provides a Staffing Data Administrator role responsible for managing and maintaining employment/staffing data collections, including pages, validations, reports, and availability managed by CEPI. This role maintains user roles and authorizations related to these collections.

---

## 10.2 User Support Materials,Help

This feature provides an administrative page where the Staffing Data Administrator can manage and maintain training manuals and user support materials related to staffing collections. A key function is the ability to auto-generate data definitions using metadata from the CEDS standards.

---

## 10.3 Data Quality Monitoring and Review

This feature provides the Staffing Data Administrator an administrative page to manage and maintain business rules and validations necessary for staffing data collections. This includes assigning rules to specific data elements or collections, setting the severity status (Error, Warning, Data Quality), editing end-user validation text, and managing justification requirements. The functionality leverages the Business Rules Management framework.

---

## 10.4 Manage System Alerts,Notifications

This feature allows the Staffing Data Administrator to manage system alerts and notifications for Staffing collections. This involves defining alert text with versioning, setting effective dates and target user roles, and linking alerts to specific system events, such as incomplete worklist tasks or new data quality issues. The administrator can also view existing alerts in a searchable table.

---

## 10.5 Manage Email,Notification Templates

This feature uses the established email framework to allow the Staffing Data Administrator to create and manage email templates within the "Staffing" template group. Functionality includes setting effective dates, defining triggers, reviewing historical usage metrics (e.g., count of uses), and viewing/filtering existing templates.

---

## 10.6 Dashboard

This feature provides an administrative page where the Staffing Data Administrator can manage Staffing Data for the role-based Dashboards. This includes defining configurable tiles, setting which reports are displayed, and managing help and support links related to staffing dashboard functionality.

---

## 10.7 Manage Reports

This feature provides the Staffing Data Administrator an administrative page to manage reporting functionalities. This includes defining report availability and access based on user roles, providing options for ad-hoc/custom report creation, managing export options, and explicitly marking reports as required for the collection worklist.

---

## 10.8 Data Element Definition Administration

This feature supports the CEDS alignment by allowing the Staffing Data Administrator to add or update data elements (defining their name, type, legal citations, and dependencies) and subsequently define and manage sets of data elements as Categories. This ensures data elements are clearly defined and logically grouped for assignment to collections.

---

## 10.9 Position Roster Definition

This feature is managed via the central Add/Modify Collection administrative page. The Staffing Data Administrator uses this page to set collection parameters specifically for the Position Roster, including dates (Open/Close, Certification), required Entity Types, and the Categories of data elements that must be submitted (e.g., job position status, expected start date).

---

## 10.10 Employee Roster Definition

This feature is managed via the central Add/Modify Collection administrative page. The Staffing Data Administrator uses this page to set collection parameters specifically for the Employee Roster, focusing on demographic and identity data necessary to define who the employees are, setting applicable dates, and mandatory data Categories.

---

## 10.11 Employee Assignment Details Definition

This feature is managed via the central Add/Modify Collection administrative page. The Staffing Data Administrator uses this page to set collection parameters specifically for Employee Assignment Details, ensuring accurate reporting of teacher-course alignment and credential verification requirements. This includes setting applicable dates and defining Categories related to assignments.

---

## 10.12 Staffing ID Matching and Maintenance Definition

This feature provides the Staffing Data Administrator with an administrative page to view and manage key configurations related to identity processing (My Key integration) that affect staffing collections. This includes reviewing identity data authorized user roles/permissions, business rule validations applied to identity processing, available reports, alerts/communications, and managing the Identity Management Admin Role Worklist. The Staffing Admin is primarily setting the rules, not resolving mismatches directly.

---

## 10.13 Business rule management

This feature provides the Staffing Data Administrator an administrative page to define business rules and validations. This functionality includes assigning rules to specific data contexts (Collection, Category, or Data Element), setting severity (Error, Warning, Data Quality), editing the text shown to the end user, and providing justification for changes. This functionality utilizes the framework developed in Business Rules Management (4.0).

---

## 10.14 Queue Management (API,File Uploads)

This feature provides the Staffing Data Administrator with an administrative page to monitor and control data submissions that occur via bulk file upload or API input methods. Key functions include viewing detailed file queue status (File Name, Type/Size, Collection, Status, Input Method), viewing error details, viewing audit details, and performing corrective actions such as resetting/reprocessing stuck files or deleting queued files.

---

## 10.15 Data Closeout and Processing

This feature allows the Staffing Data Administrator to control the timing and method of processing finalized collection data. This processing includes ensuring data takes immediate effect to populate the longitudinal database (CEPI CEDS Data Warehouse) or allows the Administrator to schedule closeout processing for a future date.

---

## 10.16 Historical Data Connections,Conversions

This feature encompasses the definition of system requirements necessary to handle historical data. This includes ensuring that historical views are aligned with the Common Education Data Standard (CEDS) and defining the retention schedule, which is anticipated to be a 5 to 7-year retention window. The system must allow users to view historical/legacy data, though the primary source of historical data storage is the Data Warehouse.

---

## 11.1 Temporary credential - Bulk renewal

This feature allows a MiEdWorkforce user, specifically those with the Business user/Temporary Credential role, to search for and complete renewals or bulk renewals of temporary credentials for educators. The system applies specific criteria (e.g., PPR flags, Needs Responses flag) to exclude ineligible records and route eligible applications either to the OEE worklist or to the educator for payment via email.

---

## 11.2 Landing pages - Initial and common Informational Pages

This feature ensures that appropriate users (MiEdWorkforce Business users with the Temporary Credential role, and Citizen users) have access to necessary guidance documents (Temporary Credential and Certificate guidance documents, respectively) via a tile on the MiEdWorkforce dashboard.

---

## 11.3 Historical Data

The system will allow authorized MiEdWorkforce users to view an applicant’s current and historical data related to credentials, permits, documents, enrollment, and conviction information.

---

## 11.4 Professional practice review

PPR involves checking an educator's conviction history (misdemeanor/actionable offenses) to streamline application workflows. The system uses PPR flags and a Needs Responses flag (Y/N) to determine eligibility, renewal search inclusion/exclusion, and application routing to the appropriate worklist. The specific logic for setting PPR flags is detailed in Epic 9.

---

## 11.5 Work list, admin pages

The system supports the creation, management, and access of various worklists for credential processing. These worklists provide role-specific search options and display relevant columns for processing permits, certificates, and other credential types. Additionally, the Credential Administrator role has specific administrative functions, such as suspending, revoking, or nullifying credentials, and manually adding certificates.

---

## 11.6 Email,Communication

The system supports email communications and alerts throughout the credential application review process using Credential Email Templates. Communications are sent to applicants and business users for various events (e.g., submission, status change, payment pending). Credential Processors can customize communications without altering the original templates, and Credential Admins can enter internal and external comments viewable by appropriate roles.

---

## 11.7 Dashboard

The MiEdWorkforce system will integrate educator credentialing data with dashboard functionality based on business needs. This integration will allow the display of statistics and information related to credential types, timeframes, and geographical areas (District/ISD). The dashboard functionality also enables the control and viewing of related data based on user roles.

---

## 11.8 MTTC Verification

During the credential application review and approval process, the system must reference the educator’s MTTC score(s) relevant to their endorsement(s). These scores are used, based on business rules defined in Business Rule Management (4), to approve or deny a credential application.

---

## 11.9 Identity Management Integration

The MiEdWorkforce system integrates with ‘my login’ for user authentication. For system development and administrative purposes, a select number of users (dev team members) have the ability to override 'my login' in lower (non-production) environments. During initial account creation, if a near match is detected, the user's profile page is locked pending admin resolution, and the user is notified.

---

## 11.10 Document upload

The system supports the acceptance of supporting documents uploaded by users (Citizen or Business) to credential applications. Once uploaded, these documents must be available for authorized credential application processors to view, download, print, and export. The system will also receive and parse Upload/API data submissions, potentially using formats like JSON LD.

---

## 12.1 System Administration_Assignment Codes

The system must support the creation of an Admin screen to integrate assignment and endorsements to determine appropriate placement of educators [Focus for this BRD]. This functionality is part of the overall Employee and Credential Audit related functions managed by the MiEdWorkforce System Administrator (Shared MDE/CEPI role) [2-4]. The mappings defined via this interface will allow the system to identify individual records that are out of alignment with the defined appropriate placement rules [5-7]. Additionally, the system must trigger an action item for the MiEdWorkforce System Admin whenever certain changes are made by other functional area administrators, requiring new mappings to be defined.

Parent FSD

System Administration – MiEdWorkforce Admin Users

---

## 12.2 System Administration_Landing Page

The system must support the creation of a public landing page for MiEdWorkforce landing page and applicable links [Focus for this BRD]. This administrative function allows the MiEdWorkforce System Administrator (a shared MDE/CEPI role) to maintain/update the public entry point to the system. Maintenance includes adding and updating hyperlinks and associated text, and defining and managing page text/verbiage, such as Headings, Text/Info/Announcements, and User Support materials.

Parent FSD

System Administration – MiEdWorkforce Admin Users

---

## 12.3 System Administration_Admin Update Links

The system must support the ability of the administrator to update all the links and administrative newsletters/messages displayed on the internal user dashboard. This capability is managed by the MiEdWorkforce System Administrator (Shared MDE/CEPI role) and includes maintaining/updating hyperlinks, headings/subheadings, text/verbiage (for announcements/newsletters), and system messaging/notifications directly on the dashboard structure [1-3]. This administration ensures that internal users receive relevant and timely information immediately upon logging into the system.

Parent FSD

System Administration – MiEdWorkforce Admin Users

---

## 12.4 System Administration_Data Dictionary

This feature ensures that the system provides a comprehensive data dictionary/reference manual that the MiEdWorkforce System Administrator can maintain and update in real time. The system will be able to output this documentation in a consumable format. The reference manual must include details such as database mappings, field dependencies identified, user guides, and process flow mappings and diagrams (for the code). The maintenance of this reference manual is intended to support the administration of all other features of the system.

Parent FSD

System Administration – MiEdWorkforce Admin Users

---

## 13.1 AdminScreens

This feature provides theEPP  Admin users with the ability to add and manage Educator Preparation Providers with tools such as Search and Worklists.

---

## 13.2 Historical Data

This feature provides the MiEdWorkforce user with EPP Admin permissions the ability to view the historical data for all Active or Closed Educator Preparation Providers (EPPs). The retention period for this historical data, which includes Pathways, Certificates, and Endorsements, will be defined.

---

## 13.3 Work list

The feature allows the MiEdWorkforce user with EPP Admin permissions to create and manage worklists for Educator Preparation Providers. Functionality includes setting task status, deadlines, defining access levels, and managing the workflow for reviewing and approving worklist items.

---

## 13.4 Email,Comm

This feature utilizes the Email Communications Framework (Epic 6.0) to allow the EPP Admin to create and manage email templates for EPPs. It includes setting effective dates, defining authorized recipients and triggers, and viewing historical usage data of the templates. The EPP Admin can also resend historical emails with a CC option.

---

## 13.5 Dashboard

The EPP Admin will manage the EPP Dashboards by defining available configurable tiles, setting access levels for EPP users, defining reports for role-based dashboards, and setting the availability of necessary forms and worklists on the dashboard.

---

## 13.6 Document upload

This feature provides an administrative page for the EPP Admin to manage system configurations for processing uploaded files and data sets received through file upload or API data input. The admin can view file queues, status details, and perform management actions like resetting stuck files or deleting files in the queue.

---

## 13.7 Identity Management Integration

This feature was originally created to support building match logic, creation or retrieval of Unique IDs, and an admin screen for Identity Management related to Educator Preparation Providers. However, the system design has since captured the full system requirements for identity management within Identity Management Integration (15.0), and thus this feature (13.7) is no longer necessary.

---

## 13.8 ProPrep - Set up & Maintenance

This feature encompasses administrative actions related to the Pro Prep public page, allowing EPP Admin to manage guidance text, alerts, communications, and toggle program availability. It also covers the management of EPP Reports/Report Settings and defining Question Sets and validations for EPP processes.

---

## 14.1 Admin Screens

This feature provides the MiEdWorkforce System Administrator with the Professional Learning (PL) Admin role the capability to manage and maintain core Professional Learning system pages, applications, reports, availability, and underlying configurations. This administrative page allows for maintenance of public resources and application forms.

---

## 14.2 Historical Data

This feature ensures that the MiEdWorkforce system includes historical Professional Learning data, encompassing Sponsor Program details, Educator attendance records, SCECH history, and administrator processing details. This historical context is important for processes like credential renewal.

---

## 14.3 Worklist

This feature allows the Professional Learning Admin to create and manage the professional learning worklists used by Professional Learning Admin Processing staff and Sponsors. This includes configuring the display of tasks, statuses, and visual progress indicators. The admin can also define new hold statuses for workflow management (e.g., hold document review).

---

## 14.4 Email,Comm

This feature leverages the system's framework (Epic 6.0) to allow the Professional Learning Admin to create, manage, and configure system alerts, notifications, and email templates. This includes setting effective dates, managing versioning, defining authorized user roles, and tying communication to specific system events/triggers.

---

## 14.5 Dashboard

This feature provides an administrative page for the Professional Learning Admin to manage and configure the dashboards related to Professional Learning, ensuring role-based access control and configurable tile availability for Educators, Sponsors, and Coordinators. Audit details related to dashboard configuration must be stored.

---

## 14.6 Document Upload

This feature grants the Professional Learning Admin the ability to manage file upload availability and oversee the system configurations for processing bulk files and data sets (both file upload and API input). Crucially, this feature allows the Admin to view file queues, filter files, and manage processing issues like resetting or deleting files that are "stuck".

---

## 14.7 Identity Management Integration

This feature defines the system logic required to award State Continuing Education Clock Hours (SCECHs) to session attendees by utilizing Unique ID identity matching. This process is triggered when professional learning records (such as attendee lists) are successfully processed.

---

## 15.1 Update Demographic,Personal Data (Business User)

This feature covers the district user functionality of updating the demographic data associated with an established Unique ID. The process involves the user updating data in an external HR system, transferring data to MiEdWorkforce via the online interface, sending validated data to Mi-Key for updates, and updating the person record upon successful completion. This process will not result in New ID creation.

---

## 15.2 Add New Record (Business User Request for ID)

This feature covers the district user functionality related to obtaining an existing state education ID and/or creating a new state education ID record, required for data processing. The process culminates in the user receiving a unique ID based on a Mi-Key match, no match (new ID creation), or near match resolution.

---

## 15.3 ID Resolution

This feature defines the process when the Mi-Key identity service returns a “Near Match” result (score >X<%), requiring manual resolution or administrative review to ultimately assign or update a Unique ID. The process and permissions vary significantly between Business Users and Citizen Users.

---

## 15.4 Person,ID Search

This feature details the ability for Business and Admin users to search for and review associated identity data established within MiEdWorkforce. The search function supports various search criteria, partial matches, display of retired IDs (if searched directly), and provides access to actions like viewing history, adding an employee, or requesting identity management actions (linking or splitting IDs).

---

## 15.5 Bulk,API Identity Processing

This feature allows Business and Admin users to submit identity-related data for updates for multiple individuals in a single submission via bulk file upload or API input. This process is restricted to performing updates to demographic data associated with existing Unique IDs. This processing uses an asynchronous Mi-Key Assignment service to perform matching.

---

## 15.6 Link IDs

Linking involves merging two or more identity records (Unique IDs) that belong to a single person (Mi-Key refers to this as retire and merge). This administrative process is initiated by a user request, requires administrative approval, results in non-primary IDs being retired, and all associated MiEdWorkforce data being merged to the designated primary Unique ID.

---

## 15.7 Historical records

This feature was designed to account for MiEdWorkforce needing to connect to historical "Person" data. However, it has been determined that MiEdWorkforce will not have a need to store historical “Person” data, and thus this feature is no longer necessary.

---

## 15.8 Citizen Account Creation

This feature provides a streamlined process for new citizen users to create an account in the MiEdWorkforce system. The workflow begins after a user authenticates through MiLogin and identifies as a 'Citizen'. The user completes a registration form, and the submitted data is cross-referenced with the MiKey identity management system to prevent duplicate records. The system is designed to handle three outcomes from this identity verification: a definitive Match, a No Match (resulting in a new user), or a Near Match, which triggers a manual review and resolution workflow by an Identity Administrator to ensure data integrity.

---

## 15.9 Admin Identity Processing

This feature details the administrative functions required to manage identity processing. Admin Users process end-user requests (New ID, Link, Split, Citizen Near Match resolution) via a dedicated Worklist, manage system configurations (business rules, user support materials), monitor processing status (file/API queues, Mi-Key batches), and perform direct manual person record updates.

---

## 15.10 Citizen Account Updates

This feature allows citizen users to update demographic data associated with their established Unique ID. Updates are subject to system validation and probabilistic matching via the Mi-Key Assignment service. Crucially, updates to "Primary Demographic" fields are restricted if the user has active employment, requiring intervention by a district user.

---

## 15.11 Split IDs

Splitting IDs resolves the scenario where a single Unique ID is used for multiple individuals. The process involves users requesting admin assistance, admin indicating which history/associated data belongs to the new ID, Mi-Key splitting the records, and MiEdWorkforce updating its data accordingly.

---

## 15.12 Retire IDs

Retiring an ID removes it from active Mi-Key tables for IDs that do not represent a real person or have fake data. This administrative function is intended for IDs that have not been consumed in reporting and have no associated MiEdWorkforce data. This functionality is TBD for MVP as direct Mi-Key access is currently available for administrators.

---

## 15.13 Email,Communications

This feature details the system requirements for the Identity Management Administrator to configure and maintain email templates and user communications related to identity processing. The functionality is built upon an existing email communication framework and allows admins to control content, trigger points, recipient roles, and history.

---

## 15.14 ID Migration & Conversion

This feature was created to address potential needs for MiEdWorkforce to migrate and convert "Person" data. It has been determined that MiEdWorkforce will not have additional migration and conversion needs beyond those handled by the Mi-Key and Data Warehouse teams, making this feature no longer necessary.

---

## 15.15 Citizen File Upload

This feature was created to allow citizens to upload supporting documentation during the account creation process. However, during process definition, it was determined that document upload was not a necessary component for processing, and thus this feature is no longer necessary.

---

## 16.1 Pending credential application review and process

This feature encompasses the processes by which credential applications submitted by Citizen or Business users are routed to the appropriate MiEdWorkforce worklists (e.g., OEE worklist) based on credential type, status, PPR flags, and Needs Responses flags. Users with processing roles can then access the applications, view relevant data (MTTC scores, convictions), and take defined actions to move the application through the workflow.

---

## 16.2 Assigning Certificate status

This feature defines how credential application statuses are managed throughout the workflow (Submitted, Approved, Denied, Pending Payment, etc.). It covers the system's ability to auto-approve applications meeting all criteria, and grants the Credential Administrator role the ability to manually issue, suspend, revoke, or nullify certificates/credentials/endorsements.

---

## 16.3 Communication

This feature ensures defined users are communicated with via emails and alerts throughout the application workflow, notifying them of submission, status changes (e.g., Pending Payment), or requests for additional documentation. It also provides MiEdWorkforce Credential Administrators the tools to enter and manage internal comments (for authorized users only) and external comments (viewable externally).

---

## 16.4 Historical records

This feature ensures that authorized users can view an applicant's profile details, including historical and current data related to Certificates, Permits, Enrollment information, and Conviction information. Additionally, it supports the management of uploaded supporting documents, allowing authorized users to view, download, print, and export these files.

---

## 17.1 Temporary Credentials Create

The Temporary Credentials Create feature supports the process where a school district, Intermediate School District (ISD), or staffing agency user applies for various temporary educator permits on behalf of an individual who may or may not hold a current teaching certificate. The system guides the user through application questions, validates prerequisites (e.g., PPR status, educational requirements), and ensures appropriate endorsements and grade levels are designated for the assignment. The application determines eligibility for new permits or renewals and sets the resulting status (e.g., Pending PPR, Pending Evaluation).

---

## 17.2 Temporary Credentials User Feedback

The Temporary Credentials User Feedback feature ensures that users (district/ISD staff) and the affected educators receive timely, relevant, and actionable information regarding the status of a submitted temporary credential application. This includes displaying required next steps, such as the educator needing to complete Professional Practices Review (PPR) responses, payment submission, or noting that the application is under Conviction Review or Evaluation. Users must be able to verify the status of any permit via their MiEdWorkforce account.

---

## 18.1 Add New Position

This feature enables MiEdWorkforce users with the School District role to create new position records within the organizational chart (org chart) using the online interface by providing required position data elements. Users will also have the option to duplicate existing positions for streamlined creation.

---

## 18.2 Edit,remove position

This feature allows the MiEdWorkforce user with the School District role to manage established positions by editing details (e.g., renaming) or updating the Job Position Status to reflect changes like filling, freezing, or canceling a position. It also defines the mechanism for removal, distinguishing between deletion in a working window and cancellation (end-dating) of established positions.

---

## 18.3 Create new collection

This feature encompasses the actions and system behavior associated with transitioning the Position Roster data from one academic year to the next. This involves displaying the previous Roster data, carrying forward active positions, and maintaining the historical organizational chart with audit details.

---

## 18.4 Position Roster Certification

This feature defines the formal process used by the School District User to finalize the Position Roster submission for a required interval. The process includes tracking workflow steps, completing pre-certification requirements, running Quality Review validations, providing justification for certain errors, and performing attestation and e-signature.

---

## 18.5 Data Input via API and File Upload

This feature allows School District Users to update or modify the established organization chart (Position Roster) using non-online methods, specifically API integration and bulk file upload. This functionality supports updating individual position components and performing mass status/date changes.

---

## 18.6 Email,Communication

This feature details how the MiEdWorkforce system utilizes email notifications and system alerts to communicate vital information to the authorized School District User regarding the Position Roster submission process, including deadlines, data quality issues, and required user actions.

---

## 19.1 Submission Reports and Data Review

This feature provides authorized users (primarily at the District level) with the ability to review and interact with the results of Data Quality (DQ) checks performed by the system or triggered ad-hoc. Users can acknowledge DQ checks, provide justification for offending data, access individual records for correction, and review historical DQ status.

---

## 19.2 Data Processing- Business Rule Validations

This feature enables the Data Quality (DQ) Administrator to define, manage, version, and schedule Data Quality Checks (DQC) within the system. These checks analyze submitted data against business specifications, historical data, and other system data at various aggregation levels to identify violations.

---

## 19.3 Dashboard

The Data Quality Dashboard serves as the central landing page for users to interact with and review Data Quality (DQ) check results and assigned Action Items related to their submitted data. It displays dynamic reports and summary information tailored to the user's organizational level and role.

---

## 19.4 Email,Communication

This feature provides the Data Quality (DQ) Administrator with tools to manage communication templates and alert definitions. The system automatically notifies submitting organizations when DQ issues are identified and assigns necessary Action Items.

---

## 19.5 MiEdWorkforce ADA Compliance

This feature ensures that the entire MiEdWorkforce system and all its components (UI/UX, interfaces, data display, and accessibility) adhere to necessary compliance standards, specifically The Americans with Disabilities Act (ADA) and WCAG 2.0 ADA Guidelines.

---

## 20.1 CEPI comments and flags

This feature provides a standardized mechanism for users, particularly state staff, to enter comments (justifications) on datasets or records. Comments can be designated as internal (viewable only by select state staff with appropriate privileges) or external (viewable with any level of permissions). Additionally, flags may be set on specific data conditions to convey status or create "Action Items" for users, roles, or worklists, utilizing the Alerts, Emails and Communications framework. All actions related to comments and flags will be stored with audit details.

---

## 20.2 MDE comments and flags

This feature provides a standardized mechanism for users, particularly state staff, to enter comments (justifications) on datasets or records. Comments can be designated as internal (viewable only by select state staff with appropriate privileges) or external (viewable with any level of permissions). Additionally, flags may be set on specific data conditions to convey status or create "Action Items" for users, roles, or worklists, utilizing the Alerts, Emails and Communications framework. All actions related to comments and flags will be stored with audit details.

---

## 21.1 Add New Employee

This feature allows a business user with authorized access to an educating organization (School or District) to add employment data details for individuals to the Employee Roster. The process establishes the initial employment record, requiring the Unique ID to have been previously established via Identity Management. The system validates the submission and sets a status (Pending, Error, or Active) based on completeness and successful validation.

---

## 21.2 Edit,Remove Active Employees

This feature allows an authorized user to edit the details of individuals already on the Employee Roster, including adding termination information (Employment End Date and Separation Reason) to mark the end of employment. All edits must pass validation as defined by the Staffing Data Admin.

---

## 21.3 Create a New Collection

The system establishes a process enabling an authorized business user to create a school year Employee Roster (collection). Based on collection availability defined by the Staffing Data Admin, the system automatically includes all active employee records from the previous school year while excluding records with a reported Employment End Date and Separation Reason.

---

## 21.4 Employee Roster Certification

The system allows an authorized user to certify the full Employee Roster, thereby finalizing the data set. Certification triggers a set of validation checks on all included records, which may generate warnings requiring acknowledgment or errors requiring correction before final submission.

---

## 21.5 Data Input via API and File Upload

In addition to online entry (system interface), the Employee Roster data can be input via File Upload and API service connections directly from HR systems. The system aims to leverage a shared upload functionality and maintain open endpoints for future, seamless HR system integration.

---

## 21.6 Email,Communication

The system must utilize customizable alerts and email templates defined by the Staffing Data Administrator to communicate relevant information to authorized users regarding submission progress, errors, warnings, and required actions.

---

## 21.7 Employee Audit

This feature establishes a workflow for users with the ISD Auditor role to perform district-level audits on constituent districts. It provides detailed employee, assignment, and credential data for monitoring, supports reporting audit findings using a customizable form, and includes processes for requesting documentation and finalizing/certifying the audit.

---

## 22.1 Instructional Staff

The Instructional Staff feature enables the School District user to update Employee Assignment Details for roles classified as "Administrator" or "Instructional". This includes ensuring the individual holds a valid credential or initiating the Temporary Credential Application process or providing a Credential Error Justification. It also incorporates tracking for New Teacher Monitoring and reporting Evaluation Outcomes.

---

## 22.2 Non-Instructional Staff

The Non-Instructional Staff feature enables the School District user to submit Assignment Details using defined CEDS elements for positions such as Director of Food Services or Administrative Support. These roles have fewer mandatory data elements compared to Instructional Staff, and credential checks are generally not required.

---

## 22.3 Student Support Staff

The Student Support Staff feature enables the School District user to submit Assignment Details using defined CEDS elements for positions such as Special Education Paraprofessionals. Certain positions within this category, such as Special Education Paraprofessionals, require mandatory submission of Education History.

---

## 22.4 Data Input via API and File Upload

The system must support updating Employee Status Details and Employee Assignment Details for the new academic year using bulk file upload and API methods. The file upload process utilizes a common file staging area and adheres to system file upload standards. The system also allows users to extract position roster data in various formats (e.g., Excel, CSV, XML).

---

## 22.5 Email,Communication

The system utilizes alerts and email templates, defined by the Staffing Data Administrator, to notify users of critical information. These communications cover upcoming deadlines, new data quality issues, required user actions, and updates related to Education History, New Teacher Monitoring, and Evaluation Outcomes.

---

## 22.6 Appropriate Placement Audit

The Appropriate Placement Audit supports a data quality review that investigates the accuracy of reported employee rosters and the appropriateness of instructional staff placements. This process involves ISD Auditors reviewing district submissions (including School Safety/RAPBack and credential verification), documenting findings using a customizable form, and submitting the certified report for SOM Auditor review.

---

## 24.1 Create New Collection

This system feature ensures every employee added to the MiEdWorkforce system has the correct Unique ID through Mi-Key integration. A Staffing Authorized User utilizes the established Employee Roster to perform Unique ID matching and assignment to an individual. The process begins with selecting "Add New Employee," navigating to a Person Search page, and entering search criteria. Based on Mi-Key services, the result returns a probabilistic match of Match, No Match (New ID Created), or Near Match (Requires Resolution). Upon successful assignment of the Unique ID, the demographic data fields become read-only, necessitating an explicit "Update Demographics" action for subsequent changes. The identity data set will be maintained within the larger employment collection and will not be collected within a separate, defined "Request for ID Collection".

---

## 24.2 ID Matching and Assignment

This system feature ensures every employee added to the MiEdWorkforce system has the correct Unique ID through Mi-Key integration. A Staffing Authorized User initiates the process via the Employee Roster by selecting "Add New Employee," which directs them to a Person Search page. The user enters search criteria meeting minimum requirements. The system utilizes Mi-Key services to perform probabilistic matching, returning a result of Match (Unique ID found), No Match (New ID Created), or Near Match (Requires Resolution). If a Match is returned, demographic data is pre-populated. If No Match is returned, the user completes a blank form to trigger new ID creation. Upon successful ID assignment (Match or New ID Created), the user can proceed with reporting employment details.

---

## 24.3 ID Resolution

The Near Match Resolution process is triggered when a Staffing Authorized User attempts to either Request a new Unique ID (Add New Employee) or Update an existing Unique ID's demographic data, and the Mi-Key service returns a probabilistic match score that falls between the predefined match and no match thresholds (Near Match). When a Near Match is returned, the record is automatically set to a status of "Requires Resolution". The user must then access the resolution screen, typically via an Action Item on their dashboard, where they review the submitted data against potential matches. Based on the context (new ID request vs. update request), the user must choose whether to "Use this Potential" match, "Request New ID" (if applicable), or cancel the request. Requests for a New ID following a Near Match require providing a justification and are routed to the MiEdWorkforce Identity Administrator worklist for manual review.

---

## 24.4 Data Input via API and File Upload

This feature addresses the system's ability to consume identity-related data in bulk, via File Upload or API service connections. The system must be capable of processing these bulk inputs, applying robust business rule validations, and utilizing Mi-Key services (specifically the Assignment service) to perform probabilistic matching for updates to existing Unique IDs (demographic data). The goal is to apply business rule validations to data submissions regardless of the modality in which they were entered. API connections to local HR systems may largely perform this process outside of MiEdWorkforce, requiring specific interfacing specifications.

---

## 24.5 Email_Communication

This feature ensures that all user actions related to identity management (such as Request for ID, Update Demographics, and Link/Split/Retire IDs) result in the use of defined email templates and triggers, alerts, and communications as managed by the MiEdWorkforce Identity Admin. The system must support the Identity Management Administrator in creating, managing, and setting triggers for these communications, and delivering notifications to relevant parties (Business Users, Citizen Users, and affected school districts) upon critical identity events. The system will use the SendGrid API to create the email notification service.

---

## 25.1 Educator Preparation Provider workflow search (worklist)

This feature allows the MiEdWorkforce user with Education Preparation Provider (EPP) permissions to view and interact with a worklist of credential applications. The worklist includes applications for Teaching, School Psychologist, School Counselor, School Administrator, School Social Worker, Additional Endorsements, and Approvals. Users must be able to search, filter, and view application details, including applicant personal information and MTTC results.

---

## 25.2 Track Candidates

This feature provides EPP users with a Candidate Tracking worklist, allowing them to switch between Pending Verification and Verified Candidates lists. Users can accept or reject enrollment for candidates awaiting verification. For verified candidates, users can add new programs, update enrollment statuses (e.g., Enrolled, Student Teaching, Exited), and view detailed candidate-level enrollment information.

---

## 25.3 Recommending Candidates

This feature defines the actions available to an EPP user when reviewing a credential application (Teacher, School Psychologist, etc.) or an Approval application. Actions include Hold, Deny, Cancel, and Recommend. The Recommendation process specifically allows EPP users to add or inline edit endorsements, and initiates the next step in the application approval process.

---

## 25.4 Document upload

This feature provides functionality for both EPP users and MiEdWorkforce System Admins (with EPP permissions) to upload bulk files containing teacher candidate tracking data. Additionally, System Admins are able to download and extract this submitted data. This functionality is intended to streamline the entry of large numbers of candidate records.

---

## 26.1 Sponsor Application

The Professional Learning Sponsor Application process digitizes the manual 2024 SCECH Sponsor Application Form. The system requires new Professional Learning Sponsors to complete this form, including naming a Coordinator and/or Assistant Coordinator. Upon completion, the application is routed to the MiEdWorkforce Professional Learning Admin for review and processing.

---

## 26.2 Program Applications - Create,update

The feature provides a structured application form (SCECH Program Application Template/Wizard) to capture all necessary details about a professional learning program, including type, description, dates, fees, and documentation. The sponsor must review a summary and agree to attestations before submission, which routes the application for administrative processing. Any subsequent change to an approved program must also be routed through the approval process.

---

## 26.3 Document upload

This feature enables Professional Learning Sponsors to upload supporting documentation, such as the Program Agenda, when submitting a Professional Learning Program application. The system must ensure that file types adhere to standards defined by the Professional Learning Admin.

---

## 26.4 Professional Learning Catalog

This feature ensures that Sponsors can designate whether their professional learning program should be displayed in the public catalog (searchable by educators). This designation is made via a mandatory Y/N indicator within the Program Type section of the Program Application.

---

## 26.5 Awarding Professional Learning Hours

Once a program is approved, the Professional Learning Sponsor manages the attendance roster by adding attendees (online or via bulk upload), adjusting the awarded SCECHs (single or bulk, default is maximum). The sponsor must then review attendee details, agree to attestations, and certify the final roster, triggering the official award of SCECHs and attendee notification.

---

## 26.6 Worklist

The system provides a worklist displaying sponsor applications (26.1) and program applications (26.2) that require attention or have specific statuses assigned by the Professional Learning Processing Administrator (28.0).

---

## 27.1 Registration

This feature enables the MiEdWorkforce Citizen User to view the link to the respective sponsor program’s website, allowing them to register for currently offered or future Professional Learning programs, as registration is handled externally. The system also stores details of this external registration for internal reporting on potential attendees.

---

## 27.2 Worklist

This feature provides the MiEdWorkforce Citizen User the ability to submit required evaluations after attending a program in order to receive Professional Learning credit hours, and to submit queries to the Professional Learning Administrator regarding any perceived inaccuracies in awarded credit hours.

---

## 27.3 Communication

This feature ensures that users who have attended Professional Learning programs are reminded to submit their required evaluation through both dashboard alerts upon login and automated email notifications. The timing of the email notifications is configurable by the Professional Learning Administrator.

---

## 28.1 Process vendors

This feature provides the Professional Learning Administrator with the tools necessary to review, view supporting documentation, and take action (Approve, Reject, Edit, Delete, Comment, Email) on pending applications submitted by potential Professional Learning Sponsors and Vendors.

---

## 28.2 Publishing the Professional Learning Catalog

This feature allows the Professional Learning Administrator to search for, view, and edit the established Sponsor entities and the list of approved Programs, forming the basis of the Professional Learning Catalog.

---

## 28.3 Document upload

This feature ensures that Administrators can access and review documents (such as Program Agenda Details or other supporting documents) submitted by applicants, enabling proper validation during the review process.

---

## 28.4 Program Maintenance

This feature provides the Professional Learning Administrator with comprehensive tools to maintain approved programs, including editing program details, managing event dates/locations, tracking changes via an activity log, and maintaining foundational program metadata (categories, descriptions, evaluations).

---

## 28.5 Worklist

This feature provides the Professional Learning Administrator with dedicated dashboard views (worklists) to manage pending Sponsor/Vendor Applications and Program Applications, offering robust search and filtering options to efficiently organize and locate items requiring review.

---

## 29.1 Credential landing page - Certificate

This feature details the requirements around the necessary tools and services provided to the citizen population of educators with certificates or as a candidate enrolled in an educator preparation program, specifically focusing on the initial landing page/dashboard interaction and customization.

---

## 29.2 All types (Apply,Renew)

This feature describes the process by which a MiEdWorkforce citizen user can select "Apply/Renew" from the landing page and initiate an application flow for various certificate types (Teaching, Interim Teaching, School Counselor, School Administrator, School Psychologist, and School Social Worker), including system validations and routing.

---

## 29.3 Certificates - Additional Endorsements

This feature describes the process by which a MiEdWorkforce citizen user can add an additional endorsement to an existing, eligible certificate, covering the specific input requirements, validations (including MTTC scores), and application routing for Teacher (including CTE) and School Administrator roles.

---

## 29.4 Certificates - View and Print

This feature enables citizen users to access and manage documentation and status information related to their credentials, including the ability to print approved certificates and cover letters, view pending application statuses, enrollment details, and MDE guidance documents.

---

## 30.1 Rapbacks Search - Search Results

This feature provides a dedicated administrative page where the Rapback Admin can search and filter retrieved Rapback records based on various criteria and view the resulting matched records in a sortable table format.

---

## 30.2 Rapbacks Detail - View Rapsheet Details

This feature provides the workflow for navigating from the Rapback search results to the individual record detail page, where the Rapsheet can be dynamically generated and displayed via a service call to the MSP system.

---

## 30.3 Rapbacks Search - Bulk Processing Buttons

The system provides bulk actions—Notify School District, Refer to Professional Practice Review (PPR), and No Referral Required—that can be applied to a selection of Rapback records from the search results page.

---

## 30.4 Rapbacks Detail - Notes and Notes History

The Rapback record detail page provides an interface for the administrative user to input and save free-text notes detailing actions taken or relevant findings during the review process. This history provides traceability and context regarding the disciplinary path (PPR) or resolution.

---

## 30.5 Rapbacks Search - File Downloads

The system supports two primary exports: the SID Review File (for internal review of employment/termination data) and the District Notification File (for notifying school districts of Rapback status).

---

## 30.6 Rapbacks Detail - Action(s) Taken

On the Rapback detail page, the user can view an audit trail of all status updates (e.g., Notified School District, Referred to PPR, No Referral Required). Key actions are presented as clickable hyperlinks that display the associated output, such as the content of the notification email or the PPR work item created.

---

## 30.7 Rapbacks - Employment,Credential Checks

When a Rapback notification is received and matched to a unique ID, the system automatically runs checks against the Employee Roster, Assignment Details, and Credentialing data to set the 'Is Employed' and 'Has Credential' flags, along with recording associated details.

---

## 30.8 Rapbacks Detail - Alias Names

The Rapback record detail page includes functionality to add and edit alias names, storing a historical record of all aliases used by the individual associated with the Rapback.

---

## 31.1 Search Function

The Public Credential Search allows any member of the public to search the MiEdWorkforce system for educator credentials without requiring a login. The system provides a basic free-form search and an advanced search that includes criteria such as Credential Number and Credential Type. The search logic must handle special characters, partial matches, and filter results based on defined certificate and temporary credential statuses.

---

## 31.2 Public Search - Results Detail

Following a successful search (31.1), the system must display initial results and allow users to drill down into the full Credential History. This feature includes requirements for sorting results, presenting details clearly, and handling the specific display rules for credentials associated with adverse actions (e.g., Nullified, Revoked, Suspended).

---

## 31.3 Public Search - Legacy Reports

This feature was originally defined to provide public access to legacy reports (L2K and SCR) for Special Education Approvals issued prior to 2014, previously available in MOECS. It has been determined that this functionality will not be migrated or carried into MiEdWorkforce.

---

## 31.4 Public Search - ProPrep

The ProPrep search provides visibility into approved education preparation providers and their current program details. Users can navigate to the search page without login and choose to search specifically by Provider or by Program, using search fields and filters such as Program Type, Pathway, EPP Type, and City.

---

## 31.5 Public Search - Professional Learning

The Professional Learning Search supports both basic and advanced search methods for finding SCECH sponsors and programs with an "Active" status. Advanced search options include detailed geographical filters, category filtering, date ranges, and maximum SCECH hours. The results grid provides dynamic, real-time data with hyperlinked names to view sponsor and program details.

---

## 32.1 Application standards

The system must maintain interface and behavior standards. Application behaviors should be consistent with the State of Michigan (SOM) standards for UI/UX, unless overridden by documented business needs. Explicit business approval is required for deviations from cybersecurity standards, based on documentation explaining the increased risk and the business owners deeming it acceptable. All deviations must be linked to business needs and either requested or approved by business owners. Documentation must be maintained for use during the Authority to Operate (ATO) process. For all SOM standards, the corresponding document will govern the required behavior or configuration. For any outstanding items, business owner guidance and approval are required.

---

## 32.2 Device UI,UX

The Device UI/UX feature requires the MiEdWorkforce system to be accessible and operational on mobile devices. This functionality has a particular focus for educators and non-certified staff. The system must adhere to the State of Michigan Digital Standards. No code can be made "Live" without an approved review from DTMB's eMichigan. The DTMB eMichigan review acts as the approval of all Digital Standard acceptance.

---

## 32.3 Auditing

The system must be able to mark data as inactive or “delete” in accordance with required audit and retention schedules. The application must allow administrator users to review recent actions and data changes. In general, no data should be permanently deleted; instead, data should be soft-deleted or marked for deletion. Exceptions apply only for purely transactional data meant for proper system function, files erroneously uploaded by users, or similar cases preapproved or requested by business. Where appropriate, the system should allow for reconstruction of historical data. Auditing must capture detailed changes (micro-level changes).

---

## 32.4 Database

The system must utilize the latest CEDS Integrated Data Store (IDS) as the primary database for educator information storage and retrieval. While CEDS IDS is currently SQL Server based, the State of Michigan is considering converting to a NoSQL DB with a JSON-LD schema (likely Azure Cosmos DB). The State retains the right to determine the final data model (CEDS IDS or CEDS JSON-LD schema). Modifications made to the data model must be approved by the State team.

---

## 32.5 Software (Frameworks)

The system must be built using current industry standards for maintainability and upgradability. The State prefers the following frameworks for delivery:

1. Latest .NET LTS at time of delivery (likely to be 10).

2. Source code primarily in C#, ANSI SQL/T-SQL, and JavaScript (TypeScript preferred).

3. If Angular, Vue, or similar SPA technologies are used, they must be on the latest LTS version that the State supports.

---

## 32.6 Software (Industry Standards and Open-Source Approval)

All technology used in the MiEdWorkforce solution must be industry standard. Any open-sourced software utilized must be formally approved for use by the State. For any approved non-standard stacks, appropriate maintenance arrangements must be documented.

---

## 32.7 Software (Licensing)

All technology used must be appropriately licensed (Contractor or public domain). All licensing requirements, regardless of whether they incur costs or not, must be properly documented and submitted for review by business owners. For paid licenses, terms must be documented for any future maintenance and costs. For non-paid licenses, terms must be documented to ensure compliance.

---

## 32.8 Software (Security Updates)

Technology used must support a mechanism for quick updates, especially security-related, relative to software industry vulnerability categories (e.g., OWASP defined) and remediation speed. Results of security scans or audits must be made available to appropriate SOM employees on an ongoing basis. The system must ensure no deprecated software components are in use in main application parts, and security scan results must be in line with expected SSP guidelines.

---

## 32.9 Software (Error Reporting)

Software application generated errors must be reported in near real-time. These errors must be addressable (corrected by administrators) in suitable online and/or batch options. Industry standard logging and interfaces must be supported by the application for easy accessibility and ingestion by SOM logging platforms like Dynatrace, QRadar, etc.. Application health dashboards must be fully functioning and display critical information including running processes, CPU load, error messages, and exception logs.

---

## 32.10 Software (Integration Capability)

The solution must allow for integration with State hosted and non-hosted data sources. Industry standards must be supported and preferred for data exchange (for example, REST APIs, SFTP), and reasonable effort must be made to support legacy or custom SOM solutions. The software and application must integrate with SOM existing systems and environments.

---

## 32.11 Data migration and conversion

The Contractor must convert and migrate all identified data from the MOECS and REP systems throughout the development life cycle to the production and lower environments. The system must be able to import historical data, forms, and documents from internal and other systems for Rap backs, professional learning, Educator Preparation Providers (EPPs), and districts and support agencies. This migration effort must not duplicate data being handled by the CEPI Data Warehouse efforts (e.g., staffing, certificate data, MTTC) or the My Key project (person data). All data in MiEdWorkforce must be stored in a CEDS-compliant format.

---

## 32.12 Integration

The Contractor must provide an implementation/engagement exit plan that guarantees the State of Michigan (SOM) always retains an operational system, regardless of future updates from the Contractor (subject to licensing and contract terms). The system must be built on an environment compatible with SOM standards, or easily migratable/reconfigurable. Integration requires vendor staff availability to work with SOM employees. Post-deployment, the system must undergo verification including basic configuration audits, interfaces testing, and security audits (DAST, SAST, SCA) to ensure no critical, high, or medium vulnerabilities are found.

---

## 32.13 Reporting

The solution must integrate with analytics tools such as Power BI and SAS. All MiEdWorkforce data (CEDS or otherwise, hot/cold storage, online status) must be accessible via the reporting platform. Appropriate measures (indexing, data replication) must be taken for adequately performant data queries. All legacy reports must be available in the new system. Crucially, reporting must not interfere with overall system performance; datasets for long-running queries must reside in separate but synchronized storage. Report creation and viewing must be embedded in MiEdWorkforce.

---

## 32.14 Batch processing

The system must support the State Admin's ability to create, manage and run ad hoc and scheduled batch jobs with the capability to view history for audit controls. Batch processing is critical for parsing external files (like CEPAS payment files and MTTC test score files). Batch processing must work in accordance with other related SOM systems and must not interfere with or degrade external processes. User authentication for the scheduling system must be consistent with other MDE/CEPI systems (e.g., SSO login, ADM accounts).

---

## 32.15 Worklist

The system must provide State users the ability to view and manage work queues/worklists within various programs. MiEdWorkforce will provide role-based worklist access and assignments with respect to the business process. State Admins must be able to configure and allocate privileges to manage assignments with full audit controls. Specific worklists are required for program areas including Credentials, Education Preparation Providers (EPPs), Professional Learning, Professional Practice/Rapbacks, Staffing data collection, and Identity management. Worklists must be accessible from the user Dashboard based on access permissions.

---

## 32.16 Electronic Document Management

The system must support uploading and downloading various documents collected into categories, with version controls for retrieval, auditing, and processing for worklist/batch management, and bulk management. The system must allow for integration with malware scanning tools (ransomware, virus, spyware). Appropriate security measures, including strong encryption, RBAC controls, and access logs, must be implemented for different document types. Granular record retention rules (and legal holds) must be implemented for all document types.

---

