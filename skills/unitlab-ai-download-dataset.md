---
name: Download an annotated dataset from Unitlab
description: >-
  Retrieve labeled results from Unitlab AI - either the annotation archive
  for a dataset release (optionally one split) or the raw files manifest.
api: openapi/unitlab-ai-sdk-openapi.yml
operations: [listDatasets, downloadDataset]
generated: '2026-07-21'
method: generated
---

# Download a Unitlab dataset release

Auth: `Authorization: Api-Key <key>` header on every request; base URL
`https://api.unitlab.ai`.

1. List releases with `listDatasets` (`GET /api/sdk/releases/`) and pick the
   dataset `pk` (UUID).
2. Call `downloadDataset` (`POST /api/sdk/releases/{dataset_id}/`) with a
   JSON body:
   - `{"download_type": "annotation"}` returns `{"file": "<url>"}` - stream
     that URL to disk. Add `"split_type": "train"` (or `validation`) to
     restrict to one split; omit it for all splits.
   - `{"download_type": "files"}` returns an array of file entries; write
     inline `content` entries directly and stream each `source` URL
     (the official SDK uses up to 50 parallel downloads).
3. Guard against path traversal when writing manifest `file_name` paths -
   resolve them under the target folder and reject escapes, as the official
   SDK does.

Errors: 401 = bad API key, 404 = unknown dataset UUID; envelope per
`errors/unitlab-ai-problem-types.yml`.
