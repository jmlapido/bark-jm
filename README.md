# bark-jm

[Bark](https://github.com/Finb/Bark) push notification server, running on
Cloudflare Workers + D1, deployed at **bark.jmlapido.com**.

Based on [cwxiaos/bark-worker](https://github.com/cwxiaos/bark-worker) (D1 version).

## Prerequisites

- A Cloudflare account with the `jmlapido.com` zone already active in it
  (custom domain routing below depends on this).
- Node.js 18+ and `npm`.
- A Cloudflare API Token with `Workers Scripts:Edit`, `D1:Edit`, and
  `Workers Routes:Edit` permissions (or just log in interactively with
  `wrangler login`).

## One-time setup

```bash
npm install

# Log in to Cloudflare (opens a browser), or export CLOUDFLARE_API_TOKEN instead
npx wrangler login

# Create the D1 database
npm run db:create
```

`db:create` prints a `database_id`. Copy it into `wrangler.jsonc`, replacing
`<unique-ID-for-your-database>` in the `d1_databases` block.

Run the schema migrations against the new database:

```bash
npm run db:migrate:remote
```

## Deploy

```bash
npm run deploy
```

This publishes the worker and (because of the `routes` entry in
`wrangler.jsonc`) attaches it to the custom domain `bark.jmlapido.com`.
Cloudflare will automatically create the DNS record for that hostname in the
`jmlapido.com` zone the first time you deploy — no manual DNS edit needed as
long as the zone lives in the same Cloudflare account.

## Configuration (`wrangler.jsonc` → `vars`)

| Var | Default | Purpose |
|---|---|---|
| `ALLOW_NEW_DEVICE` | `true` | Allow new Bark devices to self-register. Consider setting to `false` once all your devices are registered. |
| `ALLOW_QUERY_NUMS` | `true` | Allow `/info` to report device counts. |
| `ROOT_PATH` | `/` | Path prefix if you don't mount the server at the domain root. |
| `BASIC_AUTH` | unset | `user:password` to protect `/info` and `/mcp`. Recommended — uncomment in `wrangler.jsonc`. |

After changing `vars`, redeploy with `npm run deploy`.

## Using it with the Bark app

In the Bark iOS app, add a custom server: `https://bark.jmlapido.com`. The
app registers itself and gets a device key back; use that key (or the full
push URL it gives you) to send notifications, e.g.:

```bash
curl "https://bark.jmlapido.com/<device_key>/Hello/World"
```

See [Bark-Server API docs](https://github.com/Finb/bark-server) for the full
push/register/ping API surface.

## Local development

```bash
npm run db:migrate   # apply migrations to a local D1 instance
npm run dev           # wrangler dev
```
