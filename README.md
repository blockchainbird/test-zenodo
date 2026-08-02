# Zenodo DOI — minimum-effort developer guide

**Audience:** you need a permanent DOI for a Spec-Up-T repo. You do not want to learn Zenodo. Do the least work that works.

**Official background (optional):**  
https://trustoverip.github.io/spec-up-t-website/docs/developer-documentation/create-zenodo-doi

---

## The only mental model you need

Zenodo gives you **two** identifiers. Ignore everything else.

| Name | Also called | Changes? | What to put in the spec |
|------|-------------|----------|-------------------------|
| **Main ID** | Concept DOI | **Never** | Prefer this (after first publish) |
| **Sub-ID** | Version DOI | New one every publish | Use only for the *first* release ZIP |

Example from this test repo:

- Main ID: `10.5281/zenodo.21759353` → https://doi.org/10.5281/zenodo.21759353  
- First Sub-ID: `10.5281/zenodo.21759354`  
- Later Sub-ID (from GitHub Action): `10.5281/zenodo.21759414`

**Link format to paste in markdown** (near the top of `spec/spec-head.md`):

```markdown
**DOI**

<https://doi.org/10.5281/zenodo.XXXXXXXX>
```

---

## What is automated vs not

| Task | Who does it |
|------|-------------|
| First DOI (reserve → embed → first GitHub Release → upload ZIP → Publish on Zenodo) | **You, once** (manual) |
| Later updates (new GitHub Release → Zenodo “New version”) | **GitHub Action** (automatic) |

Workflow file: `.github/workflows/zenodo-publish.yml`  
Trigger: publishing a GitHub **Release** (or manual `workflow_dispatch`).

**Do not** run the Action for the first publish. It only does *updates*.

---

## Part A — One-time setup (about 15–20 minutes)

Do this once per specification repository. Work on the **`main`** branch of the **original** repo.

### A1. Create a Zenodo account (if needed)

1. Open https://zenodo.org/signup/  
2. Prefer an institutional email if you have one.  
3. Log in and confirm you can open https://zenodo.org/deposit  

If this record will live under an **organization** Zenodo account and you are not that account: someone with manage rights must add you with **Can manage** on the record (or community) before you can publish updates. Wait overnight after signup if search cannot find your username yet.

### A2. Create an API token (needed for the GitHub Action later)

1. Open https://zenodo.org/account/settings/applications/tokens/new/  
2. Name it something obvious, e.g. `github-actions-test-zenodo`.  
3. Enable **both**:
   - `deposit:write`
   - `deposit:actions`
4. Create the token and **copy it once**. Store it in a password manager.  
5. You will paste it into GitHub Secrets in step **A8** (`ZENODO_TOKEN`). Do **not** commit it to git.

### A3. Reserve a DOI (keep the browser tab open)

1. Open https://zenodo.org/uploads/new  
2. When asked if you already have a DOI, choose **No, I need one**.  
3. Click the button to **Reserve a DOI**.  
4. Copy the reserved DOI (looks like `10.5281/zenodo.21759354`).  
5. Build the URL: `https://doi.org/` + that DOI.  
6. **Leave this draft upload page open.** You return here in A6. Do not publish yet.

That reserved value is the first **Sub-ID** (Version DOI).

### A4. Put the reserved DOI in the specification on `main`

1. Edit `spec/spec-head.md` (or equivalent head file).  
2. Near the top, add:

   ```markdown
   **DOI**

   <https://doi.org/10.5281/zenodo.YOUR_RESERVED_SUB_ID>
   ```

3. Commit and push to **`main`**.

Example commit message: `Embed reserved Zenodo DOI`.

### A5. Create the first GitHub Release from `main`

1. Open your repo → **Releases** → **Create a new release**  
   (or: `https://github.com/OWNER/REPO/releases/new`).  
2. Choose **Choose a tag** → type e.g. `v0.1.0` → create tag on **`main`**.  
3. Title e.g. `v0.1.0`.  
4. Publish the release.  
5. Download the source ZIP GitHub builds for that tag  
   (**Source code (zip)** on the release page).

### A6. Upload the ZIP to the open Zenodo draft and Publish

Back on the Zenodo draft from A3:

1. Upload the GitHub Release ZIP.  
2. Fill the **minimum** required fields (Zenodo will block Publish until these are OK):
   - **Title** — specification title  
   - **Creators / Authors** — at least one name  
   - **Description** — one short paragraph is enough  
   - **Upload type** — e.g. Publication → Standard (or whatever matches your deliverable)  
   - **Access** — Open  
   - **License** — pick one (e.g. CC-BY-4.0 or your project license)  
   - **Version** — e.g. `v0.1.0`  
3. Optional but useful: add related identifier → your GitHub repo URL.  
4. Click **Publish**. Confirm.

You now have:

- **Sub-ID** = the reserved DOI (this version)  
- **Main ID** = Concept DOI shown on the record page (never changes)

Copy both. Example record page: https://doi.org/10.5281/zenodo.21759353  

On the record, note:

- Concept DOI / Main ID  
- Latest version’s numeric record id (the number after `zenodo.` in the *version* DOI, e.g. `21759354`)

### A7. Switch the spec to the Main ID (stable forever)

1. Edit `spec/spec-head.md` again.  
2. Replace the reserved Sub-ID link with the **Main ID**:

   ```markdown
   **DOI**

   <https://doi.org/10.5281/zenodo.YOUR_MAIN_ID>
   ```

