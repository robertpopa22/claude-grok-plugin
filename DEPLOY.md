# DEPLOY — applying plugin edits

> **Why this file exists:** Claude Code does **not** run this plugin from the repo. At install time it
> copies the plugin files into a versioned cache dir. Editing the repo has **no effect** until the cache
> is re-synced **and** Claude Code is restarted.

## How it loads (this machine)

- Marketplace `geseidl-grok` → source `directory` = `D:\github\claude-grok-plugin`.
- Plugin payload is a **snapshot copy** at:
  `C:\Users\<user>\.claude\plugins\cache\geseidl-grok\grok\<version>\`
- Config: `~\.claude\plugins\installed_plugins.json` + `known_marketplaces.json`.

So: **repo edits → cache snapshot → restart** before they take effect.

## Deploy after editing the plugin

1. Edit files in `D:\github\claude-grok-plugin\` (repo = source of truth, commit them).
2. Re-sync the cache snapshot:

   ```powershell
   $repo  = "D:\github\claude-grok-plugin"
   $ver   = (Get-Content "$repo\.claude-plugin\plugin.json" | ConvertFrom-Json).version
   $cache = "$env:USERPROFILE\.claude\plugins\cache\geseidl-grok\grok\$ver"
   robocopy $repo $cache /E /XD .git | Out-Null
   if ($LASTEXITCODE -ge 8) { throw "robocopy failed ($LASTEXITCODE)" } else { "synced -> $cache" }
   ```

   - `robocopy` exit codes 0–7 = success; only `>=8` is a real error.
3. **Restart Claude Code** (close + reopen the session).
4. Verify: run `/grok:rescue test` or `grep "grok-4.3" "$cache\agents\grok-rescue.md"`.

## Auth

The runtime needs `XAI_API_KEY` in the environment. It is persisted as a Windows user env var
(`setx XAI_API_KEY "xai-..."`) and backed up in Vault `xAI-Grok-API`. A shell that predates the
`setx` will not have it — restart the session (or `export` it inline) so the subagent's `curl`
calls authenticate.
