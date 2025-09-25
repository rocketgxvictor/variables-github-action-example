# Figma to GitHub Tokens Sync - Implementation Scope

## Phase 1 — Prepare GitHub Action ✅
1. Create a workflow file at `.github/workflows/figma-sync.yml`.
2. Configure it to:
   - Use `workflow_dispatch` with an input for `file_key`.
   - Run `npm ci` and then `npm run sync-figma-to-tokens -- --output tokens`.
   - Pass environment variables:
     - `FILE_KEY` = `${{ github.event.inputs.file_key }}`
     - `PERSONAL_ACCESS_TOKEN` = `${{ secrets.GH_ACTION_VARIABLES_SYNC_FIGMA_TOKEN }}`
3. Store your Figma token in GitHub repository secrets as `GH_ACTION_VARIABLES_SYNC_FIGMA_TOKEN`.

---

## Phase 2 — Create Vercel API Route
1. Add a file at `app/api/figma-webhook/route.ts`.
2. Implement it to:
   - Read the raw request body.
   - Validate the HMAC signature using `FIGMA_WEBHOOK_SECRET`.
   - Filter for `LIBRARY_PUBLISH` events only.
   - Extract `file_key` from the payload.
   - Call GitHub's `workflow_dispatch` endpoint with:
     - `ref` set to the target branch (e.g., `main`).
     - `inputs.file_key` set from the webhook payload.

---

## Phase 3 — Configure Vercel Environment
1. In the Vercel dashboard, add environment variables:
   - `FIGMA_WEBHOOK_SECRET` = your chosen webhook secret.
   - `GITHUB_PAT` = GitHub personal access token with `repo` and `workflow` scopes (used only by Vercel).
   - `GH_OWNER` = GitHub org or username.
   - `GH_REPO` = name of your repository.
   - `GH_WORKFLOW` = workflow filename (e.g., `figma-sync.yml`) or workflow ID.
   - `GH_REF` = branch to run workflow on (e.g., `main`).
   - Optional: `FIGMA_DEFAULT_FILE_KEY` = fallback file key.
2. Deploy to Vercel and confirm the function URL (e.g., `https://<project>.vercel.app/api/figma-webhook`).

---

## Phase 4 — Register Figma Webhook
1. In Figma, create a webhook for your design system file.
2. Configure it with:
   - Event type: `LIBRARY_PUBLISH`.
   - Endpoint URL: your Vercel route.
   - Secret: must match `FIGMA_WEBHOOK_SECRET`.
3. Confirm the webhook is active.

---

## Phase 5 — Validate End to End
1. Trigger the GitHub Action manually from the Actions tab with a test `file_key`.
2. Publish a change in the Figma library.
3. Check:
   - Vercel logs show a successful POST and "ok" response.
   - GitHub Actions run starts with the correct `inputs.file_key`.
   - Tokens are updated in the repo as expected.

---

## Phase 6 — Hardening
1. Add simple console logs in the Vercel function for debugging.
2. Ensure the GitHub PAT is scoped minimally.
3. Keep constant-time signature comparison for HMAC verification.
4. Optionally add a retry/backoff if GitHub API returns non-200.

---

## Done Criteria
- Publishing a design system change in Figma triggers the Vercel function.
- Vercel validates the request and calls `workflow_dispatch`.
- GitHub Action runs with the `file_key` provided.
- Tokens sync into the codebase without manual input.
