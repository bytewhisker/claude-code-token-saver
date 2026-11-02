Reduce my Claude Code startup tokens by turning off the Artifact tool. Follow these steps exactly and change nothing else.

1. Find my user settings file: `~/.claude/settings.json` (on Windows, `%USERPROFILE%\.claude\settings.json`).
2. If the file does not exist, create it containing only `{ "enableArtifact": false }`, then skip to step 6.
3. If it exists, copy it to `settings.json.pre-enableArtifact.<timestamp>.bak` in the same folder and confirm the copy is identical. If the file is not valid JSON, stop and tell me. Do not repair it.
4. Add `"enableArtifact": false` as a top-level key. If the key is already false, tell me and stop. Keep every other key and value exactly as it is.
5. Parse the edited file again and confirm that `enableArtifact` is the only key that differs from the backup. If anything else differs, restore the backup and tell me.
6. Do not touch credentials, model, effort, thinking, permissions, hooks, plugins, MCP servers, project files, or any other file. Do not print secrets from my settings.
7. Tell me what changed, the backup path, and how to undo it. Then tell me to restart Claude Code and run `/context` to see the new token count. Do not claim a savings number you did not measure.
