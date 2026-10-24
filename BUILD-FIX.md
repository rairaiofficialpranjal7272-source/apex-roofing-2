# TS18003 diagnosis and corrected source package

## What the inspection established

The provided `/home/user/apex` project already contained the complete React source (`src/main.tsx`, `src/AppointmentForm.tsx`, `src/ui.tsx`, `src/config.ts`, styles, and Vite declarations), `index.html`, all four optimized house/roof images, all eight fonts, and the Express server. Those files were tracked in the local repository. No source or asset restoration was necessary in this workspace.

There was one TypeScript config (`tsconfig.json`), no project references or extended configs, and no Vite configuration file. The original `include: ["src"]` is valid: the original unmodified project passed `tsc --noEmit && vite build` on Node 22.23.4. Therefore it would be inaccurate to claim that this include value was itself broken.

In a separate test directory containing the original config but **no `src` directory**, TypeScript reproduced the reported **TS18003** error exactly. This establishes a reproducible missing-input failure mode, not proof of which files are in your remote Vercel checkout. The Vercel deployment URL, selected commit, root-directory setting, and complete remote log were not provided, so those remote settings cannot be verified here.

The remaining remote cause is to be checked among an incomplete source upload/checkout, a different deployed commit/configuration, a wrongly selected project root, or another exclusion/path mismatch in the deployed tree. A different config may also cause this error. This package supplies the complete known-working tree and explicit project paths; it does not pretend to repair an uninspected Vercel dashboard setting.

## Exact file changes

| File                        | Change                                                                                                                                                                                                                                                                 |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tsconfig.json`             | Explicitly requires the real `src/main.tsx`, `src/vite-env.d.ts`, and `vite.config.ts` files; includes all `src/**/*.ts` and `src/**/*.tsx`; excludes generated/private directories; enables consistent file-name casing. Strict checking and `noEmit` remain enabled. |
| `vite.config.ts`            | New real Vite configuration with the React plugin, root relative to the config file, `public/` assets, and `dist/` output.                                                                                                                                             |
| `scripts/check-project.mjs` | New completeness check for the real React source, local imports, entrypoint, Express server, package/lockfile, and referenced public assets. Exits nonzero if required files are missing.                                                                              |
| `package.json`              | Adds `npm run check:project`. The build command remains **`tsc --noEmit && vite build`**. No dependency versions changed.                                                                                                                                              |
| `.github/workflows/ci.yml`  | Runs the completeness check before the existing strict production build and browser/API tests.                                                                                                                                                                         |
| `README.md`                 | Adds the source layout, diagnosis, exact Vercel build settings, deployment-root checks, and archive instructions.                                                                                                                                                      |
| `BUILD-FIX.md`              | This change/verification report.                                                                                                                                                                                                                                       |

`package-lock.json` is included **unchanged**; there was no dependency change requiring lockfile regeneration. `src/`, `public/`, `index.html`, and `server.mjs` are also byte-for-byte unchanged. The design, images, responsive CSS, anchor navigation, and API endpoints are preserved.

## Why this does not hide TypeScript failures

No empty TypeScript file, suppressed diagnostic, disabled typecheck, reduced strictness, or JavaScript-only build was introduced. Real application files are required. In a negative test, using the corrected config without those files still fails compilation, now with explicit **TS6053 file-not-found** diagnostics. The optional structure checker is additional validation, not a replacement for TypeScript.

## Verify locally

Use Node **22**:

```bash
npm ci --include=dev
npm run check:project
npm run typecheck
npm run build
npx playwright install --with-deps chromium
npm run test:production
```

The isolated production tests cover all seven requested widths, missing images/overflow, runtime errors, accessibility scans, navigation/hotspots, validation, uploads, server errors, and genuine local request persistence. Synthetic test records are deleted afterward.

## Vercel settings to check

1. Extract the source archive. Its contents are at the archive root, not under an extra `apex/` or `apex-roofing/` directory. Commit **all** source/config files and `public/` assets, including this package's `src/` directory and `package-lock.json`.
2. Set Vercel **Root Directory** to the directory containing `package.json`, `tsconfig.json`, `vite.config.ts`, `index.html`, `src/`, and `public/`. If these are at the repository root, select the repository root. Do not select `src/`, `dist/`, or a directory containing only configuration files. If using a monorepo, select the actual website subdirectory.
3. For a Vite frontend build, use Node **22.x**, Framework Preset **Vite**, Install Command **`npm ci --include=dev`**, Build Command **`npm run build`**, and Output Directory **`dist`** (relative to that selected root).
4. Confirm the deployment uses the commit containing the complete source tree. Check repository and any `.vercelignore` rules for accidental `src/` or `public/` exclusions. No `.vercelignore` is present in this corrected project, and `.gitignore` does not exclude source/assets.
5. Redeploy that commit without reusing the old build cache. Confirm the log contains both the TypeScript step and the Vite production build.

**Backend limitation:** Those Vite settings describe the frontend build, not a completed full-stack deployment. The Express backend is preserved. It needs a suitable Node runtime and private persistent storage, or an explicit serverless/external-storage adaptation for Vercel. Uploading `dist/` alone does not host `/api/appointments` or its private filesystem inbox. No placeholder API destination, routing rewrite, or backend deletion was added. Existing Render/Node deployment instructions remain in `README.md`.

## Verification status

- Original complete-tree build: passed on Node 22.23.4.
- Original config with missing source: reproduced TS18003 in an isolated fixture.
- Corrected project structure check and strict build: passed.
- Corrected config with missing required files: correctly fails (no dummy-input workaround).
- Clean extracted source archive: installed from the included lockfile, typechecked, built, and tested with the production Express server.
- Public Vercel deployment: **not verified or modified**; deployment credentials/settings were not supplied.

The final response identifies the test results from the clean extracted source package. A passing source build is not a claim that the remote Vercel service has been redeployed.
