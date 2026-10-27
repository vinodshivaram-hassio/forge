---
forge_format: 1
task_id: T-0001
parent_id: none
root_id: T-0001
type: intake
title: Make uploads reliable on flaky networks
version: 3
priority: 2
complexity: 3
modalities: [text]
tags: [backend]
---

## Original Input
Uploads sometimes fail on flaky networks. Make them more reliable.

## Goal
The upload client retries transient failures with exponential backoff and raises UploadFailed when retries run out.

## Facts
- F1 [confirmed, src: client.py] Uploads use a single HTTP PUT with no retry.
- F2 [confirmed, src: storage API docs, T-0003] Transient means HTTP 502, 503, 504 or a connection timeout.
- F3 [confirmed, src: client.py] No secrets, personal data or permissions are touched by the upload retry path.

## Constraints
- C1 Maximum 5 attempts; delay doubles from 1 second, capped at 30 seconds.

## Acceptance Criteria
- AC1 Given a server returning 503 twice then 200, when upload runs, then it succeeds on attempt 3.
- AC2 Given a server returning 400, when upload runs, then it fails at once with no retry.
- AC3 Given a server returning 503 on all 5 attempts, when upload runs, then it raises UploadFailed naming status 503 after attempt 5.

## Out of Scope
- Resumable uploads.
- Retrying non-upload HTTP calls.

## Artifacts
None

## Open Questions
- Q1 [answered, ask: researcher] Which HTTP statuses count as transient?

## Clarifications
- Q1 (answered) Which HTTP statuses count as transient? A: 502, 503, 504 and connection timeouts; see F2. Source: storage API docs, T-0003.

## Decisions
None

## Review Flags
None
