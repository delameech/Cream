# Deploying the LB 200 Lab Site to Azure Static Web Apps

The `deploy/` folder is the complete, safe-to-publish package: `index.html` (the site), `staticwebapp.config.json` (routing), and `swa-cli.config.json` (CLI config). Run everything below **from inside this folder** on your own machine.

## Do not deploy the parent folder

The Ice Cream folder contains the Modernist Cuisine PDFs and the Carpigiani manual. Publishing those to a public URL would be distributing copyrighted books. Deploy only this `deploy/` folder. (Consequence: the Document Library's relative links to the PDFs, volumes, and workbooks will 404 on the public site — they keep working when you open the local copy from the Ice Cream folder. If you want the six volume `.md` files and the two `.xlsx` workbooks downloadable publicly, copy those specific files in here — they're your own content. Never the PDFs.)

## One-time setup (your machine, not this session)

Requires Node.js and an Azure account.

```bash
npm install -D @azure/static-web-apps-cli
npx swa init --yes          # will detect the static site; config already provided
```

## Deploy

```bash
# no build step needed - it's a single static file; swa build is a no-op here
npx swa login --resource-group swa-tutorial --app-name swa-demo-site
npx swa deploy --env production
```

Notes:
- The resource group and app name must exist first. Create them once with:
  `az group create -n swa-tutorial -l westus2` and
  `az staticwebapp create -n swa-demo-site -g swa-tutorial`
  (or pick your own names — e.g. `-g ice-cream-lab -n lb200-lab` — and use them in both commands above).
- The deploy command prints your public URL (something like `https://<generated>.azurestaticapps.net`).
- Free tier is fine: one static file, no backend, no API.

## Password protection

`deploy/index.html` is now a **password-gated, encrypted** version of the site (password: `Meridian`). This is real encryption, not a hidden JS check: the entire site is AES-GCM-encrypted with a key derived from the password (PBKDF2, 310k iterations), so the deployed file contains only ciphertext — neither the content nor the password appears anywhere in the source. Wrong password = undecryptable. The unlock persists for the browser session (re-prompts in a new tab/session).

Two caveats: (1) "Meridian" is a short dictionary-style password — fine for keeping the site private from casual visitors, not for defending against someone determined to brute-force it; a longer passphrase would be stronger. (2) If you lose the password, the deployed copy is unrecoverable — but your unencrypted master (`LB200-Ice-Cream-Laboratory.html` in the Ice Cream folder) is untouched.

## Updating the site

After any change to the master file, the encrypted copy must be regenerated (ask Claude to "re-encrypt the deploy copy" after edits), then `npx swa deploy --env production` again. Don't copy the master into `deploy/` directly — that would publish the unprotected version.

## Tear down

```bash
az group delete -n swa-tutorial
```
