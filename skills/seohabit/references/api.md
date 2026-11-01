# SEOHabit REST workflow

Base URL: https://seohabit.com/api/v1

Current public contract: https://seohabit.com/openapi.json

Send `Authorization: Bearer <project key>` on every request and `Content-Type: application/json` for JSON bodies. Keep the real key in the local environment variable `SEOHABIT_API_KEY` or an existing secret manager. Do not include it in chat, source, command-line arguments, URLs or logs. Use an HTTP client that rejects redirects instead of forwarding credentials.

## Read the selected project

This read-only example lists only the websites available to the key. It does not print authentication values or complete private account responses. Run it in an environment where the key has already been configured securely.

```sh
node --input-type=module <<'NODE'
const key = process.env.SEOHABIT_API_KEY;
if (!key) throw new Error('Configure SEOHABIT_API_KEY locally first.');
const response = await fetch('https://seohabit.com/api/v1/projects', {
  headers: { authorization: 'Bearer ' + key },
  redirect: 'error',
  signal: AbortSignal.timeout(30000),
});
if (!response.ok) throw new Error('SEOHabit returned HTTP ' + response.status);
const data = await response.json();
console.log(JSON.stringify(data.projects.map(({ id, name, url, subscribed }) => ({ id, name, url, subscribed })), null, 2));
NODE
```

`GET /projects` returns `{projects: [...]}`. Confirm the intended website rather than assuming a project from another conversation. Then call `GET /projects/{projectId}` with `project:read`. Its snapshot includes:

- `project`: the selected website and its current entitlement.
- `reports`: recent reports, newest first. A report has `id`, `created_at`, `complete`, and `result`, including `issues`, observation time, scope and coverage where available.
- `tasks`: task `id`, `type`, `status`, `details` and `artifact`. Use `type === 'improvement'` for the source-free workflow. Do not surface legacy source artifacts or unrelated private fields.

The snapshot may contain other workspace data. Use the fields needed for the user's task and avoid copying the complete response into a public artifact.

## Audit and follow its jobs

With `checks:write`, queue an audit:

```http
POST /projects/{projectId}/audits
Content-Type: application/json
Idempotency-Key: <new UUID for this audit>

{}
```

HTTP 202 returns `{id, status, accessToken, statusUrl}`. Treat `accessToken` as secret; the project key can poll without it:

```http
GET /projects/{projectId}/audit-jobs/{jobId}
```

The job has `status: queued | running | completed | failed`, plus `result` and `error`. If `result.nextJob.id` exists, poll that next project job. A completed batch may still have `result.coverage.status: continuing`. Coverage may also be limited; disclose the actual checked pages and unavailable checks.

Reread `GET /projects/{projectId}` for the saved report. Its issue indexes are positions in `report.result.issues`. Preserve those original indexes when displaying a filtered subset or combining findings across pages.

## Obtain instructions and save a task

For a verified subscribed website, read source-free instructions with `project:read`:

```http
GET /projects/{projectId}/improvement-prompt?report={reportId}&issue=0&issue=2
```

The response is `{artifact: {...}}`. The artifact identifies the website, original report and finding indexes, observation time, scope, and selected findings. It supplies evidence and acceptance criteria, not source code or a patch.

Save an improvement with `tasks:write`:

```http
POST /projects/{projectId}/tasks
Content-Type: application/json
Idempotency-Key: <new UUID for this task>

{"type":"improvement","details":{"reportId":"report_example","issueIndexes":[0,2]}}
```

Replace the sample report and indexes with observed values from that same project. `issueIndexes` accepts 1–50 nonnegative integer indexes; optional `title` is at most 150 characters. Only send the documented reference fields. Do not add source, source paths, diffs or user credentials.

HTTP 201 returns the task with `status: ready`, `details` and an `artifact`. Record its ID. Improvement tasks do not use `/execute` or `/accept`; those calls cannot apply your local changes. Legacy `type: fix` source-patch writes return 410.

## Publish locally and recheck

Make and validate the authorized changes in the user's own environment. Follow that project's release process. Once the relevant changes are publicly observable, call with both `tasks:write` and `checks:write`:

```http
POST /projects/{projectId}/tasks/{taskId}/recheck
Content-Type: application/json
Idempotency-Key: <new UUID for this recheck>

{}
```

HTTP 202 returns an audit receipt. The task becomes `awaiting_recheck`. Poll its project-scoped audit job and any continuation, then read the task from the project snapshot again:

| Task status | What it establishes |
| --- | --- |
| `ready` | Instructions are saved; no publication or resolution is established. |
| `awaiting_recheck` | A fresh audit is in progress. |
| `verified` | The fresh observation covered and passed the task's selected supported checks. |
| `needs_work` | At least one selected finding was observed again. |
| `unknown` | Missing coverage, an unavailable check, or a failed audit prevented confirmation. |

Report `details.verification` where available, including its report/run reference, check time, finding outcomes and failure reason. A job's `completed` status alone is insufficient. A verified technical check does not establish Google indexing, rankings or traffic.

## Errors and retries

Errors use `{error, message}`. Keep the machine `error` value unchanged while explaining the message in the user's language.

| HTTP status | Action |
| --- | --- |
| 400 | Correct the documented input; do not invent report IDs or indexes. |
| 401 | Ask the owner to restore project-key access locally. |
| 402 | Explain the subscription requirement; do not start checkout. |
| 403 | Check project identity and required key scopes. |
| 404 | The project/resource is unavailable to this key; do not try another user's IDs. |
| 409 | Read the actual error: ownership, an active job, a task state, allowance or idempotency may block the action. |
| 410 | The requested legacy operation is retired; use the improvement workflow. |
| 429 | Stop rapid retries, back off and honor Retry-After if supplied. |

Use a random UUID as `Idempotency-Key` for each logical POST. Retries use the same key and identical JSON bytes. A reused key with a different body returns 409. Preserve known IDs and check state after an interrupted call; do not use a different key to force another audit or task through. Bound polling and report a pending job instead of waiting indefinitely or claiming failure/success without evidence.
