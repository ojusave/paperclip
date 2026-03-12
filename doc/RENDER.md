# Deploying Paperclip on Render

Deploy Paperclip to [Render](https://render.com) using the one-click Blueprint. No code changes required. The Blueprint uses a **paid** (starter) web service and a [Render Persistent Disk](https://render.com/docs/disks) so run logs and instance data persist across redeploys.

---

## Deploy with the Render Blueprint

**[Deploy to Render](https://dashboard.render.com/blueprint/new?repo=https://github.com/paperclipai/paperclip)**

Click the link (or use your fork’s URL). Render will read `render.yaml`, create the resources below, and prompt you to set **BETTER_AUTH_SECRET**. Use the same link to **update** an existing Blueprint: open it, then Apply to sync with the latest `render.yaml`.

---

## What the Blueprint creates

| Resource        | Type        | Purpose |
|----------------|-------------|--------|
| **paperclip-db** | PostgreSQL  | Database. Free tier (1 GB). |
| **paperclip**    | Web Service | Node.js app: API + React UI. **Starter** (paid) plan with a [Persistent Disk](https://render.com/docs/disks) at `/opt/render/project/src/paperclip-data`. **PAPERCLIP_HOME** is set so run logs and instance data persist. |

- **Build:** Installs dependencies with pnpm and builds the UI with Vite (server is not pre-compiled; see build note below).
- **Start:** Runs the server with `tsx` (transpiles on the fly). The app listens on `PORT` and serves both the API and the static UI.
- **Database:** The web service gets `DATABASE_URL` from the linked Postgres instance; migrations run on startup when `PAPERCLIP_MIGRATION_AUTO_APPLY` is set.

No workers, cron jobs, or Redis—just one web service, one Postgres database, and one persistent disk on the service.

---

## How to use it

### 1. Open the Blueprint

Go to **[Deploy to Render](https://dashboard.render.com/blueprint/new?repo=https://github.com/paperclipai/paperclip)** (or `https://dashboard.render.com/blueprint/new?repo=https://github.com/YOUR_USERNAME/paperclip` if using your fork).

### 2. Connect the repo

If prompted, sign in with GitHub and allow Render to access the repository. Choose the repo (and branch, if you use a fork). Render will parse `render.yaml` and show the resources to be created.

### 3. Apply the Blueprint

Click **Apply** (or **Create**). Render will create the **paperclip-db** database and the **paperclip** web service, then start the first deploy.

### 4. Set required environment variables

The Blueprint marks **BETTER_AUTH_SECRET** as a secret you must set. In the Render Dashboard:

1. Open your **paperclip** web service.
2. Go to **Environment** (or **Environment Variables**).
3. Add **BETTER_AUTH_SECRET** with a long random value (e.g. run `openssl rand -base64 32` locally and paste the result).
4. Save. Render will redeploy if needed.

That’s the only required secret for the default (private) deployment.

### 5. Open the app

When the deploy succeeds, the service URL (e.g. `https://paperclip-xxxx.onrender.com`) will be live. Open it in a browser to use the Paperclip UI. Log in or sign up as needed (auth uses the request host; no need to set `PAPERCLIP_PUBLIC_URL` for private deployment).

---

## Environment variables (reference)

The Blueprint sets these automatically:

- **NODE_ENV**, **HOST**, **PORT**, **SERVE_UI**
- **DATABASE_URL** (from the **paperclip-db** instance)
- **PAPERCLIP_DEPLOYMENT_MODE**, **PAPERCLIP_DEPLOYMENT_EXPOSURE**, **PAPERCLIP_MIGRATION_AUTO_APPLY**
- **PAPERCLIP_HOME** (set to the disk mount path so run logs and instance data persist)

You must set in the Dashboard:

- **BETTER_AUTH_SECRET** — Required. Use a long random string (e.g. `openssl rand -base64 32`).

Optional (set in the Dashboard if you need them):

- **PAPERCLIP_PUBLIC_URL** — Only if you use public exposure or a custom domain; for private deployment the app uses the request host.
- **PAPERCLIP_STORAGE_PROVIDER** = `s3`, plus **PAPERCLIP_STORAGE_S3_BUCKET**, **AWS_ACCESS_KEY_ID**, **AWS_SECRET_ACCESS_KEY** (or equivalent for R2/B2) — For persistent file storage (see below).

---

## Storage and persistence on Render

The Blueprint attaches a [Render Persistent Disk](https://render.com/docs/disks) to the **paperclip** service (mount path `/opt/render/project/src/paperclip-data`) and sets **PAPERCLIP_HOME** so run logs and instance data persist across redeploys. File storage defaults to **local_disk**, so attachments are stored on that same disk and also persist.

### Optional: S3 for attachments

To store attachments in S3 (or an S3-compatible store) instead of the local disk, set in the Dashboard:

- **PAPERCLIP_STORAGE_PROVIDER** = `s3`
- **PAPERCLIP_STORAGE_S3_BUCKET** — your bucket name
- **AWS_ACCESS_KEY_ID** / **AWS_SECRET_ACCESS_KEY** (or equivalent for Cloudflare R2, Backblaze B2, etc.)

Optional: **PAPERCLIP_STORAGE_S3_REGION**, **PAPERCLIP_STORAGE_S3_ENDPOINT** for custom endpoints.

| What                | With Blueprint (default)    | With S3                      |
|---------------------|----------------------------|------------------------------|
| Attachments / files  | Persistent (on disk)       | Persistent (S3)              |
| Run logs / instance  | Persistent (PAPERCLIP_HOME on disk) | Persistent (same disk) |

---

## Build note

The Blueprint builds only the **UI** (Vite); the server is run with **tsx** so it is not type-checked or pre-compiled on Render. This avoids upstream type errors in the server build without any code changes. Full type-checking and `node server/dist/index.js` remain for local development. If you change the build or start command in `render.yaml` and the service was created earlier, re-apply the Blueprint or update the service Render settings to match.

---

## Summary checklist

1. Open **[Deploy to Render](https://dashboard.render.com/blueprint/new?repo=https://github.com/paperclipai/paperclip)** and connect the repo.
2. Apply the Blueprint to create **paperclip-db**, **paperclip** (with persistent disk), and set **PAPERCLIP_HOME**.
3. In the **paperclip** service, set **BETTER_AUTH_SECRET** in Environment.
4. (Optional) Set **PAPERCLIP_PUBLIC_URL** for public or custom domain.
5. (Optional) Configure S3 (or compatible) for attachment storage.
