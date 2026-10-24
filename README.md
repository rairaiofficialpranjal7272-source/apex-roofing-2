# Apex Roofing Co.

A complete responsive roofing website built with React, TypeScript, Vite, and an Express backend. The final Apex Roofing brief is implemented rather than the earlier Sabrina Residences brief.

## Run

Requires **Node.js 22 LTS** and npm. Use `nvm use` to select the version in `.nvmrc`.

```bash
npm ci
npm run dev
# http://localhost:3000
```

Production:

```bash
npm run build
npm start
```

The server binds to `0.0.0.0` and serves the site and API from the same origin. Browser requests use relative URLs. Run the Node server, not just a static-file host, for the booking workflow.

## Project layout and TS18003 deployment checks

The complete source tree must be deployed, not only the configuration files or generated HTML:

```text
package.json
package-lock.json
tsconfig.json
vite.config.ts
index.html
server.mjs
src/
  main.tsx
  AppointmentForm.tsx
  ui.tsx
  config.ts
  styles.css
  fonts.css
  vite-env.d.ts
public/
  favicon.svg
  images/                 # 4 required WebP images
  fonts/                  # 8 self-hosted WOFF2 fonts
scripts/
tests/
```

Run `npm run check:project` to check the complete source/assets before building. The build remains **`tsc --noEmit && vite build`**; it is not replaced by the structure checker. `tsconfig.json` requires the real application entrypoint and Vite config and includes all source `.ts`/`.tsx` files. There are no project references pointing outside this directory.

The original complete tree builds successfully. **TS18003** means the configuration used in the failing build found no input files; it does not by itself prove the error is in your React code. In an isolated reproduction, deploying the original config without `src/` produces exactly this error. Check the deployed commit, uploaded files, selected project root, include/exclude settings, and any upload-ignore rules. See **`BUILD-FIX.md`** for the evidence and exact changed files.

For a **Vercel Vite frontend build**, select the directory containing all the files above as **Root Directory** (repository root if they are at the top level). Use Node **22.x**, Install Command **`npm ci --include=dev`**, Build Command **`npm run build`**, Framework Preset **Vite**, and Output Directory **`dist`**. Do not select `src/` or `dist/` as the project root. Commit `src/`, `public/`, both configs, `index.html`, the package files, and all supporting source files. Redeploy the correct commit without reusing the stale build cache.

The corrected source ZIP places `package.json`, `src/`, and the configuration files directly at the archive root. Extract the whole ZIP; do not upload only its top-level loose files. The Express backend is unchanged: these frontend build settings do **not** configure Node API hosting or persistent uploads. Use the full-stack deployment instructions below, or explicitly adapt the backend/storage before expecting the Vercel frontend to handle appointments.

## Included

- AI-generated architectural hero with layered lighting, subtle pointer parallax, ambient particles, animated grid lines, and a glass information card.
- Choreographed word-by-word hero text and scroll-triggered section reveals, with a reduced-motion alternative.
- Three roofing service cards with working estimate preselection.
- Interactive roof components with keyboard-, touch-, and mouse-accessible hotspots.
- Three clearly labeled illustrative project examples, benefits, and a four-step process.
- Full appointment form with accessible inline validation, loading and error states, duplicate-click protection, conditional email requirements, consent, and image uploads.
- Genuine server-side validation and private local persistence; optional live webhook delivery.
- Configurable phone and WhatsApp links; no fabricated contact numbers.
- Honest ZIP availability checker, accessible FAQ accordions, privacy and terms dialogs, dynamic copyright year, and mobile navigation.
- Local optimized WebP images and subsetted WOFF2 Google fonts; no runtime font/CDN dependency.

## Configure the business

Edit **`src/config.ts`** with verified information:

- Company name (also controls the wordmark).
- Phone in E.164 format and a separate display number.
- Email, location, and business hours.
- WhatsApp number: country code and digits, no `+` or punctuation.
- Cities and ZIP codes: `serviceAreas: [{ city: 'Your verified city', zips: ['your actual ZIP'] }]`.
- Image paths.

No real addresses, credentials, reviews, statistics, guarantees, or service locations have been invented. Unconfigured call and WhatsApp controls explain that setup is pending instead of opening fake numbers. WhatsApp sends only the generic inquiry text; it never automatically includes form data. The service-area checker only reports availability for explicitly configured ZIPs.

Update the page title, description, Open Graph values, structured-data company name, and canonical URL in **`index.html`** when publishing under a verified domain. Do not add ratings, credentials, or address metadata unless verified.

## Appointment workflow — what is real now

Without integration credentials, the site operates in **clearly labeled local-preview mode**:

1. The browser validates the fields and sends multipart data to `POST /api/appointments`.
2. The server validates all fields, consent, date, service choices, phone format, ZIP, and conditional email requirements with Zod.
3. Optional uploads are limited to one JPG, PNG, or WebP, 5 MB maximum. The server checks signature bytes and MIME type, uses generated filenames, and does not serve uploaded files publicly.
4. Valid requests are saved as private JSON records in **`data/appointments/`**, with an optional adjacent image. Directory/file permissions are restricted where supported.
5. The result explicitly says **saved locally, not sent**, with a request reference. No email is claimed, and no appointment is confirmed.

