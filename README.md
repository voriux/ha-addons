# voriux Home Assistant apps

Add this repository in Home Assistant: Settings → Apps → App store → ⋮ → Repositories → `https://github.com/voriux/ha-addons`.

| App | What it does |
|---|---|
| [Claude Remote](claude-remote/DOCS.md) | Claude Code Remote Control server next to Home Assistant |

Local build and smoke test (no Supervisor needed; options are read from `/data/options.json`):

```bash
docker build -t claude-remote:dev claude-remote
docker volume create cr-test-data
echo '{"session_name":"ha","spawn":"worktree","permission_mode":"default","capacity":3,"repositories":["voriux/ha-control"]}' \
  | docker run --rm -i -v cr-test-data:/data --entrypoint sh claude-remote:dev -c 'cat > /data/options.json'
docker run -d --name cr-test -v cr-test-data:/data claude-remote:dev
docker logs cr-test
```
