# This fork — Eddington agent context

This is Mark's personal fork of [Softeria/ms-365-mcp-server](https://github.com/Softeria/ms-365-mcp-server),
deployed to give AI agents live access to his personal Microsoft Outlook account (mail,
calendar, contacts) as part of his home-lab project, "Eddington." `upstream` is the
original project; `origin` (this fork) is where the Eddington-specific work happens.

This file is new — everything else in the repo is upstream's, including `README.md`,
which is left alone so `git fetch upstream && git merge upstream/main` stays conflict-free.

## Read this first

The full decision record for this project lives in the sibling repo `eddington-shared`
(private, checked out alongside this one — `../eddington-shared` if you're on the same
machine as Mark's other Eddington work):

- `notes/2026-09-23-outlook-agent-access.md` — why this candidate was picked, the scope
  decisions, the candidates that were ruled out, and the open questions below.
- `TODO.md` — the "MCP & agents" section tracks this as an open item across the whole
  project.
- `mcp/servers.md` — the hosting/transport pattern already used for this project's other
  self-hosted MCP servers (`ha-mcp`, `unifi-network-mcp`, `synology-mcp`, all Docker
  containers on a home server). This server should follow the same pattern once deployed;
  it isn't listed there yet because it isn't deployed yet.
- `README.md`'s "Local vs External" section — this server itself is **Local** (runs on
  Mark's own hardware) even though Outlook is an **External** third-party service it
  talks to. That distinction drives where things are allowed to live.

## Hard rule: keep this fork generic

Nothing committed here should include real LAN IPs/hostnames, Mark's personal account
identifiers, or any secret/token. That detail belongs in `eddington-shared` (private) or
in local, gitignored `.env` files, referenced by name only — same convention the other
self-hosted MCP servers already follow. This fork is public on GitHub (forks of a public
repo can't be made private), so treat everything committed here as world-readable.

## Status: decisions already made

From the shared note (don't duplicate the reasoning here, just the headline):

- Personal Outlook.com/Hotmail account only — not work/365, to avoid tenant/admin-consent
  complications.
- Full scope: mail (read, search, draft, send, delete/trash), calendar, and contacts.
- Standalone Docker container on the home server, matching `ha-mcp`/`unifi-network-mcp`.
- Must be agent-agnostic (Claude Code/Desktop, local models via Open WebUI, etc.), not
  Claude-specific. This project was already the leading candidate specifically because it
  ships both stdio *and* HTTP transport.

## Next steps for whoever picks this up

1. **Auth**: register an Entra app for the "personal Microsoft accounts" audience with
   delegated permissions (`Mail.ReadWrite`, `Mail.Send`, `Calendars.ReadWrite`,
   `Contacts.ReadWrite`, `offline_access`). Confirm the device-code flow works headless on
   a home server, and pick where the refresh token gets stored in a container (check
   whether this project's token cache has a file-based fallback to the OS keyring default).
2. **Guardrails**: with send and delete in scope, decide whether the server itself should
   require confirmation for outward-facing/destructive tools, or leave that to each agent's
   own permission model. Since this must stay agent-agnostic, enforcing it server-side is
   the safer default — e.g. delete = move to Deleted Items (never permanent), send gated
   behind an explicit confirm parameter or a draft-then-send flow.
3. **Transport**: verify HTTP mode works LAN-only (no public reverse proxy) — check
   whether the documented reverse-proxy + OAuth setup for HTTP mode can be simplified for
   a LAN-only deployment.
4. **Deploy**: adapt the existing `Dockerfile` for the home server, following the
   `docker-compose.yaml` + `/srv/shared/<name>` layout the other MCP servers use (see
   `eddington-shared/mcp/servers.md` for the exact pattern).
5. **Once live**: add the connection details to `eddington-shared/mcp/servers.md` (same
   format as the existing entries) and check off the TODO item in `eddington-shared/TODO.md`.
