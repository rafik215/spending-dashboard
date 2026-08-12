# Personal Spending Dashboard

A password-encrypted, static HTML dashboard built from `Personal_Spending_v5.xlsx`.

- **`dashboard.html`** — the dashboard itself. Single file, self-contained. Loads Chart.js from a CDN.
- **`build_dashboard.py`** — the build script. Run it after each monthly workbook update to refresh the encrypted data inside `dashboard.html`.

## How the encryption works

- Data is serialized to JSON, then encrypted with **AES-256-GCM** using a key derived from your password via **PBKDF2-SHA256** (200,000 iterations, random 16-byte salt).
- The encrypted blob is base64-encoded and embedded directly in `dashboard.html`.
- The password is never stored anywhere. Losing it means the data in that HTML file cannot be recovered — but you can always rebuild from the workbook.
- The dashboard uses the browser's native **Web Crypto API** for decryption; no external JS crypto library.

## Updating monthly

After you finish your monthly Excel update:

```bash
cd "Personal spending tracker"
python3 build_dashboard.py                          # will prompt for the password
# or non-interactive:
python3 build_dashboard.py 'YOUR_PASSWORD_HERE'

git add dashboard.html
git commit -m "Update dashboard: <month> data"
git push
```

The build script requires the `cryptography` Python package. Install once with:

```bash
pip3 install cryptography
```

## GitHub Pages setup (one-time)

1. Create a new **public** repo on GitHub (e.g. `spending-dashboard`).
2. In this folder, initialize git and push:
   ```bash
   git init
   git add dashboard.html
   git commit -m "Initial dashboard"
   git branch -M main
   git remote add origin git@github.com:<your-username>/spending-dashboard.git
   git push -u origin main
   ```
   **Important:** commit *only* `dashboard.html`. Do NOT commit `Personal_Spending_v5.xlsx`, the CSV statements, `category_mapping.json`, or `build_dashboard.py` (well, script is fine but not required for hosting). A safe `.gitignore` is included below.
3. In the GitHub repo settings → **Pages**, set the source to **Branch: main** / **Folder: /(root)**.
4. Wait ~1 minute. Your dashboard will be live at `https://<your-username>.github.io/spending-dashboard/dashboard.html`.

## Suggested `.gitignore` for the repo

```
Personal_Spending_v5*.xlsx
*.backup*.xlsx
*.backup*.json
category_mapping.json
Statements/
__pycache__/
*.pyc
.DS_Store
```

## Security notes

- **Public repo + weak password = data leak.** Anyone can download the encrypted `dashboard.html` and try passwords offline. PBKDF2 with 200k iterations slows this down a lot, but a short/common password can still be broken. Your current password (20 chars, mixed case, digits, and separators) is strong enough to shrug off any realistic attack.
- **Password rotation:** just rerun the build script with a new password. The next `dashboard.html` will only open with the new password. Old commits will still contain the old ciphertext, so treat any previously-compromised password as permanently compromised.
- **CDN dependency:** Chart.js is loaded from `cdn.jsdelivr.net`. If that CDN is blocked in your environment, download `chart.umd.min.js` locally and change the `<script src="…">` line to point at the local file.

## What's in the dashboard

- KPI header (YTD income, expenses, net, avg monthly burn, current month vs. prior)
- Category breakdown pie (YTD or per month)
- Top vendors (YTD or per month)
- Monthly trend line — toggle between total spending and per-category
- Budget vs. actual bar chart (only useful once you fill in Monthly Budget cells in the workbook)
- Anomaly flags — categories where the current month is >50% above the trailing 3-month average
- Searchable/filterable transactions table
