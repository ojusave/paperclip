# Deploying Paperclip on Render

Render's filesystem is **ephemeral**: anything written to disk is lost on redeploy or restart. Paperclip normally uses local files for:

1. **File storage** – attachments, uploads, and other blobs
2. **Instance data** – config, run logs, and other data under `PAPERCLIP_HOME`

On Render you handle these as follows.

## Auth and public URL

The blueprint uses **authenticated + private** deployment. You only need to set **BETTER_AUTH_SECRET** in the Dashboard (e.g. `openssl rand -base64 32`). **PAPERCLIP_PUBLIC_URL** is not required for private deployment; auth uses the request host (auto mode). Set it only if you use public exposure or a custom domain.

## File storage: default is local (ephemeral)

The blueprint does **not** require S3. Storage defaults to **local_disk**: attachments and run logs are written to the instance filesystem. On Render that disk is ephemeral, so they are **lost on redeploy**. The app runs and is fine for testing.

For **persistent** file storage, set in the Dashboard:

- **PAPERCLIP_STORAGE_PROVIDER** = `s3`
- **PAPERCLIP_STORAGE_S3_BUCKET** – your bucket name
- **AWS_ACCESS_KEY_ID** / **AWS_SECRET_ACCESS_KEY** (or equivalent for Cloudflare R2, Backblaze B2, etc.)

Optional: **PAPERCLIP_STORAGE_S3_REGION**, **PAPERCLIP_STORAGE_S3_ENDPOINT** for custom endpoints.

## Run logs and instance data: ephemeral vs persistent

- **Run logs** and other instance data under `PAPERCLIP_HOME` are written to **local disk**.
- On Render's **free tier** that disk is ephemeral: run logs are **lost on redeploy**. The app and DB work; you just don't retain run history across deploys.
- For **persistent** run logs and instance data:
  - Use a **paid** web service (free tier cannot use disks).
  - In the Render Dashboard, add a **Persistent Disk** (e.g. mount path `/opt/render/project/src/paperclip-data`).
  - Set **PAPERCLIP_HOME** = `/opt/render/project/src/paperclip-data`.

Summary:

| What                | Free tier (default)       | With S3 / paid + disk       |
|---------------------|---------------------------|-----------------------------|
| Attachments / files | Ephemeral (lost on deploy) | S3 = persistent             |
| Run logs / instance | Ephemeral                  | Paid + disk + PAPERCLIP_HOME = persistent |

## Build note

The blueprint builds the UI with `vite build` only (skipping `tsc -b`) to avoid React/TypeScript declaration errors in Render's build environment. Type-checking still runs locally with `pnpm build` or `pnpm typecheck`.

## Checklist after applying the blueprint

1. Set **BETTER_AUTH_SECRET** in the Dashboard (e.g. `openssl rand -base64 32`). That's all required to run.
2. (Optional) Set **PAPERCLIP_PUBLIC_URL** if you use public exposure or a custom domain.
3. (Optional) Configure S3 (see above) for persistent attachments.
4. (Optional, paid only) Add a Persistent Disk and **PAPERCLIP_HOME** for persistent run logs.
