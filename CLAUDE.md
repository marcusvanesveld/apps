# marcusvanesveld.com — rules for agents

## NEVER DRIVE A BROWSER TO LOG IN. READ THE SESSION MARCUS ALREADY HAS.

**MARCUS IS ALREADY LOGGED IN. IN HIS REAL CHROME. RIGHT NOW.** Every vendor
console — registrar, dashboard, portal, store — has a live session sitting in
`~/Library/Application Support/Google/Chrome/<Profile>/Cookies`, encrypted with
a key in the macOS Keychain. **READ IT. REPLAY IT OVER PLAIN HTTP.**

**NEVER type his password. NEVER pull a 2FA code out of his mail. NEVER open or
drive a browser to log in. NEVER hand him a click list for a web form.**

Proven on Mijndomein 2026-08-08: an entire session was burned driving cmux,
typing the password, fetching the emailed code and fighting a 20-minute expiry —
while his Chrome held a live `MDHAuth_prod` JWT and a valid `PHPSESSID` the
whole time. Plain `curl` with those cookies returned the authenticated page,
**200, no login, no 2FA, no browser.**

```bash
# 1. Key from the Keychain
security find-generic-password -w -s "Chrome Safe Storage" -a "Chrome"
# 2. PBKDF2-SHA1(pw, "saltysalt", 1003, 16) -> AES-128-CBC, IV = 16 spaces
# 3. Strip the "v10" prefix, strip PKCS7 padding, THEN STRIP 32 BYTES
#    (macOS Chrome prefixes a SHA-256 of the domain; forget it and every
#     value is garbage that looks like a bad Keychain password)
```

**THE TRAP THAT WILL COST YOU AN HOUR: SCOPE COOKIES PER HOST, LIKE A BROWSER
DOES.** Several hosts under one domain each set their own `PHPSESSID`. Send them
all in one header and the server takes the first — and you bounce to the login
wall looking *exactly* like an expired session. Host cookies beat domain cookies
on a name collision. Send only what that host would receive.

Only if there is genuinely no session: use the cmux browser, never his Chrome
(AppleScript is isolated-world, `-1723`; CDP is blocked on the default profile).
Full playbook — including `eval` taking one line, and never snapshotting a live
modal — is in `chrome-extension-development/OPERATIONS.md`, section
*"Driving a logged-in web console"*. Reference implementation of the pattern:
`quilter-cli-v2/docs/AUTH.md` (`auth from-chrome`), whose rule is the same one
in its own words: **"NEVER launch a real Chrome for auth, ever."**


## What this repo is

The static site behind `marcusvanesveld.com`, served from GitHub Pages out of
the public repo `marcusvanesveld/apps`. Published 2026-08-08. It carries the
front door, the Sparkle `appcast.xml`, and the ViddyGrid privacy + support pages
the App Store submission requires.

The DNS that points the domain here was set by an agent through the registrar's
web console using exactly the pattern in the banner above — see `README.md` for
the record of how, and `chrome-extension-development/scripts/mijndomein-dns.sh`
for the script.

**Do not start the Porkbun registrar transfer before ~18 September 2026** —
transferring within 45 days of the 4 August renewal can void the added year.
That is a registrar move and has nothing to do with DNS records, which are
editable today.
