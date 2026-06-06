# ECMonitor Daily Ingestion Update - 2026-06-06

This folder records the platform-side update made to the ECMonitor daily literature ingestion workflow on 2026-06-06.

## Scope

- Diagnose why the daily 08:00 ingestion task was running but public counts were not increasing.
- Improve discovery, PDF retrieval fallback, and conservative abstract-derived ingestion.
- Run one controlled production sync and record the verified public result.

## Root Cause

The scheduler was active, but most daily runs produced `inserted_records=0`. The dominant failure modes were:

- `pending_verification`: DOI verified but no direct PDF URL was available.
- `download_failed`: publisher PDF endpoints returned 403 or timed out.
- `no_water_records`: the extractor found no water-environment concentration rows.
- weak candidate quality: framework, modelling, editorial, sediment-only, and retracted-like items were entering the processing queue.

## Implementation Summary

- `scripts/download_pdf_by_doi.py` now keeps the existing `auto-paper-harvester` resolver and adds a short-timeout, legal-only `scansci-pdf` fallback.
- `scripts/daily_new_pollutant_update.py` now prioritizes OpenAlex candidates with open access/full-text signals, strengthens relevance filtering, and deprioritizes non-monitoring candidates.
- Abstract-derived fallback rows were expanded conservatively for explicit evidence such as named pollutants with concentration ranges and aggregate PFAS/PPCP range statements.
- `.env.daily-update` enables the `scansci-pdf` fallback and points it to a persistent checkout at `/home/zhome/scansci-pdf`.

## Production Verification

Controlled run:

- run key: `20260606T044257Z_emerging`
- status: `completed`
- candidates found: `6`
- processed papers: `2`
- verified papers: `2`
- imported references: `1`
- inserted records: `3`

Production public summary after sync:

- pollutants: `1203`
- sites: `2357`
- references: `420`
- records: `23029`

The run imported:

- DOI: `10.1002/etc.5953`
- title: `Occurrence and Environmental Risk Assessment of Contaminants of Emerging Concern in Brazilian Surface Waters`
- inserted rows: `3`
- evidence type: abstract-derived fallback because publisher PDF access returned 403.

## Constraints

- The fallback is intentionally legal-only: no Sci-Hub, Tor, or institutional-login path is enabled.
- The workflow still cannot guarantee five successful imported papers per day. It can scan more candidates, but successful growth depends on accessible full text or explicit concentration evidence in abstracts.
- Abstract fallback rows are lower precision and should remain visibly marked as `abstract-derived`.

