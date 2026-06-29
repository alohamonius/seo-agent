---
description: Set up seo-agent for this project — create the service account, grant access, write config
---

Walk the user through configuring the seo-agent for the **current project**. Be concrete and do the file/CLI work for them where you can; only the Google Cloud Console clicks and the GA4/GSC access grants must be done by the user.

Read `${CLAUDE_PLUGIN_ROOT}/SETUP.md` and follow it. The essence:

1. **Create the service account** (the "user" that reads the APIs):
   - Either via `gcloud` (offer to run these, substituting a project id):
     ```bash
     gcloud iam service-accounts create seo-agent --display-name "SEO Agent"
     gcloud services enable searchconsole.googleapis.com analyticsdata.googleapis.com --project <PROJECT_ID>
     gcloud iam service-accounts keys create seo-agent-key.json \
       --iam-account seo-agent@<PROJECT_ID>.iam.gserviceaccount.com
     ```
   - Or via the Console (https://console.cloud.google.com/iam-admin/serviceaccounts) — create the account, enable **Search Console API** + **Google Analytics Data API**, then create and download a JSON key.
2. **Grant the service-account email access** (it looks like `seo-agent@<project>.iam.gserviceaccount.com`):
   - Search Console → Settings → Users and permissions → add it as **Full** (or Restricted) user.
   - GA4 → Admin → Property Access Management → add it as **Viewer**.
3. **Write the project config**: create `.claude/seo/.env` (copy `${CLAUDE_PLUGIN_ROOT}/.env.example`), drop the key JSON next to it as `service-account.json`, and set:
   - `GSC_SERVICE_ACCOUNT_KEY_FILE=service-account.json`
   - `GSC_SITE_URL=` (the exact GSC property — `https://example.com/` or `sc-domain:example.com`)
   - `GA4_PROPERTY_ID=properties/<numeric id>` (GA4 Admin → Property Settings → Property ID)
   - (optional) `DATAFORSEO_LOGIN` / `DATAFORSEO_PASSWORD`, and niche tuning like `SEO_STRONG_DOMAINS`.
   - Ensure `.claude/seo/` is git-ignored (the key is a secret).
4. **Verify**: `npm install --prefix "${CLAUDE_PLUGIN_ROOT}/scripts"` then `npx tsx "${CLAUDE_PLUGIN_ROOT}/scripts/ping.ts"`. All configured integrations should report OK.

If the user already has a service account from another project, they can reuse the same key JSON — they only need to grant that email access to this project's GSC property and GA4 property, and point this project's `.env` at it.
