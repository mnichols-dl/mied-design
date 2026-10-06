## Scheduled Orchestrations

### Azure Synapse Pipelines

**IAM Domain Pipelines:**

| Pipeline                             | Schedule                      | Purpose                                                       | Trigger Mechanism                                                         |
| ------------------------------------ | ----------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Authorization Request Expiration** | Nightly at 2 AM               | Mark expired authorization requests and trigger notifications | Synapse scheduled trigger -> POST `/admin/jobs/expire-requests`           |
| **Monthly Inactivity Deactivation**  | First Friday of month at 3 AM | Deactivate inactive user accounts                             | Synapse scheduled trigger -> POST `/admin/jobs/deactivate-inactive-users` |
| **Weekly Reports Generation**        | Monday at 8 AM                | Generate and email weekly authorization reports               | Synapse scheduled trigger -> POST `/admin/jobs/generate-reports`          |

**Organizations Platform Capability Pipelines:**

| Pipeline                   | Schedule            | Purpose                                                                                                               | Trigger Mechanism                                                                                             |
| -------------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **CEPI Organization Sync** | Nightly at 1 AM EST | Pull org data from CEPI CEDS JSON-LD API; update local replica; rebuild materialized hierarchy; publish change events | Synapse scheduled trigger -> CEPI API pull -> ETL into `organizations-db` -> POST `/admin/jobs/sync/complete` |

**Communications Domain Pipelines:**

| Pipeline                | Schedule            | Purpose                                                                                  | Trigger Mechanism                                                                                                  |
| ----------------------- | ------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Email Content Purge** | Nightly at 2 AM EST | Permanently delete email content from Cosmos DB after retention period (default 90 days) | Azure Synapse Pipelines -> Query SQL for eligible instances -> Delete from Cosmos DB -> Update SQL metadata status |

**Documents Domain Pipelines:**

| Pipeline                 | Schedule        | Purpose                                         | Trigger Mechanism                              |
| ------------------------ | --------------- | ----------------------------------------------- | ---------------------------------------------- |
| **Hard Delete Job**      | Nightly at 2 AM | Permanently delete blobs after retention period | Azure Synapse Pipeline -> Direct blob deletion |
| **Staging File Cleanup** | Daily at 3 AM   | Delete staging files older than 7 days          | Azure Synapse Pipeline -> Direct blob deletion |

**Payments Domain Pipelines:**

| Pipeline                              | Schedule          | Purpose                                                            | Trigger Mechanism                                                               |
| ------------------------------------- | ----------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| **CEPAS Posting File Reconciliation** | Daily at 3 AM EST | Retrieve posting file, match transactions, update missing payments | Synapse scheduled trigger -> SFTP retrieval + reconciliation logic + DB updates |

**Professional Practice Review Domain Pipelines:**

| Pipeline                 | Schedule          | Purpose                                               | Trigger Mechanism                                                            |
| ------------------------ | ----------------- | ----------------------------------------------------- | ---------------------------------------------------------------------------- |
| **NASDTEC Nightly Sync** | Daily at 2 AM EST | Pull interstate disciplinary records from NASDTEC API | Synapse scheduled trigger -> REST API call + Mi-Key matching + local storage |

**Professional Learning Domain Pipelines:**

| Pipeline                      | Schedule            | Purpose                                                                    | Trigger Mechanism                                                                      |
| ----------------------------- | ------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **STARR College Course Sync** | Nightly at 1 AM EST | Pull completed college course data from STARR for SCECH credit eligibility | Synapse scheduled trigger -> ETL from STARR -> Store in Professional Learning Database |

**Staffing Domain Pipelines:**

| Pipeline                              | Schedule                  | Purpose                                                        | Trigger Mechanism                                                            |
| ------------------------------------- | ------------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **Collection Reminder Notifications** | Weekly during open period | Send reminder emails to districts with uncertified collections | Synapse scheduled trigger -> Query uncertified collections -> Publish events |

**Design Note:** Synapse Pipelines are used for scheduled batch orchestrations. They trigger API endpoints that perform the actual business logic, ensuring domain logic remains in the API layer. The EEM sync pipeline writes directly to the organization reference data database as it's a pure ETL operation. Email content archival and document hard delete are infrastructure operations (not business logic) and execute directly against storage. CEPAS posting file reconciliation and NASDTEC sync involve both ETL and business logic, so they combine direct data access with domain service calls.
