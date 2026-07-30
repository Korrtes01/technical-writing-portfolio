# TaskFlow API — Reference Documentation

## Portfolio Summary

This is documentation for a fictional API, TaskFlow — a simple task management service. Unlike the other two samples, there's no existing product or docs behind this one. It exists to show a different skill than a rewrite or a from-scratch doc for a real API: designing the *shape* of good documentation from nothing — deciding what endpoints a task manager needs, how auth and pagination should work, what a sensible error format looks like, and writing it the way a developer would expect to read it before the API is even built. This is a common real task: technical writers are often brought in during development, working from a spec rather than a finished product.

---

## Overview

TaskFlow is a REST API for managing tasks and task lists. It supports creating, updating, and organizing tasks, with support for due dates, priorities, and simple filtering.

**Base URL:** `https://api.taskflow.dev/v1`

All requests and responses use JSON. All endpoints require authentication except where noted.

## Authentication

TaskFlow uses API key authentication. Include your key in the `Authorization` header on every request:

`Authorization: Bearer YOUR_API_KEY`

Requests without a valid key return `401 Unauthorized`.

## Quick Start

Create a task:

Request: `POST https://api.taskflow.dev/v1/tasks` with header `Authorization: Bearer YOUR_API_KEY` and body: `{"title": "Review compliance checklist", "due_date": "2026-08-05", "priority": "high"}`

Response (201 Created): `{"id": "tsk_7f3a9c", "title": "Review compliance checklist", "due_date": "2026-08-05", "priority": "high", "status": "open", "list_id": null, "created_at": "2026-07-30T09:12:00Z"}`

## Endpoints

### List Tasks

Endpoint: `GET /tasks` — returns all tasks for the authenticated account.

| Query Parameter | Type | Description |
|---|---|---|
| `status` | string | Filter by `open`, `in_progress`, or `done`. |
| `priority` | string | Filter by `low`, `medium`, or `high`. |
| `list_id` | string | Return only tasks belonging to a specific list. |
| `limit` | integer | Max results per page. Default `20`, max `100`. |
| `cursor` | string | Pagination cursor from a previous response's `next_cursor`. |

Example: `GET /tasks?status=open&priority=high&limit=10`

Response: `{"data": [{"id": "tsk_7f3a9c", "title": "Review compliance checklist", "status": "open", "priority": "high", "due_date": "2026-08-05"}], "next_cursor": "eyJpZCI6InRzazU1In0="}`

### Get a Single Task

Endpoint: `GET /tasks/{task_id}` — returns `404 Not Found` if the task doesn't exist or doesn't belong to the authenticated account.

### Create a Task

Endpoint: `POST /tasks`

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | string | Yes | Max 200 characters. |
| `due_date` | string (`YYYY-MM-DD`) | No | |
| `priority` | string | No | `low`, `medium`, or `high`. Defaults to `medium`. |
| `list_id` | string | No | Assigns the task to an existing list. |

### Update a Task

Endpoint: `PATCH /tasks/{task_id}` — accepts any subset of the fields from Create. Only included fields are changed.

Example — mark a task done: `PATCH /tasks/tsk_7f3a9c` with body `{"status": "done"}`

### Delete a Task

Endpoint: `DELETE /tasks/{task_id}` — returns `204 No Content` on success. Deletion is permanent — there is no undo endpoint.

### Task Lists

Lists group related tasks (e.g. "Q3 Compliance", "Client Onboarding").

Endpoints: `GET /lists`, `POST /lists`, `GET /lists/{list_id}`, `DELETE /lists/{list_id}`

`POST /lists` accepts a single required field: `name` (string, max 100 characters). Deleting a list does not delete its tasks — their `list_id` is set to `null`.

## Pagination

All list endpoints use cursor-based pagination. When a response includes a `next_cursor`, pass it as the `cursor` query parameter to fetch the next page. A missing or `null` `next_cursor` means you've reached the last page.

## Errors

Errors return a consistent JSON body alongside a standard HTTP status code:

`{"error": {"code": "validation_error", "message": "title is required"}}`

| Status | Code | Meaning |
|---|---|---|
| `400` | `validation_error` | Request body failed validation (see `message` for the specific field). |
| `401` | `unauthorized` | Missing or invalid API key. |
| `404` | `not_found` | The resource doesn't exist or isn't visible to this account. |
| `429` | `rate_limited` | Too many requests. See Rate Limits below. |
| `500` | `internal_error` | Something went wrong on TaskFlow's side. Safe to retry once. |

## Rate Limits

- 100 requests per minute per API key.
- Current usage is returned on every response via headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (Unix timestamp).
- Exceeding the limit returns `429` with a `Retry-After` header (seconds until you can retry).

**Recommended client behavior:** back off using `Retry-After` rather than retrying immediately; don't poll `GET /tasks` more than once every few seconds for the same filter set.

## Notes

- All timestamps are ISO 8601, UTC.
- Task IDs are prefixed (`tsk_`), list IDs are prefixed (`lst_`) — useful for client-side validation before making a request.
- There is no bulk-create endpoint in v1. Creating many tasks requires one `POST /tasks` call per task; a bulk endpoint is a natural v2 candidate if usage patterns show demand.

---
*TaskFlow is a fictional API created for portfolio purposes, to demonstrate designing API documentation from a specification rather than an existing product.*
