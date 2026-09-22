# Family Insurance Dashboard

A pure-frontend dashboard for managing your family's insurance policies. Without signing in it runs fully offline and all data stays in your own browser; after signing in, the policy ledger syncs across your devices — while **ID numbers and contract attachments always stay on your machine and never leave for the cloud**.

[简体中文](README.md)

> The sample data bundled in this repository is **anonymized** and contains no real personal information.

---

## Preview (mockups · anonymized data)

> The images below are **mockups** built from the dashboard's built-in anonymized sample data (Sample Policyholder A / Sample Insured B, etc.). They contain **no real personal information**. When you open the dashboard, it shows your own data.

### 1. Overview & renewal reminders
The top shows total policies, this year's total premium, and policyholder / coverage-type statistics; the middle shows a coverage-type donut chart and per-insured premium breakdown; the renewal banner lists policies expiring soon or already expired.

![Overview & renewal reminders](assets/overview.svg)

### 2. Policy details & entry
The detail table supports drag-to-reorder columns, manual / multiple sort modes, double-click cell editing, and batch-updating records that share the same name; the "Add / Edit Policy" form records basic info (insured, policyholder, coverage type, product, insurer, premium, etc.).

![Policy details & entry](assets/detail.svg)

### 3. Data analysis
For each coverage type (e.g. million-medical insurance), view each insured person's yearly premiums with year-over-year change, plus a trend line chart; coverage types, years and insured persons can be added / removed.

![Data analysis](assets/analysis.svg)

---

## 1. How to run

The main interface is a single HTML file (`src/family-insurance-dashboard.html`) plus a few scripts in `src/repository/` used for cloud sync. No build step, no dependencies to install. Three ways to start:

1. **Open directly** (simplest; local features only)
   Double-click `src/family-insurance-dashboard.html` to open it in any modern browser. All data stays in your local browser.

2. **Clone from GitHub and open**
   ```bash
   git clone https://github.com/wanghoufan/family-insurance-dashboard.git
   # Enter the directory and double-click src/family-insurance-dashboard.html
   ```

3. **Local static server / self-hosting** (signing in for cloud sync requires http:// — the login callback cannot complete over file://)
   ```bash
   cd family-insurance-dashboard/src
   python -m http.server 8000
   # Visit http://localhost:8000/family-insurance-dashboard.html
   ```
   You can also self-host with the bundled `Dockerfile` + `compose.yaml` (static hosting on nginx:alpine, stateless container, host port 3200 by default). To enable cloud sync, copy `src/config.example.js` to `src/config.js` and fill in your own Supabase public config (that file is not committed to Git).

> Note: the repository only contains **anonymized sample data**. The first launch shows samples; to restore your own data see "Backup & restore".

---

## 2. Features

- **Policy detail table**
  - Drag to reorder columns (remembered automatically)
  - Drag to manually reorder rows (when "manual sort" is selected)
  - Multiple sort modes: by creation asc / desc, by name, by current-year premium
  - Double-click any cell to edit; add, edit, delete records
  - Contract column: upload / open / replace / delete
- **Premium trend analysis**: manually enter yearly premiums per coverage type; year-over-year change is computed automatically; coverage types can be deleted
- **Expiry reminders**: flags "expiring soon / expired" based on the policy period; optional desktop notifications
- **Charts**: coverage-type donut chart and premium composition bar charts by insured / policyholder / insurer
- **Theme switcher**: dark / light / follow system, cycled with one click and remembered
- **Multi-device sync (optional)**: after signing in, policies and premium rates sync to the cloud; sign in with the same account on another device to continue; offline changes queue up and upload when back online
- **Encrypted backup**: encrypted/decrypted locally with WebCrypto; the backup file contains all policy fields and contract attachments, and the password never leaves your machine

---

## 3. Data storage & privacy (important)

| Data | Stored in | Uploaded? |
|------|-----------|-----------|
| Policy ledger (insured, coverage type, premium, etc.) + phone | Local browser; synced to cloud after sign-in | Yes, when signed in |
| Derived fields (birth date / expiry date, computed) | Synced with the ledger | Yes, when signed in |
| Insured's ID number, policy number, payment card number, raw policy period text | Local `localStorage` | **No** |
| Contract attachments (PDF / images) | Local `IndexedDB` | **No** |
| Encrypted backup file | A `.json` file you keep yourself | No (decryption also happens locally) |

- Not signed in = pure local mode; nothing is uploaded.
- When signed in, only the first two rows above go to the cloud; ID numbers and contract attachments **never** leave your machine.
- Clearing browser data / switching browsers / switching computers loses local data — make an "Encrypted backup" or "Export JSON" first.

---

## 4. Backup & restore

### Encrypted backup (recommended; most complete)
- **Encrypt & back up**: click "加密备份" and set a password → downloads `家庭保单加密备份.json` (all policy fields, todos, premium rates, and **all contract attachments**).
- **Restore**: click "加密恢复", pick the file and enter the password → local data is overwritten and the page refreshes.
- The password exists only in your head; if you forget it, the backup cannot be decrypted.

### Policy data (JSON / Excel)
- **Backup (JSON)**: click "导出 JSON" → downloads `家庭保单备份.json` (does not include contract attachments or ID info).
- **Export Excel**: click "导出 Excel" → downloads `家庭保单明细表.xls` (current data in the 15-column desktop template, incl. status column; generated offline).
- **Restore / import**: click "导入" → supports JSON, `.xls` exported by this dashboard, legacy blank templates, or standard CSV, with automatic column-name normalization; importing **overwrites** current data, so back up first. Arbitrary xls/`.xlsx` files cannot be imported as backups.

### Batch-editing records that share the same name
- The filter bar has a "sync same-name records on edit" checkbox (checked by default).
- Editing any of **insured / policyholder / coverage type / product name / insurer** updates all records sharing that field's value at once.
- Uncheck it to apply the edit to the current record only.

### Contract attachments (limitations)
- Contracts are stored in the browser's `IndexedDB` and are **not** included in a plain "Export JSON".
- To migrate contracts across machines, use "Encrypted backup" for a complete backup/restore; otherwise contracts will not come back after importing JSON.

---

## 5. FAQ

**Q: The page is blank / shows sample data — where are my policies?**
A: The repository only has sample data. If you used the dashboard on this machine before and didn't clear the browser, your data is still in `localStorage` — just refresh. In a new environment, restore with "加密恢复" or "导入".

**Q: Can multiple people share one live dataset?**
A: Signing in with the same account syncs your own ledger across devices; different accounts are isolated, and real-time multi-user collaboration is not provided. Without signing in, each person maintains their own local copy and can pass data around via export/import.

**Q: How do I back up contract attachments?**
A: Use "Encrypted backup" for a complete backup (contracts included), or keep using the same browser.
