# Release Notes

## 2026-06-06 Daily Ingestion Stabilization

The ECMonitor daily ingestion pipeline was updated after investigation showed that scheduled runs were executing but not increasing public counts.

### Changes

- Added a `scansci-pdf` fallback path for legal-only PDF retrieval after the existing downloader fails.
- Improved OpenAlex candidate selection by preferring open-access/full-text candidates.
- Strengthened academic relevance filters to exclude editorial, review, framework, modelling, sediment-only, treatment/removal, and retracted-like candidates.
- Added conservative abstract-derived fallback extraction for explicit concentration evidence.
- Verified D1 synchronization after a controlled import.

### Verified Production Result

The controlled run `20260606T044257Z_emerging` completed and synced to production.

Before the successful controlled run, the public summary was:

- pollutants: `1202`
- sites: `2356`
- references: `419`
- records: `23026`

After sync, the public summary was:

- pollutants: `1203`
- sites: `2357`
- references: `420`
- records: `23029`

### Next Operational Check

Check the next scheduled run after `2026-06-07 08:00 CST`:

```bash
curl -sS https://eco-website.pages.dev/api/public/summary
curl -sS https://eco-website.pages.dev/api/public/latest-additions
```

