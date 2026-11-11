# Deploying tv-minis to production

The Vercel project (`somil9/tv-minis`) is **not connected to this Git repo**. Pushing to `main` does not deploy anything. Production is updated by a manual prebuilt upload from a local checkout.

## Recipe

```bash
# from the repo root, with a Vercel auth token (project-scoped is enough)
npx vercel pull --yes --environment=production --token=$VERCEL_TOKEN
npx vercel build --prod --token=$VERCEL_TOKEN
rm -f .vercel/output/static/index.html
npx vercel deploy --prebuilt --prod --token=$VERCEL_TOKEN
```

## Why the `rm` step

Subdomain routing (`breakingbad.somiljain.com` -> `/01-desert-rv/`, etc.) lives in `vercel.json` rewrites, which Vercel evaluates **after** the static filesystem. If a root `index.html` ships in the build output, every subdomain serves the landing page instead of its scene. Previous production deployments shipped no root `index.html`; keep it that way until the landing page gets its own hostname rewrite.

## Why `--prebuilt`

Plain `vercel deploy --prod` uploads the raw directory and does not compile `vercel.json` rewrites into the build output, so the host-based rewrites silently stop working. `vercel build` + `vercel deploy --prebuilt` produces `.vercel/output/config.json` with the routes, matching how production has always been deployed.

## Tokens

Use a short-lived token scoped to the `tv-minis` project (Vercel dashboard -> Account Settings -> Tokens), pass it with `--token`, and delete it in the dashboard when the deploy is verified. Never commit tokens.
