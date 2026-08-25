# Zenodo DOI — minimum-effort developer guide

**Audience:** you need a permanent DOI for a Spec-Up-T repo. You do not want to learn Zenodo. Do the least work that works.

**Official background (optional):**  
[https://trustoverip.github.io/spec-up-t-website/docs/developer-documentation/create-zenodo-doi](https://trustoverip.github.io/spec-up-t-website/docs/developer-documentation/create-zenodo-doi)

---

## The only mental model you need

Zenodo gives you **two** identifiers. Ignore everything else.


| Name        | Zenodo may call it | Changes?              | What to put in the spec              |
| ----------- | ------------------ | --------------------- | ------------------------------------ |
| **Main ID** | Concept DOI        | **Never**             | Prefer this (after first publish)    |
| **Sub-ID**  | Version DOI        | New one every publish | Use only for the *first* release ZIP |


Ignore Zenodo’s word “concept” day to day; use **Main ID** / **Sub-ID**.

Example from this test repo:

- Main ID: `10.5281/zenodo.21759353` → [https://doi.org/10.5281/zenodo.21759353](https://doi.org/10.5281/zenodo.21759353)  
- First Sub-ID: `10.5281/zenodo.21759354`  
- Later Sub-ID (from GitHub Action, v0.11.0): `10.5281/zenodo.22093110`

**Link format to paste in markdown** (near the top of `spec/spec-head.md`):

```markdown
**DOI**

<https://doi.org/10.5281/zenodo.XXXXXXXX>
```

---



## What is automated vs not


| Task                                                                                | Who does it                   |
| ----------------------------------------------------------------------------------- | ----------------------------- |
| First DOI (reserve → embed → first GitHub Release → upload ZIP → Publish on Zenodo) | **You, once** (manual)        |
| Later updates (new GitHub Release → Zenodo “New version”)                           | **GitHub Action** (automatic) |


Workflow file: `.github/workflows/zenodo-publish.yml`  
Trigger: publishing a GitHub **Release** (or manual `workflow_dispatch`).

**Do not** run the Action for the first publish. It only does *updates*.

### Copy the workflow to another Spec-Up-T repo

Copy `.github/workflows/zenodo-publish.yml` **unchanged**. It has no hardcoded repo name or DOI.

Then in **that** repo, under **Settings → Secrets and variables → Actions**:

1. **Secret** `ZENODO_USER_API_TOKEN` — Zenodo user token (same token OK across repos if that user can manage those records).
2. **Variable** (Variables tab, not Secrets) `ZENODO_SPEC_MAIN_DOI` — **that spec’s** Main ID only.

Do not copy this test repo’s Main ID (`10.5281/zenodo.21759353`) into ACDC/CESR/KERI.

The first Zenodo publish for that spec must already exist. After that, a GitHub Release is enough.

---



## Who owns what: tokens, secrets, and permissions

You asked whether things are bound to a **Zenodo user** or a **GitHub repo**. Short answer: **both systems are involved**, but different pieces bind to different places.

### Binding cheat sheet


| Name                               | What it really is                                                                                                   | Bound to Zenodo user?                                    | Bound to GitHub repo?                                                                      | Same value in every repo?                                        |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------- |
| `ZENODO_USER_API_TOKEN`            | Zenodo personal access token ([create at zenodo.org](https://zenodo.org/account/settings/applications/tokens/new/)) | **Yes** — belongs to whichever Zenodo account created it | **Stored** per repo (GitHub Actions secret), but the token itself is not “owned” by GitHub | **Can be** — one org/service token may be pasted into many repos |
| `ZENODO_SPEC_MAIN_DOI`             | Main ID of one specification                                                                                        | **No** — identifies **this spec** on Zenodo              | **Yes** — each spec repo gets its **own** variable                                         | **No** — ACDC, CESR, KERI each have a different Main ID          |
| `ZENODO_SPEC_LATEST_DEPOSITION_ID` | Numeric id of one published Zenodo version                                                                          | **No** — same spec as above                              | **Yes** — per repo (optional if Main DOI is set)                                           | **No**                                                           |


There is also `GITHUB_TOKEN`: GitHub creates this automatically for each workflow run. You do not configure it. It is **per repo, per run** and is only used to download the release ZIP from GitHub — not to talk to Zenodo.

**None of these names are Zenodo or GitHub reserved words.** They are labels chosen for this workflow; only the workflow file must use the same names.

### What actually runs when someone publishes a GitHub Release

```text
GitHub user (e.g. @SmithSamuelM) clicks “Publish release”
        ↓
GitHub Action starts in THAT repo
        ↓
Uses repo secret ZENODO_USER_API_TOKEN  ← Zenodo account that created the token
Uses repo var/secret for THAT spec’s Main DOI / optional deposition id
        ↓
Zenodo API: new version → upload ZIP → publish
```

Important consequences:

1. **The person who creates the GitHub Release does not need a Zenodo account** for the Action to work.
  They need **permission to publish releases** on that GitHub repo (org role / write access).
2. **The Action always uses the repo’s** `ZENODO_USER_API_TOKEN`, not the releasing user’s GitHub or Zenodo identity.
  If the token’s Zenodo user cannot manage that spec’s Zenodo record, the Action fails — even if the GitHub release succeeded.
3. `ZENODO_USER_API_TOKEN` **is one Zenodo user’s credential.**
  Reusing the **same token string** in three repos is fine **only if** that Zenodo user has **Can manage** on all three Zenodo records (or owns them).
4. `ZENODO_SPEC_MAIN_DOI` **is one spec’s permanent id.**
  Each specification repo must store **its own** Main ID. Do not copy ACDC’s DOI into CESR’s repo.
5. **Rotating the Zenodo token** means: create a new token on Zenodo (same or different user), update the GitHub secret in **every repo** that used the old token, revoke the old token.
6. **Revoking a Zenodo user’s access** on a record does not remove the GitHub secret — the next release will fail until someone fixes the token or Zenodo permissions.



### Special case: several spec repos, several GitHub releasers (ToIP KSWG)

**Example repos** (each is its own GitHub repo and its own Zenodo record):


| Specification | GitHub repo                                                                                   |
| ------------- | --------------------------------------------------------------------------------------------- |
| ACDC          | [trustoverip/kswg-acdc-specification](https://github.com/trustoverip/kswg-acdc-specification) |
| CESR          | [trustoverip/kswg-cesr-specification](https://github.com/trustoverip/kswg-cesr-specification) |
| KERI          | [trustoverip/kswg-keri-specification](https://github.com/trustoverip/kswg-keri-specification) |


**Example GitHub users** who should be able to ship a release without touching Zenodo manually:

- [Samuel Smith (@SmithSamuelM)](https://github.com/SmithSamuelM)  
- [Philip Feairheller (@pfeairheller)](https://github.com/pfeairheller)



#### What each person needs


| Need                                            | @SmithSamuelM / @pfeairheller                                               | Who sets it up (once)                                                     |
| ----------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Publish GitHub Releases on the trustoverip repo | GitHub **Write** or **Maintain** (or Admin) on that repo                    | trustoverip org admins                                                    |
| Zenodo workflow runs green                      | **Nothing personal on Zenodo** if repo secrets are already correct          | Whoever did first-time DOI + A8                                           |
| Zenodo “New version” API succeeds               | N/A at release time — Action uses `ZENODO_USER_API_TOKEN` from repo secrets | Token must be a Zenodo user with **Can manage** on **that** spec’s record |


So: **GitHub permission = who may cut a release.** `ZENODO_USER_API_TOKEN` **+** `ZENODO_SPEC_MAIN_DOI` **in repo settings = whether the Action can publish.**

#### Recommended layout for these three repos

Use **one shared Zenodo service approach** (simplest for releasers):

1. **One Zenodo account** used for automation — e.g. a ToIP/org Zenodo user, or a dedicated “github-actions” Zenodo user (not Samuel’s or Philip’s personal accounts unless you want that).
2. **One API token** from that account (`ZENODO_USER_API_TOKEN`).
3. On Zenodo, that account **owns** each record **or** has been granted **Can manage** on ACDC, CESR, and KERI records ([grant access](https://trustoverip.github.io/spec-up-t-website/docs/developer-documentation/create-zenodo-doi#grant-yourself-manage-access) if records live under another Zenodo login).
4. In **each** GitHub repo, under **Settings → Secrets and variables → Actions**:

  | Repo                      | Secret `ZENODO_USER_API_TOKEN` | Variable `ZENODO_SPEC_MAIN_DOI`                    |
  | ------------------------- | ------------------------------ | -------------------------------------------------- |
  | `kswg-acdc-specification` | same shared token              | ACDC Main ID only (e.g. `10.5281/zenodo.18792085`) |
  | `kswg-cesr-specification` | same shared token              | CESR Main ID only                                  |
  | `kswg-keri-specification` | same shared token              | KERI Main ID only                                  |

5. Add `.github/workflows/zenodo-publish.yml` to each repo (copy unchanged from this test repo, or via boilerplate).

After that, **Samuel or Philip** (or any other trusted releaser):

1. Merge to `main`.
2. **Releases → New release** → new tag → Publish.
3. Action runs with the **repo’s** user API token and **repo’s** Main DOI — no Zenodo login required.



#### What **not** to do

- Do **not** expect each releaser to paste their **personal** Zenodo token into GitHub — brittle, and most personal tokens will lack manage rights on org-owned records.  
- Do **not** use one Main ID for all three specs — each repo must point at its **own** Zenodo Main ID.  
- Do **not** assume “I released on GitHub” implies Zenodo updated — check the Actions tab if the DOI version did not move.



#### If the Action fails for a releaser


| Symptom                             | Likely cause                                                                     |
| ----------------------------------- | -------------------------------------------------------------------------------- |
| Release created, Action red         | Missing/wrong `ZENODO_USER_API_TOKEN` or `ZENODO_SPEC_MAIN_DOI` in **that** repo (Main DOI must be a **variable**) |
| 403 / permission errors from Zenodo | Token’s Zenodo user lost **Can manage** on that record                           |
| Wrong spec updated on Zenodo        | `ZENODO_SPEC_MAIN_DOI` in repo settings points at another spec                   |
| “Empty files” / upload 400          | Stale workflow without the Python upload; or leftover draft — discard and re-run |


Fix permissions or secrets once; releasers keep using GitHub Releases only.

---



## Part A — One-time setup (about 15–20 minutes)

Do this once per specification repository. Work on the `main` branch of the **original** repo.

### A1. Create a Zenodo account (if needed)

1. Open [https://zenodo.org/signup/](https://zenodo.org/signup/)
2. Prefer an institutional email if you have one.
3. Log in and confirm you can open [https://zenodo.org/deposit](https://zenodo.org/deposit)

If this record will live under an **organization** Zenodo account and you are not that account: someone with manage rights must add you with **Can manage** on the record (or community) before you can publish updates. Wait overnight after signup if search cannot find your username yet.

### A2. Create an API token (needed for the GitHub Action later)

This token is **bound to one Zenodo user account** (whoever is logged in when you create it). It is **not** bound to a GitHub repo until you paste it into GitHub Secrets in **A8**. See [Who owns what](#who-owns-what-tokens-secrets-and-permissions) above.

1. Open [https://zenodo.org/account/settings/applications/tokens/new/](https://zenodo.org/account/settings/applications/tokens/new/)
2. Name it something obvious, e.g. `github-actions-trustoverip-specs`.
3. Enable **both**:
  - `deposit:write`
  - `deposit:actions`
4. Create the token and **copy it once**. Store it in a password manager.
5. You will paste it into GitHub Secrets in step **A8** (`ZENODO_USER_API_TOKEN`). Do **not** commit it to git.

The Zenodo user behind this token must be able to **manage** the deposition for this specification (owner or **Can manage** on the record).

### A3. Reserve a DOI (keep the browser tab open)

1. Open [https://zenodo.org/uploads/new](https://zenodo.org/uploads/new)
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
3. Commit and push to `main`.

Example commit message: `Embed reserved Zenodo DOI`.

### A5. Create the first GitHub Release from `main`

1. Open your repo → **Releases** → **Create a new release**
  (or: `https://github.com/OWNER/REPO/releases/new`).
2. Choose **Choose a tag** → type e.g. `v0.1.0` → create tag on `main`.
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
- **Main ID** = stable id shown on the record page (never changes; Zenodo may label it “Concept DOI”)

Copy both. Example record page: [https://doi.org/10.5281/zenodo.21759353](https://doi.org/10.5281/zenodo.21759353)  

On the record, note:

- Main ID (stable)  
- Latest version’s numeric record id (the number after `zenodo.` in the *version* / Sub-ID, e.g. `21759354`) — only if you use the optional deposition secret



### A7. Switch the spec to the Main ID (stable forever)

1. Edit `spec/spec-head.md` again.
2. Replace the reserved Sub-ID link with the **Main ID**:
  ```markdown
   **DOI**

   <https://doi.org/10.5281/zenodo.YOUR_MAIN_ID>
  ```
3. Commit and push to `main`.

You usually never change this line again.

### A8. Wire GitHub so updates are automatic (paste the token here)

This is where the token from **A2** goes. GitHub stores it **per repository**; the token itself still belongs to **one Zenodo user** (see [Who owns what](#who-owns-what-tokens-secrets-and-permissions)).

1. Open the GitHub repo → **Settings** → **Secrets and variables** → **Actions**.
  Direct URL: [https://github.com/blockchainbird/test-zenodo/settings/secrets/actions](https://github.com/blockchainbird/test-zenodo/settings/secrets/actions)  
   (other repos: `https://github.com/OWNER/REPO/settings/secrets/actions`)
2. Under **Repository secrets**, click **New repository secret**.

**Warning:** `ZENODO_SPEC_MAIN_DOI` is a **variable**, not a secret. If you create it as a secret, the Action will not see it and will fail with “Set variable ZENODO_SPEC_MAIN_DOI”.

#### Secret (required) — bound to a **Zenodo user**


| Name                    | Value                                                                                                                                                   |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ZENODO_USER_API_TOKEN` | Paste the token from **A2** (full string, no spaces, no trailing newline). Same Zenodo user token may be reused in other spec repos if that user can manage those specs too. |




#### Variable (required in practice) — bound to **this specification**

Open the **Variables** tab on the same Settings page → **New repository variable**.


| Name                   | Value                                                                                                                           |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `ZENODO_SPEC_MAIN_DOI` | **This repo’s** Main ID only, e.g. `10.5281/zenodo.21759353`. One Main ID per specification — not shared across ACDC/CESR/KERI. |




#### Secret (optional if Main DOI variable is set)


| Name                               | Value                                             |
| ---------------------------------- | ------------------------------------------------- |
| `ZENODO_SPEC_LATEST_DEPOSITION_ID` | Numeric id of the *latest published* version only |


Prefer `ZENODO_SPEC_MAIN_DOI`. Then you do not need to bump `ZENODO_SPEC_LATEST_DEPOSITION_ID` after every release.

Confirm `.github/workflows/zenodo-publish.yml` exists on `main` (it does in this test repo).

#### Migrating from old names (this test repo only)

Skip this on a new spec repo. Only relevant if you still have names from an earlier draft of this workflow:

| Old name               | New name                           | Type                                             |
| ---------------------- | ---------------------------------- | ------------------------------------------------ |
| `ZENODO_TOKEN`         | `ZENODO_USER_API_TOKEN`            | Secret                                           |
| `ZENODO_CONCEPT_DOI`   | `ZENODO_SPEC_MAIN_DOI`             | Variable                                         |
| `ZENODO_DEPOSITION_ID` | `ZENODO_SPEC_LATEST_DEPOSITION_ID` | Secret (optional; can delete if Main DOI is set) |

Create the new ones with the same values, then delete the old ones. GitHub does not rename secrets in place.

---



## Part B — Forever after (about 2 minutes per update)

You do **not** open Zenodo for normal updates.

1. Merge whatever you want onto `main`.
2. Create a **new** GitHub Release / tag from `main` (e.g. `v0.2.0`, `v1.1.0`).
  - Do **not** reuse an old tag.
3. Wait for the Action **Publish release to Zenodo (update)** to finish green.
4. Done. Main ID is unchanged. Zenodo has a new Sub-ID for this snapshot.

Check:

- Actions tab → latest run of `zenodo-publish.yml` → success  
- [https://doi.org/YOUR_MAIN_ID](https://doi.org/YOUR_MAIN_ID) → shows the new version

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

- Work on `main` of the original repo.  
- Reserve DOI **before** first publish if you need the DOI inside the first ZIP.  
- After first publish, keep only the **Main ID** in the markdown.  
- Use **New version** for updates (the Action does this).  
- Create a **new tag** for every publish.



### Don’t

- Don’t create a second unrelated upload at [https://zenodo.org/uploads/new](https://zenodo.org/uploads/new) for the same spec.  
- Don’t expect the Action to do the **first** publish.  
- Don’t commit `ZENODO_USER_API_TOKEN` into the repo.  
- Don’t overwrite files on an already-published Zenodo version as your normal update path.  
- Don’t put `ZENODO_SPEC_MAIN_DOI` in **Secrets**. It must be a **Variable**.
- Don’t put a new Sub-ID into the spec on every release if the Main ID is already there.

---



## If something breaks (minimum debugging)



### Action fails: missing secrets

Error mentions `ZENODO_USER_API_TOKEN`, `ZENODO_SPEC_MAIN_DOI`, or deposition id.

→ Re-do **A8**. Token scopes must include `deposit:write` and `deposit:actions`.

### Action fails: “Set variable ZENODO_SPEC_MAIN_DOI”

The Main ID was stored as a **secret**, or the variable name is misspelled.

→ **Settings → Secrets and variables → Actions → Variables** (not Secrets). Name must be exactly `ZENODO_SPEC_MAIN_DOI`. Value like `10.5281/zenodo.21759353`.

### Action fails: “Please remove all files first” / cannot create new version

An unpublished **New version** draft is leftover from a previous failed run. The current workflow tries to discard those automatically.

If it still fails: https://zenodo.org/deposit/ → unpublished draft for this spec → **Discard** → re-run the Action.

### Action fails: upload HTTP 400 / “Empty files are not accepted”

The ZIP on the runner is fine; Zenodo rejected the **upload transport**. The current workflow uploads with Python (`Content-Length` + `application/octet-stream`) for that reason. Confirm `main` has that workflow, then:

1. Discard any leftover unpublished draft (see above).
2. Re-run the Action.

This is **not** rate limiting. Rate limits return HTTP **429**.

### Action fails: 403 / permission denied from Zenodo

The repo’s `ZENODO_USER_API_TOKEN` is valid but the **Zenodo user who owns that token** cannot manage this spec’s record.

→ On Zenodo, grant that user **Can manage** on the record, or replace the secret with a token from an account that already has access. Releasers on GitHub do not need to change anything.

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
3. Update GitHub secret `ZENODO_USER_API_TOKEN`.

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


| Item                             | Value                                                                                          |
| -------------------------------- | ---------------------------------------------------------------------------------------------- |
| GitHub                           | [https://github.com/blockchainbird/test-zenodo](https://github.com/blockchainbird/test-zenodo) |
| Main ID                          | [https://doi.org/10.5281/zenodo.21759353](https://doi.org/10.5281/zenodo.21759353)             |
| First Sub-ID (`v0.1.0`, manual)  | [https://doi.org/10.5281/zenodo.21759354](https://doi.org/10.5281/zenodo.21759354)             |
| Second Sub-ID (`v0.2.0`, Action) | [https://doi.org/10.5281/zenodo.21759414](https://doi.org/10.5281/zenodo.21759414)             |
| Later Sub-ID (`v0.11.0`, Action) | [https://doi.org/10.5281/zenodo.22093110](https://doi.org/10.5281/zenodo.22093110)             |
| Workflow                         | `.github/workflows/zenodo-publish.yml`                                                         |
| Spec DOI line                    | `spec/spec-head.md`                                                                            |




### Local Spec-Up-T commands (unrelated to Zenodo)

```bash
cd test-zenodo
npm install
npm run render   # or: npm run menu
```

---



## One-page summary

1. **Once:** reserve DOI → paste Sub-ID in spec → release `v0.1.0` → upload ZIP on Zenodo → Publish → paste Main ID in spec → set secret `ZENODO_USER_API_TOKEN` + **variable** `ZENODO_SPEC_MAIN_DOI` (this repo’s Main ID).
2. **Later:** anyone with GitHub Release permission creates a new release; the Action uses the **repo’s** secrets — not the releaser’s Zenodo login.
3. **Cite forever:** the Main ID (`https://doi.org/10.5281/zenodo.…`).
4. **Copy to another spec:** same YAML file; new secret/variable on that repo; different Main ID per spec.

