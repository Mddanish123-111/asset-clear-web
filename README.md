# AssetClear — Web (standalone deploy repo)

This is a **standalone copy** of the `apps/web` Next.js frontend, pulled out
of the main AssetClear monorepo specifically for Netlify deployment.

## Why a separate repo?

Netlify's Next.js Runtime has a well-documented, still-unresolved bug with
monorepos (confirmed across pnpm/yarn/npm, Turborepo, Nx setups — see
Netlify's own support forum, e.g. the "Cannot find module
'next/dist/server/lib/start-server.js'" thread). No combination of `base` /
`package directory` / `netlify.toml` placement reliably fixes it. The one
consistently working fix reported by multiple people: deploy the Next.js app
from a repo where it **is** the root, not nested inside a monorepo.

Your API, database schema, and full source of truth still live in the main
`assetclear` monorepo — this repo exists only so Netlify has a clean,
un-nested Next.js project to build.

## Deploying this repo

1. Create a new, empty GitHub repository (e.g. `assetclear-web`).
2. Push this folder's contents to it:
   ```bash
   cd assetclear-web-standalone
   git init
   git add .
   git commit -m "Standalone frontend for Netlify"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/assetclear-web.git
   git push -u origin main
   ```
3. In Netlify: **Add new site → Import an existing project**, point it at
   `assetclear-web`. Leave every build setting on Netlify's auto-detected
   defaults — don't set Base directory or Package directory, don't add the
   `@netlify/plugin-nextjs` plugin manually.
4. Set the one environment variable: **Site settings → Environment
   variables**:
   ```
   NEXT_PUBLIC_API_URL = https://assetclear-api.onrender.com/api
   ```
5. Deploy.

## Keeping this in sync with the main monorepo

Whenever you change anything under `apps/web` in the main `assetclear`
repo, copy those changes here and push again:

```bash
# from the main assetclear repo
rsync -av --exclude node_modules --exclude .next apps/web/ ../assetclear-web-standalone/
cd ../assetclear-web-standalone
git add .
git commit -m "Sync from monorepo"
git push
```

This is a manual step for now. If this becomes a recurring pain point, the
next-best option is a GitHub Actions workflow that does this sync
automatically on every push to `apps/web` in the main repo (ask me to set
that up when you're ready).
