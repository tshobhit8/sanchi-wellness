# One-time setup after this migration merges to main

Two manual steps are needed once — Claude can't do either of these (they require your GitHub login and account access).

## 1. Switch GitHub Pages to build via GitHub Actions

The site now needs a build step (Eleventy), so GitHub Pages can no longer serve `main` directly.

1. Go to `https://github.com/tshobhit8/sanchi-wellness/settings/pages`
2. Under **Build and deployment → Source**, change it from "Deploy from a branch" to **GitHub Actions**.
3. That's it — every push to `main` will now run `.github/workflows/deploy.yml`, which builds the site and deploys it.

## 2. Enable login for the blog admin panel (`/admin`)

The blog editor at `sanchidietitian.in/admin` needs to log you in with GitHub before it'll let you create or publish posts. Since this site isn't hosted on Netlify, that login needs a small free "OAuth proxy" — a few clicks, no code to write.

**Create a GitHub OAuth App:**
1. Go to `https://github.com/settings/developers` → **New OAuth App**.
2. Application name: `Sanchi Wellness CMS` (or anything).
3. Homepage URL: `https://sanchidietitian.in`
4. Authorization callback URL: `https://<your-worker-name>.<your-subdomain>.workers.dev/callback` (you'll fill this in after step 2 below — come back and edit it).
5. Save, then copy the **Client ID** and generate + copy a **Client Secret**.

**Deploy the OAuth proxy (Cloudflare Workers, free tier):**
1. Sign up at `https://dash.cloudflare.com` if you don't have an account.
2. Create a new Worker using the `netlify-cms-oauth-provider` or `sveltia-cms-auth` template (both are free, open-source, and made exactly for this — search either name on GitHub for deploy instructions, they're a few lines of code with a "Deploy to Cloudflare" button).
3. In the Worker's settings, add environment variables `OAUTH_CLIENT_ID` and `OAUTH_CLIENT_SECRET` with the values from the GitHub OAuth App you created above.
4. Copy the Worker's URL (e.g. `https://sanchi-cms-auth.yourname.workers.dev`) and go back to the GitHub OAuth App to set the callback URL to `<worker-url>/callback`.

**Point the CMS at your proxy:**
1. Open `src/admin/config.yml` in this repo.
2. Under `backend:`, add a line: `base_url: https://<your-worker-url>` (no trailing slash).
3. Commit and push — the next deploy will pick it up.

Once both are done, go to `sanchidietitian.in/admin`, click "Login with GitHub," and you'll land in the blog editor: create a post, upload images (they auto-fit into the layout), save as a draft, move it through the review workflow, and publish — it commits straight to this repo.

If this feels like too much right now, the site works fine without it — you can keep asking Claude to add blog posts the way we have been. This just gives you a self-serve option going forward.
