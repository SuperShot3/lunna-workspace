# OpenClaw + Lunna workspace (operator notes)

This folder is an [OpenClaw agent workspace](https://docs.openclaw.ai/concepts/agent-workspace): the Gateway loads `AGENTS.md`, `SOUL.md`, `USER.md`, and related files into context. Config and secrets live under `~/.openclaw/`, not in this repo.

## Gateway and LINE

- Install the LINE plugin: `openclaw plugins install @openclaw/line` ([LINE channel docs](https://docs.openclaw.ai/channels/line)).
- Configure `channels.line` in `~/.openclaw/openclaw.json` (channel access token, channel secret, webhook HTTPS to `https://<gateway-host>/line/webhook`).
- DM access: default pairing — use `openclaw pairing approve line <CODE>` when testing.
- Point `agent.workspace` (or `agents.defaults.workspace`) at this directory on the machine that runs the Gateway.

## Markdown vs LINE behavior

Workspace `.md` files are **context for the model** (instructions, memory). Customer-facing text on LINE is processed separately: OpenClaw **strips markdown** for LINE and may turn code/tables into Flex cards ([LINE message behavior](https://docs.openclaw.ai/channels/line)). If replies looked broken, simplify formatting in answers (plain sentences, simple bullets, URLs alone) and avoid relying on tables or fenced code blocks in the *reply text*.

## Catalog data (fixed paths)

- Product URLs and slugs: read **`catalog.json`** at the workspace root (not `catalog/products.json` or `content/catalog/...`).
- Regenerate that file from your shop pipeline when the catalog changes; keep `BUSINESS.md` and `AGENTS.md` aligned with the real filename.

## Persona

All Lunna persona, mission, and LINE behavior live in **`SOUL.md`**. There is no separate `BOOTSTRAP.md` (removed after consolidation with OpenClaw’s recommended workspace layout).

## Subagent → main agent

If LINE runs as a **customer** session and you use a **main** session for the operator, **`docs/skills/subagent-to-main-report.md`** defines how the LINE agent **reports** (new threads, stalled/expired/no outcome, user asks for help). It is instruction text only—no shop HTTP route.

## Useful CLI / docs

- Control UI: `openclaw dashboard` (default [http://127.0.0.1:18789/](http://127.0.0.1:18789/)).
- Markdown pipeline overview: [Markdown formatting](https://docs.openclaw.ai/concepts/markdown-formatting).
- Index of all pages: [llms.txt](https://docs.openclaw.ai/llms.txt).
