# marcusvanesveld.com

Static site on GitHub Pages. **Published 2026-08-08** — repo
`marcusvanesveld/apps` (public), Pages deploying from `main` / root, custom
domain read from the `CNAME` file, DNS pointed at GitHub. All three pages
answer 200. The publish checklist further down is kept as the record of how,
not as work outstanding.

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

## How it was published — 2026-08-08, all of it by agent

Marcus clicked nothing. Recorded so the next domain takes minutes.

1. ~~Create a public repo.~~ **Done.** `gh repo create marcusvanesveld/apps --public`.
   Public is required for free Pages; **no source goes in it**, only this site and
   release binaries.
2. ~~Push.~~ **Done.** One trap: the first push authenticated as the *other*
   GitHub account (`Marcuse-69`) from a stale keychain entry and 403'd. Forcing
   the right account needs the helper chain **reset** first, because a bare
   `-c credential.helper=...` only appends:
   ```
   git -c credential.helper= -c credential.helper='!gh auth git-credential' push -u origin main
   ```
   Both `-c` flags last one process and persist nothing — verified afterwards
   that `git config --global --get-regexp credential` is still empty, per the
   global no-ambient-auth rule.
3. ~~Pages.~~ **Done via API**, no dashboard:
   ```
   gh api -X POST repos/marcusvanesveld/apps/pages -f "source[branch]=main" -f "source[path]=/"
   gh api -X PUT  repos/marcusvanesveld/apps/pages -F "https_enforced=true"
   ```
   The custom domain needed no setting — Pages read it from the `CNAME` file.
   Two notes: `https_enforced` must be `-F` (boolean), not `-f` (string, 422);
   and it 404s with *"The certificate does not exist yet"* until Let's Encrypt
   has issued against the new DNS, so it wants a retry loop, not a one-shot.
4. ~~DNS at Mijndomein.~~ **Done, scripted, including 2FA.** Four A records
   (`185.199.108.153`, `.109.153`, `.110.153`, `.111.153`) on the root plus a
   `www` CNAME to `marcusvanesveld.github.io.`, and the four Mijndomein parking
   records (2×A `213.249.67.10`, 2×AAAA `2a01:448:2001::10`) deleted. SPF, DMARC,
   NS and SOA untouched.

   **Do not hand this to Marcus as a click list.** The whole path is automated in
   `~/Code/active/chrome-extension-development/scripts/mijndomein-dns.sh` —
   cmux browser for the session, keychain for the password, `gog` for the emailed
   code, and the page's own `dnseditor` object for the records. The reasoning,
   and the four traps that cost an hour, are in that repo's `OPERATIONS.md` under
   *"Driving a logged-in web console"*.

   **The Porkbun transfer is still ~18 September 2026** — transferring within 45
   days of the 4 August renewal can void the added year. Unrelated to any of the
   above; the verhuiscode is already in his Gmail.
5. Attach `ViddyGrid-1.0.dmg` to a GitHub Release and put its URL in `appcast.xml`.
   **Still outstanding.**
6. ~~Check the App Store URLs.~~ Both answer:
   `marcusvanesveld.com/viddygrid/privacy` and `.../viddygrid/support`. Apple
   rejects a support URL that does not resolve, so re-check over **HTTPS** once
   the certificate has issued — that is the last gate on the app record.

**The Porkbun transfer is a separate matter and is not on this critical path.** The site publishes
fine on the current Mijndomein nameservers; step 4 above is a DNS record change, not a transfer.
Nothing about the App Store submission requires touching the registrar before ~18 September.
Proven, not assumed: the site went live on 2026-08-08 with the nameservers still at Mijndomein.

## Where the related law lives

Three repos, none restating another:

- `~/Code/active/chrome-extension-development/OPERATIONS.md` — **how to drive a
  logged-in web console** (cmux browser + keychain + `gog` 2FA), and the
  `mijndomein-dns.sh` script. Owns that law.
- `~/Code/active/apple-shipping/SHIPPING.md` — the money and store side, and why
  this domain matters to it: privacy/support URLs for App Store records, and the
  host for notarized DMGs sold through Polar.
- This file — what is on the site and how it got published.

## Before the update feed actually works

Three things, and only the third is automatic:

1. **A Sparkle signing key.** None exists — `security find-generic-password -s
   'https://sparkle-project.org'` returns nothing, and `SUPublicEDKey` is empty in `project.yml`
   and in the shipped bundle. Generating it is Marcus's call. `Sparkle/bin/generate_keys` makes it.
2. **ViddyGrid repointed.** `project.yml` still says `https://viddygrid.app/appcast.xml`, a domain
   that does not resolve. It should say `https://marcusvanesveld.com/appcast.xml`.
3. **The updater started.** `ViddyGrid.swift` builds `SPUStandardUpdaterController(startingUpdater:
   false)` and never starts it, so no check fires today. Flip it, rebuild, re-notarize (~7 min).
