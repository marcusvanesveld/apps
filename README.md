# marcusvanesveld.com

Static site for GitHub Pages. **Nothing here is published.** It is built and waiting.

## Why it exists

Three jobs, one host:

1. **The front door.** A name, what he does, and what he has made. The consultancy angle needs a
   real address more than it needs a brochure.
2. **The update feed.** ViddyGrid's Sparkle updater points at a URL. Until that URL exists and
   serves HTTPS, every copy shipped can never be updated. Measured 2026-08-06: `marcusvanesveld.com`
   answers on port 80 with a 404 and **has no listener on 443 at all**. GitHub Pages issues a
   certificate for a custom domain, which fixes address and TLS together, free.
3. **The two URLs the App Store submission cannot proceed without.** Added 2026-08-07. Apple
   requires a **privacy policy URL** and a **support URL** for every app, both mandatory, and both
   have to actually resolve — a reviewer opens them. They do not need a site of their own; any page
   on any site he controls satisfies it. So they live here:

   ```
   viddygrid/privacy/index.html   ->  https://marcusvanesveld.com/viddygrid/privacy
   viddygrid/support/index.html   ->  https://marcusvanesveld.com/viddygrid/support
   ```

   **This makes publishing this site a blocker on the App Store submission**, which it was not
   before. The App Store Connect app record cannot be filled in honestly until these two URLs are
   live. See `metadata/app-record.md` §9 in the ViddyGrid repo.

   Directories with `index.html` rather than `privacy.html`, deliberately: plain HTTP semantics that
   work on any static host, not only on Pages. Both `/viddygrid/privacy` and
   `/viddygrid/privacy/` work — the first 301s to the second. Verified locally, 200 both pages, no
   external resources on either.

   **`viddygrid.app` is not registered and does not need to be.** Checked 2026-08-07: RDAP 404 from
   Google's `.app` registry, no DNS delegation. An App Store app needs a URL, not a domain. Do not
   buy it for this.

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
6. **Check the two App Store URLs resolve over HTTPS** before filling in the app record:
   `https://marcusvanesveld.com/viddygrid/privacy` and `.../viddygrid/support`. Pages can take a few
   minutes to issue the certificate after the custom domain is set; Apple will reject a support URL
   that does not answer.

**The Porkbun transfer is a separate matter and is not on this critical path.** The site publishes
fine on the current Mijndomein nameservers; step 4 above is a DNS record change, not a transfer.
Nothing about the App Store submission requires touching the registrar before ~18 September.

## Before the update feed actually works

Three things, and only the third is automatic:

1. **A Sparkle signing key.** None exists — `security find-generic-password -s
   'https://sparkle-project.org'` returns nothing, and `SUPublicEDKey` is empty in `project.yml`
   and in the shipped bundle. Generating it is Marcus's call. `Sparkle/bin/generate_keys` makes it.
2. **ViddyGrid repointed.** `project.yml` still says `https://viddygrid.app/appcast.xml`, a domain
   that does not resolve. It should say `https://marcusvanesveld.com/appcast.xml`.
3. **The updater started.** `ViddyGrid.swift` builds `SPUStandardUpdaterController(startingUpdater:
   false)` and never starts it, so no check fires today. Flip it, rebuild, re-notarize (~7 min).
