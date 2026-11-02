# Claude Code Token Saver

**Cut about 10k tokens from every new Claude Code session with one setting.** Paste one prompt into Claude Code and it sets everything up, with a backup.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Claude Code loads a tool called `Artifact` at the start of every session, even if you never publish an Artifact. Turning it off frees that context for your code.

## Quick setup

1. Run `/context` in Claude Code and note the total.
2. Paste this prompt:

```text
Reduce my Claude Code startup tokens by turning off the Artifact tool. Follow these steps exactly and change nothing else.

1. Find my user settings file: `~/.claude/settings.json` (on Windows, `%USERPROFILE%\.claude\settings.json`).
2. If the file does not exist, create it containing only `{ "enableArtifact": false }`, then skip to step 6.
3. If it exists, copy it to `settings.json.pre-enableArtifact.<timestamp>.bak` in the same folder and confirm the copy is identical. If the file is not valid JSON, stop and tell me. Do not repair it.
4. Add `"enableArtifact": false` as a top-level key. If the key is already false, tell me and stop. Keep every other key and value exactly as it is.
5. Parse the edited file again and confirm that `enableArtifact` is the only key that differs from the backup. If anything else differs, restore the backup and tell me.
6. Do not touch credentials, model, effort, thinking, permissions, hooks, plugins, MCP servers, project files, or any other file. Do not print secrets from my settings.
7. Tell me what changed, the backup path, and how to undo it. Then tell me to restart Claude Code and run `/context` to see the new token count. Do not claim a savings number you did not measure.
```

3. Approve the settings edit, restart Claude Code, and run `/context` again.

Prefer to do it yourself? Add this one line to `~/.claude/settings.json`:

```json
{ "enableArtifact": false }
```

If the file already has settings, add only the key and keep the rest.

## How many tokens does it save?

Expect **roughly 9k to 13k tokens per new session**. Your number depends on your version and setup.

| Test | Before | After | Saved |
|---|---:|---:|---:|
| Claude Code 2.1.316, headless run | 44,914 | 31,525 | **13,389** |
| Claude Code 2.1.284, headless run | 35,207 | 26,429 | **8,778** |
| [Original Reddit post](https://www.reddit.com/r/ClaudeCode/comments/1w2ja43/tip_instantly_save_10k_tokens_on_every_new_session/), `/context` | 30k | 19.8k | **about 10k** |

The two headless rows are single runs on one Windows PC, using Haiku and the same plugins and connectors, with total context read from the usage report. They are examples, not a benchmark. Run `/context` for your own number.

## What changes

**You lose:** the `Artifact`, `ArtifactComments`, and `ArtifactData` tools, so Claude can no longer publish hosted Artifact pages. Claude can still write HTML files to your project.

**Not changed by this setting:** your model, effort, and thinking settings, and the tools for reading, writing, and editing files, running terminal commands, searching, and browsing the web.

## Other ways to turn it off

| Method | Scope |
|---|---|
| `"enableArtifact": false` in `~/.claude/settings.json` | Every session |
| `claude --disallowed-tools Artifact` | One session |
| `CLAUDE_CODE_DISABLE_ARTIFACT=1` environment variable | Wherever the variable is set |

## Undo

Delete the `enableArtifact` line from `~/.claude/settings.json`, then restart Claude Code. You can also restore the `.bak` file the prompt created, but that also undoes any settings changes made after the backup.

## FAQ

### How do I reduce Claude Code token usage?

Start with the setting above, because it applies to every session and takes a minute. Then run `/context` to see what else is using space. Connectors and plugins you don't use can also add up.

### Will this save me money or make my usage limit last longer?

It shrinks the context each session starts with. How much that changes your bill or limits depends on your plan and on caching, and it has not been measured here.

### Does it work on my version?

Versions 2.1.284 and 2.1.316 were tested and both list the setting. Claude Code 2.1.81 does not appear to include it. That version still started normally with the key present, but the setting probably does nothing there. Update first if `/context` shows no change.

### Will I be asked to approve the change?

Usually yes. Read the prompt first: it is short and only edits one settings file.

### Is this an official Anthropic project?

No. This is a community tip and is not affiliated with Anthropic.

## Credit

The tip comes from [u/nNaz on r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1w2ja43/tip_instantly_save_10k_tokens_on_every_new_session/).

If this saved you tokens, star the repo so other Claude Code users can find it. Better numbers or tips are welcome as issues and pull requests.

## License

[MIT](LICENSE)