3. Commit and push to **`main`**.

You usually never change this line again.

### A8. Wire GitHub so updates are automatic (paste the token here)

This is where the token from **A2** goes.

1. Open the GitHub repo → **Settings** → **Secrets and variables** → **Actions**.  
   Direct URL pattern: `https://github.com/OWNER/REPO/settings/secrets/actions`  
2. Under **Repository secrets**, click **New repository secret**.

#### Secret (required)

| Name | Value |
|------|--------|
| `ZENODO_TOKEN` | Paste the token from **A2** (full string, no spaces) |

#### Variable (recommended — set this and forget deposition IDs)

| Name | Value |
|------|--------|
| `ZENODO_CONCEPT_DOI` | Main ID, e.g. `10.5281/zenodo.21759353` |

#### Secret (optional if Concept DOI variable is set)

| Name | Value |
|------|--------|
| `ZENODO_DEPOSITION_ID` | Numeric id of the *latest published* version only |

Prefer **`ZENODO_CONCEPT_DOI`**. Then you do not need to bump `ZENODO_DEPOSITION_ID` after every release.

Confirm `.github/workflows/zenodo-publish.yml` exists on `main` (it does in this test repo).

---

## Part B — Forever after (about 2 minutes per update)

You do **not** open Zenodo for normal updates.

1. Merge whatever you want onto **`main`**.  
2. Create a **new** GitHub Release / tag from `main` (e.g. `v0.2.0`, `v1.1.0`).  
   - Do **not** reuse an old tag.  
3. Wait for the Action **Publish release to Zenodo (update)** to finish green.  
4. Done. Main ID is unchanged. Zenodo has a new Sub-ID for this snapshot.

Check:

- Actions tab → latest run of `zenodo-publish.yml` → success  
- https://doi.org/YOUR_MAIN_ID → shows the new version  

CLI equivalent:

```bash
git checkout main
git pull
# ... commits already on main ...
gh release create v0.2.0 --title "v0.2.0" --notes "…" --target main
gh run list --workflow=zenodo-publish.yml --limit 1
```

---

## Exact “do / don’t” checklist

### Do

- Work on **`main`** of the original repo.  
- Reserve DOI **before** first publish if you need the DOI inside the first ZIP.  
- After first publish, keep only the **Main ID** in the markdown.  
- Use **New version** for updates (the Action does this).  
- Create a **new tag** for every publish.

### Don’t

- Don’t create a second unrelated upload at https://zenodo.org/uploads/new for the same spec.  
- Don’t expect the Action to do the **first** publish.  
- Don’t commit `ZENODO_TOKEN` into the repo.  
- Don’t overwrite files on an already-published Zenodo version as your normal update path.  
- Don’t put a new Sub-ID into the spec on every release if the Main ID is already there.

---

## If something breaks (minimum debugging)

### Action fails: missing secrets

Error mentions `ZENODO_TOKEN` or Concept/Deposition ID.

→ Re-do **A8**. Token scopes must include `deposit:write` and `deposit:actions`.

### Wrong files / DOI missing in ZIP

The release ZIP is whatever commit the tag points at.

```bash
git checkout main
git pull
# fix source, commit, push
git tag v0.1.1
git push origin v0.1.1
gh release create v0.1.1 --title "v0.1.1" --target main
```

If the bad tag was never published to Zenodo, you may delete/recreate that tag. If Zenodo already has that version, use a **new** tag and let the Action publish another New version.

### Accidental second Zenodo record

You started `/uploads/new` again instead of New version / Action.

→ Stop. Use the **original** record’s Main ID. Prefer fixing process over “merging” records (Zenodo does not merge concepts casually). Ask Zenodo support only if you truly published a duplicate by mistake.

### Token leaked (chat, screenshot, commit)

1. Revoke it in Zenodo applications settings.  
2. Create a new token (A2).  
3. Update GitHub secret `ZENODO_TOKEN`.

---

## Copy-paste metadata starter (first publish)

Use this on the Zenodo draft if you want zero thinking:

- **Title:** same as `specs.json` → `title`  
- **Creators:** `Lastname, Firstname` + affiliation  
- **Description:** 2–4 sentences from the spec intro  
- **Version:** same as the Git tag (`v0.1.0`)  
- **Related identifier:** `https://github.com/OWNER/REPO` (relation: isSupplementTo / software)

---

## This test repository (worked example)

| Item | Value |
|------|--------|
| GitHub | https://github.com/blockchainbird/test-zenodo |
| Main ID | https://doi.org/10.5281/zenodo.21759353 |
| First Sub-ID (`v0.1.0`, manual) | https://doi.org/10.5281/zenodo.21759354 |
| Second Sub-ID (`v0.2.0`, Action) | https://doi.org/10.5281/zenodo.21759414 |
| Workflow | `.github/workflows/zenodo-publish.yml` |
| Spec DOI line | `spec/spec-head.md` |

### Local Spec-Up-T commands (unrelated to Zenodo)

```bash
cd test-zenodo
npm install
npm run render   # or: npm run menu
```

---

## One-page summary

1. **Once:** reserve DOI → paste Sub-ID in spec → release `v0.1.0` → upload ZIP on Zenodo → Publish → paste Main ID in spec → set `ZENODO_TOKEN` + `ZENODO_CONCEPT_DOI`.  
2. **Later:** only create a new GitHub Release. The Action publishes the Zenodo update.  
3. **Cite forever:** the Main ID (`https://doi.org/10.5281/zenodo.…`).
