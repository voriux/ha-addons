# Claude Remote

Runs Claude Code as a **Remote Control server** next to Home Assistant. Open claude.ai/code or the Claude mobile app, pick this machine, and work with a Claude that can read live HA state and open PRs against your config repository.

Needs a Claude subscription (Pro, Max, Team or Enterprise). API keys and `claude setup-token` tokens cannot run Remote Control.

## How it works

- `claude remote-control` runs in `/data/workspace/<first repository>`, a clone of your config repo. It connects out to Anthropic; no port is opened.
- `/config` is mounted **read-only** at `/homeassistant`. Config changes go through branches and pull requests, never straight into the live config.
- Live state comes from the repository's `.mcp.json` (Home Assistant MCP server), using `ha_token`.
- Permission prompts can trigger an HA webhook (`webhook_id`), for example an Alexa announcement.
- The **Claude** panel is a terminal (ttyd) for login and debugging. Only the ingress proxy can reach it.

## Setup

1. **HA user and token.** Create a non-admin user `claude`, log in as it, and create a long-lived token (Profile → Security). Expose to Assist only the entities Claude may control. Put the token in `ha_token`.
2. **GitHub token.** Create a fine-grained token for the repositories in `repositories` only: Contents read/write, Pull requests read/write, Metadata read. Protect `main` with "require a pull request". Put the token in `github_token`.
3. **Webhook (optional).** Put the webhook id of your "Claude needs input" automation in `webhook_id`.
4. Start the app, open the **Claude** panel and run:
   ```
   claude-setup
   ```
   - Step 1: `claude auth login` prints a URL. Open it on your phone or laptop, sign in, paste the code back.
   - Step 2: answer `y` to "Trust this folder?" and "Enable Remote Control?". When the session shows as ready, press Ctrl-C.
5. Within a minute the server starts (see the app log). Open claude.ai/code or the Claude app: the machine shows under the name in `session_name`.

## Options

| Option | Default | Meaning |
|---|---|---|
| `session_name` | `ha` | Name shown in claude.ai/code |
| `spawn` | `worktree` | `worktree`: each session gets its own git worktree. `same-dir`: all share the clone. `session`: one session, then exit |
| `permission_mode` | `default` | Permission mode for spawned sessions. `bypassPermissions` is not offered |
| `capacity` | `3` | Max concurrent sessions. Each needs 200–400 MB RAM |
| `repositories` | `voriux/ha-control` | `owner/name` list, cloned into `/data/workspace/`. The first is the working folder |
| `ha_token` | | Long-lived token of the non-admin `claude` user |
| `github_token` | | Fine-grained GitHub token for the repositories above |
| `webhook_id` | | HA webhook called by the Notification hook |

## Login expiry

Claude logins expire. Every 10 minutes the app checks `claude auth status`. When logged out it creates a persistent notification in HA. Run `claude-setup` in the panel again; the server restarts by itself.

## Managed files

Rewritten on every start, so edit the options, not these files:
- `/data/.claude/settings.json`: deny rules (no push to `main`, no force push, no reading `secrets.yaml`, options or tokens), MCP approval, hooks.
- `/run/claude/env` (tmpfs): tokens and options for the services and the terminal.

Existing clones are only fetched on start, never reset, so work in progress survives restarts.

## Security notes

- Anyone who controls your Claude account can run commands in this container. The container has no host network, no Docker socket, no Supervisor API and a read-only `/config`.
- The deny rules stop Claude's Read tool, not shell commands. In `default` mode every shell command asks first.
- The login file is excluded from HA backups. After a restore, run `claude-setup` again.
- Requirements: 4 GB RAM or more on the host.