There is deliberately no public inbox endpoint. A site administrator with server access can review records directly. Preview test submissions are removed by the test suite. The `data` directory is excluded from source archives and version control.

Set server-only `DATA_DIR` to the absolute path of your private persistent submissions directory (default: `data/appointments`). This local filesystem is suitable for the preview, **not a scalable production booking database**. Workspace snapshot retention is not a booking-data durability guarantee. Before public launch, use persistent protected hosting/storage or a verified delivery integration, restrict administrator access, define retention/deletion policies, and protect backups. Never commit real customer data.

## Connect live request delivery

Copy `.env.example` to `.env`, then configure:

```dotenv
APPOINTMENT_WEBHOOK_URL=https://your-trusted-https-receiver/roofing-requests
WEBHOOK_TOKEN=your-server-side-secret
```

The server automatically loads `.env`. Restart after changing environment settings.

The receiver must accept **multipart/form-data** with:

- `request`: a JSON string with `id`, `createdAt`, validated fields, consent, and the optional image filename.
- `photo`: the optional image binary.
- `Authorization: Bearer <WEBHOOK_TOKEN>` when configured.

Use the request `id` for deduplication. The receiver can be your own server, automation workflow, or a function that writes to Supabase and sends mail through your verified email provider. Store secrets only on the server. An HTTPS endpoint controlled by you is required; no credentials have been provisioned here.

The receiver must return a **2xx** response only after it has accepted the request for processing. A delivery failure or a 15-second timeout returns a visible error, never a false success. The frontend checks `/api/booking-status` and shows connected mode only when a webhook URL is present. A 2xx response means the request was accepted by that integration, not that an appointment has been confirmed or an email has necessarily been sent. Test the receiver, its data retention, and the actual notification path before launch.

## Security and production checklist

- HTTPS, a real company contact, verified service areas, and reviewed business policies.
- Deploy `npm run build` plus `npm start` with a persistent private writable `data/appointments` volume, or replace local storage with your protected database/object storage.
- Basic per-IP process-local rate limiting, an origin check, and a honeypot are included. For public traffic, add shared rate limiting, abuse protection/CAPTCHA, monitoring, and alerting. Configure proxy trust carefully for your infrastructure; do not blindly trust all forwarded IP headers.
- Uploads are checked for size, MIME, and signature. For a public high-volume deployment, decode/re-encode uploads, strip metadata, and add malware scanning/storage lifecycle rules. Keep uploads private.
- The preview privacy/terms content explains actual behavior but is **not a substitute for legal review**. Publish the operator's contact, processors, retention period, and rights/deletion workflow.
- Confirm delivery end-to-end. There are no automatic scheduling, emergency response, insurance, warranty, or hidden-damage diagnosis claims.

## Testing

The self-contained production test command starts an isolated server on port 4173, forces local-only booking mode, runs the browser/API suite, and stops the server:

```bash
npx playwright install --with-deps chromium
npm run build
npm run test:production
```

For an already-running local preview, `npm test` remains available. Do not point these synthetic submission tests at a live customer system. Generated reports and submissions are excluded from Git.

The test suite checks:

- Widths **375, 390, 430, 768, 1024, 1440, and 1920px**, horizontal overflow and broken images.
- Mobile navigation, keyboard roof controls, ZIP checker, FAQ, and dialog dismissal.
- Inline validation, conditional email, file selection, recoverable delivery failure, real local submission storage, and truthful result text.
- Server rejection of invalid fields, past dates, spoofed image bytes, and cross-origin submissions.
- Browser runtime errors and automated axe WCAG A/AA checks on desktop and mobile.

Reports and visual screenshots are written to `test-results/`. Automated accessibility checks do not replace a full manual assistive-technology audit. Tests are designed for local-preview mode and clean up their own records.

## Source layout

```
src/config.ts              Editable company information and service content
src/ui.tsx                 Shared word animation, logo, and section labels
src/AppointmentForm.tsx    Validated booking UI and request states
src/main.tsx               Page sections and interactions
src/styles.css             Responsive design system and animations
src/fonts.css              Self-hosted font faces
vite.config.ts             Explicit project root, React plugin, and build output
server.mjs                 Express API, validation, storage, optional webhook
public/images/*.webp       Optimized generated imagery
public/fonts/*.woff2       Self-hosted fonts
licenses/                  Font license notices
artwork-source/            Original generated PNGs (not needed at runtime)
tests/                     Browser interaction, API, and accessibility checks
```

All architectural photographs are AI-generated illustrative assets, not actual completed Apex projects. Google Fonts Space Grotesk and Inter are supplied under their included SIL Open Font Licenses.

## GitHub publication and review

