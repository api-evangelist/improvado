---
name: Connect a data source and run an extract
description: Authenticate to a workspace, pick a data source and connection, build an extract from a template, and run it on Improvado Embedded API v3.
api: Improvado Embedded API v3
base_url: https://embedded.improvado.io
operations:
  - POST /api/v3/token
  - GET /api/v3/datasources
  - GET /api/v3/datasources/{datasource_name}/connections
  - GET /api/v3/datasources/{datasource_name}/extract-templates
  - POST /api/v3.1/extracts/
  - POST /api/v3/extracts/{id}/run
  - GET /api/v3/extracts/{id}
---

# Connect a data source and run an extract

Use the Improvado Embedded API v3 to extract marketing data from a source into a data table.

## Auth
1. You are provisioned HTTP Basic credentials by Improvado. Use them (RFC 7617) to mint a
   workspace-scoped token: `POST /api/v3/token` with the target `workspace_id`.
2. Send the returned Bearer token as `Authorization: Bearer <token>` on every workspace-scoped
   call. Tokens expire after 30 minutes (renewed on use); on `401` re-mint a token.

## Steps
1. **List data sources** — `GET /api/v3/datasources` to find the `datasource_name` you want.
2. **List connections** — `GET /api/v3/datasources/{datasource_name}/connections`. If none exist,
   create one in the platform UI or via external authentication links.
3. **List extraction templates** — `GET /api/v3/datasources/{datasource_name}/extract-templates`
   to choose the template (`template_id`) and the fields it exposes.
4. **Create the extract** — `POST /api/v3.1/extracts/` with the template settings and the embedded
   account IDs (integers) that belong to a single connection.
5. **Run it** — `POST /api/v3/extracts/{id}/run`.
6. **Confirm** — `GET /api/v3/extracts/{id}` to read status and the `account_entity_id` /
   `account_embedded_id` identifiers.

## Conventions
- Pagination on list calls: `page` / `page_size` (default 100); read `count` / `next` / `previous` / `results`.
- Errors return `{"details": "..."}` with an HTTP status. `400` = validation (e.g. accounts spanning
  multiple connections); `401` = bad/expired auth; `403` = feature not enabled for the workspace.
- Every delivered data table includes system fields `account_id`, `account_name`, `__insert_date`.
