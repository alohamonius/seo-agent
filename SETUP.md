# Setup — creating the service account ("the user") and configuring a project

The GA4 and Search Console scripts authenticate as a **Google Cloud service
account** — a non-human "user" with its own email and a private key, that you
grant read access to your Analytics property and Search Console site. You create
it **once** and can reuse the same key across every project; per project you just
grant that email access and point the project's `.env` at the key.

This is the part people forget. Here it is end to end.

---

## 1. Create the service account (the "user")

You need a Google Cloud **project** (any one will do — it's just the billing/API
container; the APIs used here are free). Then create the service account inside it.

### Option A — `gcloud` CLI (fastest)

```bash
# Pick or create a GCP project
gcloud projects create my-seo-project           # or reuse an existing one
PROJECT_ID=my-seo-project

# Create the service account — this is "the user"
gcloud iam service-accounts create seo-agent \
  --display-name "SEO Agent" \
  --project "$PROJECT_ID"

# Its email (you'll grant THIS access in GA4 + GSC):
#   seo-agent@<PROJECT_ID>.iam.gserviceaccount.com

# Enable the two APIs the scripts call
gcloud services enable searchconsole.googleapis.com analyticsdata.googleapis.com \
  --project "$PROJECT_ID"

# Create and download a JSON key
gcloud iam service-accounts keys create service-account.json \
  --iam-account "seo-agent@${PROJECT_ID}.iam.gserviceaccount.com"
```

`service-account.json` is the secret. Keep it out of git.

### Option B — Google Cloud Console (clicks)

1. https://console.cloud.google.com/iam-admin/serviceaccounts → pick/create a project → **Create service account**. Name it `seo-agent`. Skip the optional role grants (it doesn't need project IAM roles — access is granted inside GA4/GSC instead). Create.
2. Open the new account → **Keys** → **Add key → Create new key → JSON** → download. That's your `service-account.json`.
3. Enable the APIs: **APIs & Services → Enable APIs and services**, enable **Google Search Console API** and **Google Analytics Data API**. (Or visit the API pages directly and click Enable.)

Either way you end up with: a key JSON file, and a service-account **email** that looks like `seo-agent@<project>.iam.gserviceaccount.com`. The email is also inside the JSON as `client_email`.

---

## 2. Grant that email access to your data

The service account can't see anything until you invite its email — exactly like
sharing with a person.

**Search Console** (https://search.google.com/search-console)
- Settings → **Users and permissions** → **Add user**
- Paste the service-account email, permission **Full** (or Restricted — read is enough for reporting).

**Google Analytics 4** (https://analytics.google.com)
- Admin (gear) → under the **Property** column → **Property Access Management**
- **+** → add the service-account email with role **Viewer**.

---

## 3. Find the two identifiers the scripts need

- **`GSC_SITE_URL`** — must match the property *exactly* as it appears in Search
  Console. A **Domain** property is `sc-domain:example.com`. A **URL-prefix**
  property is the full origin with trailing slash, e.g. `https://example.com/`.
- **`GA4_PROPERTY_ID`** — GA4 Admin → **Property Settings** → **Property ID**
  (a number like `529789574`). In config it's `properties/529789574`. Note: this
  is the numeric Property ID, **not** the `G-XXXXXXX` measurement/tag id you put
  on the site.

---

## 4. Write the project config

In the project you want to analyze:

```bash
mkdir -p .claude/seo
cp <plugin>/.env.example .claude/seo/.env
mv service-account.json .claude/seo/service-account.json
```

Edit `.claude/seo/.env`:

```
GSC_SERVICE_ACCOUNT_KEY_FILE=service-account.json
GSC_SITE_URL=https://example.com/
GA4_PROPERTY_ID=properties/123456789
# optional: DATAFORSEO_LOGIN / DATAFORSEO_PASSWORD, SEO_STRONG_DOMAINS, ...
```

Make sure the key and env are git-ignored. The conventional way:

```
echo ".claude/seo/" >> .gitignore
```

---

## 5. Verify

```bash
npm install --prefix <plugin>/scripts        # one time per plugin install
npx tsx <plugin>/scripts/ping.ts
```

You should see `OK` for GSC and GA4 (and DataForSEO if you set its creds). A
`FAIL` with `403` almost always means step 2 was skipped or the email/property
doesn't match; a `404` on GSC usually means `GSC_SITE_URL` doesn't exactly match
the property string.

---

## Reusing across projects

The service account is global. For each new project: grant its email access to
that project's GSC + GA4 (step 2), then create that project's `.claude/seo/.env`
pointing at the same key JSON (or paste the key inline). No new service account
needed.
