# Changelog

## v1.0.0 — 2026-11-05

- First public release.
- Weekly schedule (`30 4 * * 1` in the workflow timezone) fetches the community disposable-email-domains list and writes a clean `|`-joined value to the Discourse `blocked_email_domains` site setting when it changed.
- Aborts without writing if upstream looks truncated or suspiciously small.
- Keeps `simplelogin.com` and `passmail.net` unblocked by default; never blocks a built-in SAFE list of major providers.
- Optional chat alert via the global `$vars.alerts_channel` variable.
