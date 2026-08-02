# test-zenodo

Spec-Up-T test repository for automating Zenodo DOI **updates** via GitHub Actions.

Manual first-time DOI creation follows:
https://trustoverip.github.io/spec-up-t-website/docs/developer-documentation/create-zenodo-doi

## Local setup

```bash
cd test-zenodo
npm install
npm run render   # or: npm run menu
```

## First-time Zenodo DOI (manual — not automated)

1. Create a Zenodo personal access token at
   https://zenodo.org/account/settings/applications/tokens/new/
   with scopes **deposit:write** and **deposit:actions**.
2. Reserve a DOI at https://zenodo.org/uploads/new → “No, I need one” → Reserve DOI.
3. Put the reserved Version DOI in `spec/spec-head.md` as:

   ```markdown
   <https://doi.org/10.5281/zenodo.XXXXXXXX>
   ```

4. Commit to `main`, create GitHub Release `v0.1.0`, download the release ZIP.
5. Upload that ZIP to the Zenodo draft, fill metadata, **Publish**.
6. Note:
   - **Concept DOI** (main ID, never changes) — prefer this in `spec-head.md` after first publish
   - **Version DOI** (sub-ID) — the reserved one for v0.1.0
   - **Deposition / record ID** of the published version (numeric)

7. Add GitHub repository secrets/variables:
   - Secret `ZENODO_TOKEN` = your token
   - Secret `ZENODO_DEPOSITION_ID` = latest published version numeric ID  
     **or** variable `ZENODO_CONCEPT_DOI` = `10.5281/zenodo.<conceptrecid>`

## Automated updates (GitHub Action)

Workflow: `.github/workflows/zenodo-publish.yml`

Triggers on **Release published** (and `workflow_dispatch`).

It creates a Zenodo **New version**, uploads the release source ZIP, and publishes.
Do **not** use this for the first publish.

After each automated publish, if you use `ZENODO_DEPOSITION_ID`, update it to the new version id
(the Action summary prints it). Prefer `ZENODO_CONCEPT_DOI` so you do not need to bump the secret.
