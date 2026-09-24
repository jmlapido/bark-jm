---
name: bark-notify
description: Send a push notification to the user's phone via their self-hosted Bark server. Use this whenever a long-running task finishes, something needs the user's attention, an error or alert should be surfaced, or the user explicitly asks to be notified, pinged, or alerted.
---

# Bark Notify

Pushes a notification to the user's iPhone through their own Bark server at
`bark.jmlapido.com`.

## Setup (one time)

Export these in the environment this skill runs in (shell profile, CI secret,
agent runtime config — wherever it applies). Never hardcode them in this file
or commit them anywhere:

- `BARK_SERVER` — e.g. `https://bark.jmlapido.com`
- `BARK_DEVICE_KEY` — the device key from the Bark iOS app (Settings → tap your
  server → key), or generic MCP flow if not using a fixed device
- `BARK_AUTH` — `username:password` for the server's Basic Auth

If any of these are unset, tell the user the notification could not be sent
and why — do not guess values or silently skip.

## Sending a notification

```bash
curl -sS -u "$BARK_AUTH" -X POST "$BARK_SERVER/$BARK_DEVICE_KEY" \
  -H "Content-Type: application/json" \
  -d "$(cat <<JSON
{
  "title": "<short title>",
  "body": "<what happened>",
  "group": "agent"
}
JSON
)"
```

A successful call returns `{"code":200,"message":"success",...}`.

Useful optional fields (add to the JSON body as needed):
- `"level": "critical"` — bypasses silent/focus mode for urgent alerts
  (use sparingly — this rings and vibrates even in Do Not Disturb)
- `"url": "https://..."` — tapping the notification opens this link
- `"sound"`, `"badge"`, `"volume"` — see the Bark app docs for values

## When to use this

- A task the user stepped away from finishes (build, deploy, long scrape/render)
- Something fails and needs their attention (CI red, a monitored service down)
- They explicitly say "notify me", "ping me", "let me know when..."

Don't spam it — one notification per meaningful event, not per step.
