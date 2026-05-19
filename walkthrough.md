# Deployment Walkthrough: OpenWA in the Cloud (Railway)

> [!NOTE]
> **Update:** We have successfully resolved the Railway build failure! The root `Dockerfile` was updated to copy `dashboard/package*.json` into the builder stage prior to running `npm ci`. This ensures all dashboard dependencies are fully installed, allowing the build step (`nest build && npm run dashboard:build`) to compile flawlessly in the cloud environment. Your new git push has already triggered a rebuild on Railway!

We have successfully modified the **OpenWA codebase** to enable a **single-container deployment**! 

Previously, OpenWA was configured to run as multiple Docker containers coordinated by `docker-compose` and routed via a Traefik reverse proxy. While this is great for local development, it makes hosting on platforms like Railway or Render complex, requiring multiple paid services and complicated networking.

To simplify this, we have consolidated everything into a single, high-performance container:
1. **Express Static Serving in NestJS:** Modified `src/main.ts` to automatically detect and serve compiled static dashboard assets from `dashboard/dist` whenever a user opens the app in their browser.
2. **SPA Router Fallback:** Added a catch-all route wildcard handler inside NestJS so that direct refreshes or client-side React navigation links (e.g., visiting `/sessions` directly) do not trigger 404 errors.
3. **Unified Build Pipeline:** Updated `package.json`'s `build` script to compile both the NestJS backend and the Vite frontend dashboard with a single command.
4. **Optimized Dockerfile:** Updated the production Dockerfile stage to copy the built dashboard assets directly into the final image.

---

## Step 1: Push Your Code to GitHub (Done! ✅)

I have already successfully linked your local repository to your GitHub repository and pushed all the custom changes! 
Your code is fully live and up-to-date at: [github.com/My-Dev-sketch/openwa-cloud](https://github.com/My-Dev-sketch/openwa-cloud)

---

## Step 2: Deploy to Railway

Now that your code is on GitHub, deploying it to Railway takes less than 2 minutes:

1. Visit [Railway.app](https://railway.app) and sign in using your **GitHub account**.
2. Click **New Project** (or **+ New** in the top right).
3. Select **Deploy from GitHub repo** and select your newly created `openwa-cloud` repository.
4. Click **Deploy Now**. 

---

## Step 3: Attach Persistent Storage (Crucial!)

To ensure your WhatsApp login persists forever even if Railway restarts or rebuilds your container, we must attach a **Volume** (persistent hard disk):

1. In your Railway project canvas, click on your **`openwa-cloud` service box**.
2. Go to the **Settings** tab.
3. Scroll down to the **Volumes** section and click **+ Add Volume**.
4. Configure the volume:
   - **Mount Path:** `/app/data` (This matches our container's session directory!)
5. Save the settings. Railway will automatically rebuild and redeploy your service with the persistent disk active.

---

## Step 4: Add Environment Variables

In the same service screen in Railway, go to the **Variables** tab and click **New Variable** to add the following options:

| Variable Name | Value | Description |
|:---|:---|:---|
| `PORT` | `2785` | Instructs the NestJS server to bind to the port exposed in the Dockerfile. |
| `NODE_ENV` | `production` | Enables production security and optimizations. |
| `PUPPETEER_HEADLESS` | `true` | Runs Chromium headlessly in the cloud container. |
| `PUPPETEER_ARGS` | `--no-sandbox,--disable-setuid-sandbox,--disable-dev-shm-usage` | Disables security sandboxes that usually break in container environments. |

Railway will automatically run a fresh deployment whenever these variables are updated.

---

## Step 5: Generate a Public Domain & Scan!

1. Go to the **Settings** tab of your `openwa-cloud` service.
2. Scroll to the **Networking** section and click **Generate Domain** (or set a custom domain). This will generate a public URL like `https://openwa-production.up.railway.app`.
3. Open this public URL in your browser.
4. You will see the beautiful **OpenWA React Dashboard** load instantly!
5. Scan the QR code using your phone's WhatsApp Link Device feature.

> [!TIP]
> **Verification Test:** To verify that persistence works, wait for your session to connect successfully, then click **Restart** on your Railway service dashboard. Once it boots back up, refresh the page. You will see that you are still **connected** and do not need to re-scan the QR code!
