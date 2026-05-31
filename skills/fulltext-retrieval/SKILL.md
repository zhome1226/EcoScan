---
name: fulltext-retrieval
description: Retrieve literature full text for environmental database rebuilding. Use when a task requires finding, downloading, validating, and recording article PDFs, HTML full text, or supplements with a fixed priority order: open PDF, publisher API harvest, optional scansci-pdf backend, Zotero plus institutional access, then browser-assisted retrieval.
---

# Fulltext Retrieval

Use this skill whenever the job is to obtain a validated source document before extraction.

`scansci-pdf` may be used as an optional acquisition backend when it is installed and explicitly enabled. Treat it as a downloader and discovery accelerator, not as the authority for source policy or evidence quality. ECMonitor still performs the final source gate, provenance recording, and PDF integrity checks before handoff.

## Fixed retrieval order

Always try these in order and stop when one yields usable full text:

1. open or open-access PDF
2. publisher API harvest
3. optional `scansci-pdf` backend in legal/OA/institution-first mode
4. Zotero plus campus or institutional access
5. browser-assisted retrieval

Do not skip ahead unless an earlier step is impossible for that item.

## Output expectations

For each target, record:

- `source_type`
- `source_url`
- `download_path` or `snapshot_path`
- `backend`: `ecmonitor_native`, `scansci_pdf`, or `manual`
- `backend_source` when a downloader reports a source/provider
- `policy_mode`: `legal_only`, `institutional`, `user_approved_extended`, or `manual_review`
- `pdf_page_count` if applicable
- `integrity_status`
- `failure_reason`
- `supplement_paths`
- `manual_intervention_required`

## Integrity gate

Before handing anything to extraction, verify:

- article identity matches the target
- file is not corrupted
- PDF is not one-page junk or title-only
- text extraction is possible
- supplement links are captured when available

## scansci-pdf guardrails

- Prefer legal/open/institution-authorized routes.
- Do not let `scansci-pdf` bypass ECMonitor source rules.
- Sci-Hub, LibGen, Tor, or similar extended routes require explicit user or project approval and must be marked `user_approved_extended`.
- Quarantine backend downloads with unclear source URLs, article identity, or access routes until manual review.

## When to load references

- Read `references/workflow.md` for the exact retrieval decision tree.
- Read `references/scansci-pdf-backend.md` when `scansci-pdf` is configured or requested.
- Use `scripts/publisher_api_harvest.py` for DOI-first harvesting and publisher routing.
- Use `scripts/check_pdf_integrity.py` for PDF checks.
