# BMH Equipment → Coric Equipment landing site

A single-page static site that replaces the Shopify-hosted `bmhinc.com` storefront.
It announces that **BMH Equipment is now Coric Equipment** and routes every link to the
appropriate page on `coricind.com`. Built to be hosted on **Azure Static Web Apps**.

Same *approach* as the `allstatewebsite` repo (allstateequipment.com → coricind.com), but the
design keeps **BMH's own look and feel**, sampled from the live Shopify site (Warehouse theme):
navy header `#163356` with the white BMH outline logo, dark red accents `#a2090d`, warm
off-white background `#f7f4f2`, light footer, and the Source Sans Pro typeface (loaded from
Google Fonts as "Source Sans 3"). The layout is a lightly modernized single page: announcement
strip → navy masthead → nav bar → rebrand hero → six banner-style link tiles → the four
value-prop items from the old homepage → light footer.

## What's here

```
src/                                 <- the deployable site (app root)
  index.html                         <- the landing page (self-contained HTML + CSS)
  staticwebapp.config.json           <- Azure SWA routing/headers config
  images/
    bmh-logo-white.png               <- white BMH logo for the navy header (from the Shopify CDN)
    bmh-logo-navy.png                <- navy BMH logo (og:image / spare)
    coric-logo.svg                   <- Coric logo (from coricind.com/brand/coric-logo.svg, spare)
    favicon-96x96.png                <- favicon (same one the Shopify site uses)
.github/workflows/azure-static-web-apps.yml   <- CI/CD for the GitHub-based deploy path
```

`staticwebapp.config.json` rewrites all unknown paths (e.g. old Shopify deep links like
`/collections/pallet-rack` or `/products/...`) to the landing page, so no inbound link 404s.

## Preview locally

Any static file server works. For example:

```powershell
npx serve src
# or, if you have the SWA CLI:
swa start src
```

Then open the printed URL.

## Deploy to Azure Static Web Apps

### Option A — SWA CLI (fastest, no GitHub required)

1. Install the CLI and sign in to Azure:
   ```powershell
   npm install -g @azure/static-web-apps-cli
   az login                                 # requires Azure CLI
   ```
2. Create the Static Web App (Free tier) once:
   ```powershell
   az staticwebapp create `
     --name bmh-coric-landing `
     --resource-group <your-resource-group> `
     --location eastus2 `
     --sku Free
   ```
3. Grab the deployment token and deploy the `src` folder:
   ```powershell
   $token = az staticwebapp secrets list --name bmh-coric-landing --query "properties.apiKey" -o tsv
   swa deploy ./src --env production --deployment-token $token
   ```

### Option B — GitHub Actions (CI/CD)

1. Push this folder to a GitHub repo (default branch `main`).
2. In the Azure Portal: **Create resource → Static Web App**, choose **GitHub** as the source,
   pick the repo/branch, and set **App location = `src`**, **Api location = (blank)**,
   **Output location = (blank)**. Azure commits a workflow and supplies the deployment token
   automatically.
3. If you prefer the workflow already in `.github/workflows/`, instead create the SWA without a
   source, then add the deployment token as a repo secret named
   `AZURE_STATIC_WEB_APPS_API_TOKEN`. Every push to `main` deploys.

## Point the domain at Azure

Once deployed, the app gets a `*.azurestaticapps.net` URL. To serve it at
`bmhinc.com` / `www.bmhinc.com`:

1. In the SWA resource → **Custom domains**, add `www.bmhinc.com`
   (CNAME to the azurestaticapps.net host) and the apex `bmhinc.com`
   (use the ALIAS/ANAME or the TXT-validated apex option Azure provides).
2. Update DNS at the registrar to point at Azure, then remove the Shopify records
   (Shopify uses an A record to `23.227.38.x` and a CNAME to `shops.myshopify.com`).
3. Once DNS has cut over, cancel/downgrade the Shopify subscription so it stops
   serving the old store.

## Notes / things to confirm

- All outbound links point to `coricind.com` and every target returned 200 when checked
  (Aug 2026): Products → `/products`, Equipment → `/equipment`, Rentals → `/services/rentals`,
  Parts → `/services/parts`, Services → `/services`, Service & Repairs →
  `/services/service-and-repairs`, Planned Maintenance → `/services/planned-maintenance`,
  Get a Quote / Contact → `/contact`, Newsroom → `/newsroom`, About → `/about`,
  Careers → `/about/careers`, Why Coric → `/why-coric`, Locations → `/locations`.
- Header phone is BMH's main line **800.608.8300**; footer lists four branch phones
  (Dowagiac MI, Lansing IL, Milford NH, Binghamton NY — Caledonia MI intentionally
  omitted). Update if any of these numbers are being retired in the transition.
- Footer email is `BMHSales@bmhinc.com` — confirm that mailbox survives the Shopify cutover
  (email/MX records are separate from web DNS, but double-check before removing anything).
- Fonts load from Google Fonts (the only external dependency). Logos/favicon were copied out
  of the Shopify CDN into `src/images/` so nothing breaks when the store is shut down.
- Review the hero/announcement wording ("part of Coric Equipment", "BMH is now Coric
  Equipment") with marketing — swap in the official transition language if there is any.
