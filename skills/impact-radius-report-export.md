---
name: impact-radius-report-export
description: Run an impact.com performance report and collect the result through the async job pipeline instead of paging a live endpoint.
api: impact.com Brand API
version: v14
base_url: https://api.impact.com/Advertisers/{AccountSID}
generated: '2026-08-13'
method: generated
source: openapi/impact-radius-brand-reports-v14.yml, openapi/impact-radius-brand-jobs-v14.yml
operations:
  - listAllReports
  - getReportMetadata
  - runReport
  - exportReport
  - listJobs
  - getJobById
  - downloadJobResult
  - replayJob
  - exportClicks
---

# Export an impact.com performance report

Use for clicks / actions / revenue analysis, partner league tables, and any pull large enough
that paging a live list endpoint is the wrong tool.

## Steps

1. **Discover the report.** `listAllReports` returns the reports available to the account.
   Reports are templates with their own parameters — do not guess an id.
2. **Read its contract.** `getReportMetadata` tells you which filters and dimensions that report
   accepts. Skipping this step is how agents end up passing parameters the report ignores and
   then misreading the empty result as "no data".
3. **Choose synchronous or async.**
   - `runReport` returns results inline. Fine for small windows.
   - `exportReport` queues the work and returns a job. Use this for anything wide or long.
4. **Poll the job.** `getJobById` until it completes; `listJobs` shows the queue. Do not poll
   tightly — see the rate-limit note below.
5. **Collect.** `downloadJobResult` retrieves the finished payload. `replayJob` re-runs a job
   that failed rather than rebuilding the request.
6. **Clicks are separate.** Click-level data comes from `exportClicks`, not from a report.

## Rules that will bite you

- **ReportExport is capped at 100 requests per day and ClickExport at 10 per day.** These are the
  tightest limits on the platform and they are daily, not hourly — a polling loop that treats a
  job poll like a normal request will exhaust the export budget for the whole account. Poll on a
  schedule measured in minutes, and cache the job id.
- The hourly ceiling is 1,000 requests (3,600 on `/Catalogs`). Read `X-RateLimit-Remaining` and
  `X-RateLimit-Reset` on every response; `429` means stop, not retry immediately.
- The legacy Reports API was superseded by ReportExport with a stated migration date of
  2025-09-01. New integrations should target ReportExport.
- Report output is a file, not JSON pagination. Expect `text/csv`, `application/pdf` or
  `application/octet-stream` from the download step.

## References

- `rate-limits/impact-radius-rate-limits.yml`
- `lifecycle/impact-radius-lifecycle.yml`
- `conventions/impact-radius-conventions.yml`
