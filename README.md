# marcusvanesveld.com

Static site for GitHub Pages. **Nothing here is published.** It is built and waiting.

## Why it exists

Two jobs, one host:

1. **The front door.** A name, what he does, and what he has made. The consultancy angle needs a
   real address more than it needs a brochure.
2. **The update feed.** ViddyGrid's Sparkle updater points at a URL. Until that URL exists and
   serves HTTPS, every copy shipped can never be updated. Measured 2026-08-06: `marcusvanesveld.com`
   answers on port 80 with a 404 and **has no listener on 443 at all**. GitHub Pages issues a
   certificate for a custom domain, which fixes address and TLS together, free.

## To publish — Marcus's call, every step

1. Create a **public** repo `marcusvanesveld/apps` on the `marcusvanesveld` account. Public is
   required for free Pages; **no source goes in it**, only this site and release binaries.
2. Push this directory to it.
3. Repo Settings > Pages > deploy from `main` / root. Add `marcusvanesveld.com` as the custom
   domain and tick Enforce HTTPS.
4. At Mijndomein, point the DNS at GitHub: four A records
   (`185.199.108.153`, `.109.153`, `.110.153`, `.111.153`) plus a `www` CNAME to
   `marcusvanesveld.github.io`. **Do not start the Porkbun transfer before ~18 September 2026** —
   transferring within 45 days of the 4 August renewal can void the year.
5. Attach `ViddyGrid-1.0.dmg` to a GitHub Release and put its URL in `appcast.xml`.

## Before the update feed actually works

Three things, and only the third is automatic:

1. **A Sparkle signing key.** None exists — `security find-generic-password -s
   'https://sparkle-project.org'` returns nothing, and `SUPublicEDKey` is empty in `project.yml`
   and in the shipped bundle. Generating it is Marcus's call. `Sparkle/bin/generate_keys` makes it.
2. **ViddyGrid repointed.** `project.yml` still says `https://viddygrid.app/appcast.xml`, a domain
   that does not resolve. It should say `https://marcusvanesveld.com/appcast.xml`.
3. **The updater started.** `ViddyGrid.swift` builds `SPUStandardUpdaterController(startingUpdater:
   false)` and never starts it, so no check fires today. Flip it, rebuild, re-notarize (~7 min).
