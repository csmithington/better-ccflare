# csmithington/better-ccflare: fork notes

This fork exists so AgentOS can run fixes before they land upstream. It carries no permanent divergence.

## Branches

- `main`: an untouched mirror of `tombii/better-ccflare` `main`. Only ever fast-forwarded. `gh repo sync` is refused when upstream changes workflow files (it needs the `workflow` token scope), so push it over SSH: `git push fork origin/main:refs/heads/main`.
- `agentos/deploy`: `main` plus the open PR branches below, merged with `--no-ff`. AgentOS builds and runs this branch.
- One branch per upstream PR (`fix/...`, `feat/...`). Delete it once upstream merges the PR.

## Pending upstream

| Branch | Upstream PR | Why AgentOS needs it |
| --- | --- | --- |
| `fix/output-config-effort-openai` | tombii/better-ccflare#573 | Claude Code sends `output_config.effort`; without this, GPT slots behind CLIProxyAPI never receive the effort. |

Merged upstream earlier: #559 (reasoning-effort passthrough and served-model headers).

## Updating AgentOS

In `/home/ubuntu/.local/share/better-ccflare/app` on AgentOS:

1. `git fetch origin && git fetch fork`, then fast-forward the fork's `main` (above).
2. Rebuild `agentos/deploy`: branch it from `origin/main` and `git merge --no-ff` each pending branch that upstream has not merged yet. If none are left, `agentos/deploy` equals `main`.
3. Back up `~/.config/better-ccflare/better-ccflare.db` and `apps/cli/dist/better-ccflare`.
4. Build with the pinned bun (`packageManager`, currently 1.4.4): `bun install --frozen-lockfile && bun run build`.
5. `systemctl --user restart better-ccflare`, then check `journalctl --user -u better-ccflare` and one request.
6. Push the result: `git push fork local/deploy:refs/heads/agentos/deploy`.

Rollback: restore the backed-up binary and restart. The database migrations upstream adds are additive.
