---
name: Upload data samples to a Unitlab annotation project
description: >-
  Feed an existing Unitlab AI annotation project with files (images, video,
  audio, text, medical DICOM/NIfTI, or PDF documents), respecting the
  project's accepted formats and size limits.
api: openapi/unitlab-ai-sdk-openapi.yml
operations: [listProjects, getProjectUploadInfo, uploadProjectData, finalizeMedicalUploadSession]
generated: '2026-07-21'
method: generated
---

# Upload data samples to a Unitlab project

Auth: every request carries `Authorization: Api-Key <key>` (create keys at
https://app.unitlab.ai, API Keys page). Base URL `https://api.unitlab.ai`.

1. Find the target project with `listProjects` (`GET /api/sdk/projects/`);
   use the project's `pk` (UUID) as `project_id`.
2. Call `getProjectUploadInfo` and honor the response: only send files whose
   extension is in `accepted_formats`, and skip files larger than the
   per-type `max_file_sizes` (bytes).
3. Generate one UUID `session_id` for the whole run. Upload each file with
   `uploadProjectData` as `multipart/form-data` (`file`, `session_id`, and
   `fps` for video files - default 1.0). Limit concurrency (the official SDK
   uses 20 parallel uploads) and retry transient failures.
4. If any medical files (dcm/nii/nii.gz/nrrd) were uploaded successfully,
   call `finalizeMedicalUploadSession` once with the same `session_id` so the
   platform groups them into series.

Errors: 401 = bad/missing API key; 403 = plan/subscription limit; the error
body is a plain `detail`/`message` JSON envelope (see
`errors/unitlab-ai-problem-types.yml`). No idempotency keys exist - do not
blindly re-upload on ambiguous failures.
