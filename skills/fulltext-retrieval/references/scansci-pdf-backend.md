# scansci-pdf Backend Policy

`scansci-pdf` is an optional acquisition backend for the ECMonitor full-text retrieval skill. Use it to improve DOI/title resolution, batch download, publisher/OA probing, institutional access workflows, caching, and network diagnostics. Do not use it as a replacement for ECMonitor's source gate or provenance manifest.

## When To Use It

- The target has a DOI, title, PMID, PMCID, or URL and native open-source probing did not immediately resolve a validated PDF.
- The task needs batch download, resume behavior, caching, or source health diagnostics.
- The user or project configuration has enabled `scansci-pdf`.
- The run can still write a complete ECMonitor manifest entry and pass the PDF integrity gate.

## Default Policy

- Default to legal/open behavior: publisher open access, PubMed Central, Europe PMC, OpenAlex/Unpaywall-style OA locations, repositories, and institution-authorized access.
- WebVPN, CARSI, EZProxy, saved cookies, or local library paths are allowed only when the user has rights to that access path.
- Sci-Hub, LibGen, Tor, or similar extended sources are not default routes. Use them only with explicit project approval, mark `policy_mode` as `user_approved_extended`, and keep the exact backend source in the manifest.
- If the backend returns a PDF but the source URL, article identity, or access route is unclear, quarantine the result until manual review.

## Invocation Flow

1. Normalize the target identifiers with the standard ECMonitor workflow.
2. Run native open-source probes first when they are cheap and deterministic.
3. Call `scansci-pdf` for discovery or download only if enabled.
4. Copy or move the returned PDF into the ECMonitor evidence library naming scheme.
5. Run `check_pdf_integrity.py` or the equivalent integrity gate.
6. Write one manifest entry per target, including backend provenance.
7. Pass only validated PDFs to downstream extraction.

## Manifest Mapping

| ECMonitor field | Source from `scansci-pdf` result |
| --- | --- |
| `retrieval_status` | success/failure state after ECMonitor integrity gate |
| `source_type` | normalized backend source category |
| `source_url` | resolved landing page, PDF URL, repository URL, or access route |
| `download_path` | ECMonitor-controlled local PDF path |
| `backend` | `scansci_pdf` |
| `backend_source` | backend-reported source/provider name |
| `policy_mode` | `legal_only`, `institutional`, `user_approved_extended`, or `manual_review` |
| `failure_reason` | backend error plus ECMonitor validation failure, if any |

## Source Normalization

- Publisher or open-access PDF: `publisher_open`
- PubMed Central or Europe PMC full text: `pubmed_central`
- Institutional repository or preprint server: `repository_open`
- WebVPN, CARSI, EZProxy, local library, or Zotero-authorized attachment: `institutional` or `zotero`
- Manual browser capture: `browser_manual`
- Unknown, ambiguous, or unsupported source: `unknown` and `needs_manual_access`

## Failure Handling

- `scansci-pdf` network or login failure: keep native ECMonitor fallback attempts and record the backend diagnostic message.
- Duplicate download: keep the validated existing PDF unless the new file has better provenance, higher page count, or better text extraction.
- PDF fails identity or integrity checks: mark `invalid_pdf`, retain checksum/path if useful for audit, and do not pass to extraction.
- Backend reports only citation metadata or a landing page: mark `not_found` or `needs_manual_access` with the actionable URL.