The project contains a GitHub Actions workflow at `.github/workflows/ci.yml`. It installs locked dependencies, checks repository hygiene, typechecks/builds, installs Chromium, and tests an isolated production server on Node 22. It needs no deployment credentials. **A locally passing build is not a GitHub Actions result until the workflow has actually run on GitHub.**

Recommended branch: `feat/apex-roofing-launch`.

1. Select the destination repository and verify write access. Prefer a private repository until business information and legal content are approved.
2. If it already contains files, clone it, inspect its default branch and contribution rules, then create the working branch from that default branch. Integrate only the website files, resolving conflicting files explicitly; do not replace unrelated files or force-push.
3. Review `git diff --cached --stat` and `git diff --cached`. Run `npm run security:repository` after staging the intended files.
4. Commit and push the working branch using your authenticated Git client. Do not place access tokens in remote URLs or source files.
5. Open a pull request into the repository's default branch. Describe the local-preview booking limitation, deployment requirements, and test results.
6. In the PR's **Checks** tab, verify the **Website checks / build-and-test** job is green. Review the **Files changed** tab, then use **Squash and merge** (or the repository's approved merge method). Do not merge while checks or review are outstanding.
7. Deploy the merged commit separately. A pushed branch or merged PR alone does not publish the website.

If creating a new repository through GitHub's UI, select the owner/name and visibility intentionally. Do not add a license unless you have decided to license the project; this application is currently marked `UNLICENSED`. The bundled fonts retain their separate open-font licenses.

## Recommended full-stack deployment: Render

This project is **not a static-only website**. Express handles multipart submissions, server-side validation, private file storage, local inbox records, and optional authenticated webhook delivery. GitHub Pages cannot execute Node/Express or store private submission data.

A reviewed optional `render.yaml` blueprint is included. It describes a Node web service with a private persistent disk; applying it may incur charges. No hosting resource has been created by this preparation.

After merging the PR:

1. In Render, choose **New → Web Service** and connect the selected GitHub repository, or review/import the blueprint. Choose the merged default branch and the project directory as the root (repository root if these files are at the top level).
2. Use Node **22**, build command **`npm ci --include=dev && npm run build`**, and start command **`npm start`**.
3. Add a persistent disk mounted at **`/var/data`** and set **`DATA_DIR=/var/data/apex/appointments`**. Without a persistent disk or replacement database/object storage, filesystem submissions can be lost on redeploy. A live webhook does not remove this requirement: this implementation also writes local records and optional image files.
4. Keep `NODE_ENV=production`. Let the host provide `PORT`; the app listens on `0.0.0.0`. Set the health check to **`/api/health`**.
5. Add any real `APPOINTMENT_WEBHOOK_URL` and `WEBHOOK_TOKEN` only through the hosting dashboard's secret settings. Do not put them in `render.yaml`, GitHub source, or frontend `VITE_` variables.
6. Configure real business details in `src/config.ts`, update domain-specific metadata, and review privacy/terms before serving real visitors.
7. Deploy and open the generated HTTPS URL. Check the homepage, `/api/health`, `/api/booking-status`, image/font assets, mobile menu, and form error handling. With your authorization, submit a marked test request and verify receipt/storage/delivery. Remove that test record afterward.
8. Add the custom domain and the DNS records Render supplies. Wait for DNS and HTTPS verification, then repeat the checks on the final public domain.

Railway or Fly.io with a persistent volume are alternatives. Vercel/Netlify would require adapting the Express API and moving uploads/data to protected persistent services; simply uploading `dist/` there does not provide the current backend.

## GitHub Pages (static frontend only — not configured)

**Do not enable Pages expecting the booking backend to work.** Uploading this project's `dist/` alone leaves `/api/booking-status` and `/api/appointments` unavailable. GitHub repository publication is independent of Pages hosting.

If you intentionally split the frontend and backend later:

1. Deploy the backend to a Node-capable host first, with protected persistent storage.
2. Add a public API-base configuration to the frontend and update all `/api/` requests. Configure restrictive CORS and update the backend origin/CSRF checks to explicitly allow the chosen Pages origin. Never expose the webhook token in browser code.
3. Make Vite's base URL and **all image/font paths** work at the repository subpath (`/repository-name/`), or use an approved custom domain. Current root-relative paths assume a root-domain deployment and are not project-Pages-ready.
4. Add a Pages workflow that builds and uploads only `dist/` with `actions/upload-pages-artifact` and deploys it with `actions/deploy-pages`; do not deploy the entire source tree or private data. Grant only `contents: read`, `pages: write`, and `id-token: write`, and use the `github-pages` environment.
5. In **Repository Settings → Pages → Build and deployment**, select **GitHub Actions**. Merge the reviewed Pages workflow, wait for it to pass, and open the URL shown in the deployment.
6. Verify images, fonts, all navigation, and real backend submissions on that public URL before declaring launch complete.

No Pages deployment workflow is included because the current full-stack app should be deployed as one same-origin Node service.
