# HACK & Six Pillars Assessment

Self-contained static site (`dist/index.html`) — 51-question assessment on the HACK framework and the Six Pillars of the AI-Powered Professional, submitting results to a Google Sheet via Apps Script.

## Deploy to GitHub Pages
1. Upload `index.html` (repo root) and this `README.md` to `main` — e.g. via the repo's "Upload files" page.
2. Repo → Settings → Pages → Source: Deploy from branch → `main` → `/ (root)`.
3. Visit the published URL once Pages finishes building (a minute or two after the first push).

## ⚠️ Backend must be fixed before going live
The submit button posts to the Google Apps Script URL baked into the page (`Component.SHEETS_URL` inside the source `.dc.html`). Testing it end-to-end today showed the **frontend completes correctly** (all questions answerable, results screen renders, payload builds) but **no row reaches the sheet** — the response sheet still has only its header row after a full test submission.

Cause: the deployment URL is in the `/a/macros/modelcitizn.com/s/.../exec` form, which Google issues for **domain-restricted** deployments — it only accepts requests from someone logged into a `modelcitizn.com` Google account. Public visitors won't be, so the POST hits a login wall instead of `doPost()`, and the hidden-iframe submission technique can't surface that failure back to the page (it just shows the results screen regardless).

**Fix, in the Apps Script project:**
1. Deploy → Manage deployments → edit the active deployment (or create a new one).
2. Execute as: **Me**.
3. Who has access: **Anyone** (not "Anyone within modelcitizn.com").
4. Redeploy — this issues a plain `/macros/s/.../exec` URL (no `/a/<domain>/` segment).
5. Update `Component.SHEETS_URL` in the source `.dc.html` to the new URL, then re-run the bundler/export to refresh `dist/index.html`.
6. Re-test a submission and confirm a new row appears in the sheet.

## Files
- `index.html` — the deployable static page (self-contained; fonts inlined, React/ReactDOM/Babel load from unpkg at runtime).
- Source of truth for edits is the original `.dc.html` design file, not this exported `index.html` directly — re-export after any source changes.
