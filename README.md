# Blocked email domains sync

**Version: v1.0.0** (see [CHANGELOG.md](./CHANGELOG.md))

A [Discourse Workflows](https://meta.discourse.org/t/discourse-workflows/407100) workflow that keeps your forum’s `blocked_email_domains` setting in sync with the community [disposable-email-domains](https://github.com/disposable-email-domains/disposable-email-domains) list.

Disposable addresses are a common path for spam signups and throwaway accounts. That upstream list is around 10,000 domains — far too large to paste into the Discourse admin UI by hand. Pasting it once also tends to leave hidden carriage returns on every line, so most of the list never actually matches. This workflow rebuilds a clean value every week and only writes when something changed.

The domain list itself is **not** copied into this repository. Each run downloads the current file from GitHub.

## Install

1. Download [`blocked-email-domains-sync.json`](./blocked-email-domains-sync.json) from this repo (or the release attachment).
2. Go to `/admin/plugins/discourse-workflows/workflows` on your forum.
3. Click **+ New workflow**.
4. Click the **⋮** menu at the top right of the canvas.
5. Choose **Import**.
6. Pick `blocked-email-domains-sync.json` and click **Open**.
7. Follow the sticky note on the canvas, then publish when a manual run looks right.

Import adds the nodes to the workflow you have open. It does not create a new workflow by itself, so start from **+ New workflow** first. Credentials, variable *values*, the error workflow, and the timezone are not transferred by Import — set those on the destination site.

## Configuration

| What | Where | Notes |
| --- | --- | --- |
| Extra domains to block | `EXTRAS` at the top of **Build list** | Starts empty. Add hostnames you want on top of upstream. |
| Domains never to block | `EXCLUDE` in **Build list** | Defaults: `simplelogin.com`, `passmail.net` (privacy alias services). |
| Major providers never to block | `SAFE` in **Build list** | Gmail, Outlook, Yahoo, iCloud, Proton, and similar. |
| Chat alert channel | Global variable `alerts_channel` | Used as `$vars.alerts_channel`. Create it under Workflows → Variables with your chat channel id, **or** delete the **Alert: list changed** node. |
| Schedule | Trigger cron `30 4 * * 1` | Runs in the workflow timezone (Settings → Timezone). Change the expression or the timezone so it lands on your Monday morning. |
| Error workflow | Settings → Error workflow | Optional. Point it at a workflow with an error trigger so failed runs alert you. |

The setting the workflow writes is visible under **Admin → Settings → `blocked_email_domains`**.

## How it works

1. **Monday 04:30** (cron) — schedule trigger.
2. **Fetch upstream list** — HTTP GET of `disposable_email_blocklist.conf` from GitHub. No API key.
3. **Build list** — lowercase/trim, drop blanks and comments, add `EXTRAS`, remove `EXCLUDE` and `SAFE`, keep valid hostnames only, de-duplicate, sort, join with `\|`. Compares against the current site setting (and detects leftover whitespace). Throws if upstream has fewer than 5,000 entries or the download looks truncated, so nothing is written.
4. **Changed?** — continues only when the built value differs from what is stored.
5. **Save blocked_email_domains** — Discourse Workflows **Site setting** node, acting as `system`. No admin API key is required.
6. **Verify** — reads the value back and confirms it matches what was built.
7. **Alert: list changed** — optional chat message with the counts.

Measured on a ~10k-domain list: about **330–490 ms** wall time per run, of which about **33–51 ms** is code time (well under the 1000 ms code limit).

## Safety

- Aborts without writing if upstream is missing, truncated, or suspiciously small.
- Never blocks the built-in SAFE major providers, even if upstream lists them.
- Uses the plugin’s site-setting node as `system` — no API key, no credential to import.
- First run can add thousands of domains; run it once by hand and check Executions before publishing.

## Credit

Upstream domain list: [disposable-email-domains/disposable-email-domains](https://github.com/disposable-email-domains/disposable-email-domains). That project has its own licence; this repo does not redistribute the list.

This workflow export is MIT-licensed — see [LICENSE](./LICENSE).
