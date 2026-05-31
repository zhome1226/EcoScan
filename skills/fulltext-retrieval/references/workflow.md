# Retrieval Workflow

## Step 1. Direct open PDF

Try DOI landing pages, repository copies, and obvious open-access download links first.

Success condition:

- full PDF is downloaded
- file passes integrity checks

## Step 2. Publisher API harvest

Route DOI-based retrieval through the project harvester layer.

Supported priority targets:

- Elsevier
- Springer
- Wiley
- OA fallbacks through Crossref / OpenAlex when available

The harvester should:

- classify publisher from DOI
- use credentials from environment
- download article PDF or XML when supported
- attempt supplementary file capture
- write structured manifest rows

## Step 3. Optional scansci-pdf backend

Use `scansci-pdf` only when it is installed and enabled for the project or task.

Preferred backend behavior:

- legal-only or open-access first
- publisher, PubMed Central, Europe PMC, repository, and OA metadata routes before restricted routes
- institution-authorized paths only when the user has access rights
- batch, resume, cache, and diagnostics when retrieving many targets

After a backend hit:

- normalize the source into the ECMonitor manifest
- copy or move the file into the ECMonitor evidence library
- run the normal integrity gate
- pass only validated PDFs to extraction

Do not treat backend success as ECMonitor validation success.

## Step 4. Zotero plus institutional access

If the earlier steps fail:

- search existing Zotero storage with title, DOI, author, year clues
- if a Zotero item can be opened through campus access, save the resulting full text into the project workspace
- treat Zotero as a real full-text acquisition channel, not just a hint source

## Step 5. Browser-assisted retrieval

Use browser automation only after the earlier steps fail.

Preferred discovery path:

1. Google Scholar
2. Google Search
3. publisher landing page

Manual assistance is allowed for:

- login
- campus auth
- Cloudflare
- captchas

## Integrity statuses

Use only these normalized statuses:

- `fulltext_ready`
- `pdf_one_page_only`
- `pdf_corrupted`
- `html_only`
- `abstract_only`
- `supplement_available`
- `wrong_article`
- `access_blocked`
- `not_found`

## Backend manifest additions

When `scansci-pdf` is used, add:

- `backend`: `scansci_pdf`
- `backend_source`: downloader-reported source/provider name
- `policy_mode`: `legal_only`, `institutional`, `user_approved_extended`, or `manual_review`
- backend diagnostic message when download or login fails
- cache hit/miss state when available
- explicit approval note for any extended source route
