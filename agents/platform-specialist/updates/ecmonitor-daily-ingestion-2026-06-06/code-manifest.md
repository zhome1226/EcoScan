# Code Manifest

This manifest points to the implementation files used in the ECMonitor website repository.

## Website Repository

- repository: `https://github.com/zhome1226/Eco_Website.git`
- local path during update: `/home/zhome/eco_website`
- production site: `https://eco-website.pages.dev/`

## Modified Workflow Files

- `/home/zhome/eco_website/scripts/daily_new_pollutant_update.py`
- `/home/zhome/eco_website/scripts/download_pdf_by_doi.py`
- `/home/zhome/eco_website/.env.daily-update`
- `/home/zhome/eco_website/.env.daily-update.example`

## External Helper

- repository: `https://github.com/Rimagination/scansci-pdf.git`
- local path during update: `/home/zhome/scansci-pdf`
- checked revision during update: `b4e2c5630bbaf900e43ce49868c9951ce2c1474c`
- usage mode: Python package import via `PYTHONPATH`
- strategy: `legal_only`
- disabled paths: Sci-Hub, Tor, WebVPN, institutional login

## Runtime Configuration

Important `.env.daily-update` values after the update:

```env
POLLUTION_MONITOR_DB=/home/zhome/eco_website/data/pollution_monitor.db
ECOSCAN_OPENALEX_LOOKBACK_DAYS=1460
ECOSCAN_OPENALEX_PAGES_PER_QUERY=5
ECOSCAN_CANDIDATE_TIMEOUT_SECONDS=240
ECOSCAN_ENABLE_SCANSCI_PDF=1
ECOSCAN_SCANSCI_PDF_ROOT=/home/zhome/scansci-pdf
ECOSCAN_SCANSCI_TIMEOUT_SECONDS=75
ECOSCAN_PDF_DOWNLOAD_COMMAND=/usr/bin/env python3 /home/zhome/eco_website/scripts/download_pdf_by_doi.py --doi '{doi}' --landing '{landing_url}' --out-dir '{out_dir}/pdfs'
```

## Scheduled Task

- timer: `/home/zhome/.config/systemd/user/ecoscan-daily-update.timer`
- service: `/home/zhome/.config/systemd/user/ecoscan-daily-update.service`
- schedule: daily at `08:00:00` CST
- next verified trigger after update: `2026-06-07 08:00:00 CST`

Service command:

```bash
/usr/bin/env python3 /home/zhome/eco_website/scripts/daily_new_pollutant_update.py \
  --query "emerging contaminants surface water monitoring concentration" \
  --max-papers 5 \
  --max-candidates 200 \
  --search-pages 3 \
  --runs-dir /home/zhome/eco_website/runs \
  --cloudflare-dir /home/zhome/eco_website/cloudflare \
  --sync-d1
```

## Verification Commands

```bash
cd /home/zhome/eco_website
python3 -m py_compile scripts/daily_new_pollutant_update.py scripts/download_pdf_by_doi.py
curl -sS https://eco-website.pages.dev/api/public/summary
curl -sS https://eco-website.pages.dev/api/public/latest-additions
systemctl --user status ecoscan-daily-update.timer --no-pager
```

