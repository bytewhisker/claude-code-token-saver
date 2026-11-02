# Benchmark: Claude Code with the Artifact tool on vs off

**Result: turning off the Artifact tool saved 13,354 tokens per request (36.8% of starting context) on Claude Opus 5.5 at high effort.** Tests passed in every coding run with it on or off.

On a small coding task, total tokens sent fell 23.4%, while API-equivalent cost fell about 7%, because most repeated tokens are cheap cached reads.

## Setup

| | |
|---|---|
| Claude Code | 2.1.316 on Windows |
| Model | `claude-opus-5-5`, confirmed in API responses |
| Effort | `high`, set with `--effort high` (the output does not report it back) |
| Mode | Headless (`claude -p`) |
| Changed between arms | Only `enableArtifact` |
| Removed from both arms | User settings, hooks, and connectors (`--setting-sources project,local`, `--strict-mcp-config`) |
| Run order | On, off, off, on, on, off, so neither arm gets more prompt-cache warmth |

Context tokens are `input_tokens + cache_read_input_tokens + cache_creation_input_tokens` from each API response.

## Test 1: Session start

Prompt: `Reply with OK.`

| Run | Artifact on | Artifact off |
|---|---:|---:|
| 1 | 36,328 | 22,974 |
| 2 | 36,330 | 22,974 |
| 3 | 36,326 | 22,974 |
| **Average** | **36,328** | **22,974** |

**Saved: 13,354 tokens (36.8%).** The on arm had 32 tools and the off arm had 29; the difference is `Artifact`, `ArtifactComments`, and `ArtifactData`. Skills also fell from 37 to 34.

The on arm rewrote about 6,950 tokens to the prompt cache every run. The off arm wrote none after its first run. The cause was not investigated.

## Test 2: Small coding task

The fixture is four small JavaScript files with two bugs and five `node --test` tests, recreated for every run.

Prompt:

```text
The tests in this folder are failing. Run `node --test`, find the bugs in the source files, fix them without changing the test file, then run the tests again to confirm they pass. Reply with one short line when done.
```

| Run | Arm | API requests | First request | Total context sent | Output tokens | API-equivalent cost | Time | Tests pass |
|---|---|---:|---:|---:|---:|---:|---:|---|
| 1 | On | 5 | 36,411 | 192,306 | 113 | $0.168 | 14.2s | Yes |
| 2 | Off | 6 | 23,054 | 157,742 | 162 | $0.163 | 16.0s | Yes |
| 3 | Off | 6 | 23,054 | 153,479 | 161 | $0.158 | 14.0s | Yes |
| 4 | On | 4 | 36,412 | 155,814 | 114 | $0.160 | 12.0s | Yes |
| 5 | On | 5 | 36,409 | 193,063 | 137 | $0.175 | 15.1s | Yes |
| 6 | Off | 4 | 23,054 | 103,058 | 114 | $0.147 | 13.4s | Yes |

| Average | Artifact on | Artifact off | Saved |
|---|---:|---:|---:|
| First request | 36,411 | 23,054 | **13,357 (36.7%)** |
| Total context sent | 180,394 | 138,093 | **42,301 (23.4%)** |
| API requests | 4.67 | 5.33 | Not a saving: varies by run |
| API-equivalent cost | $0.168 | $0.156 | **$0.012 (7.0%)** |
| Time | 13.8s | 14.5s | No clear difference |

Runs 4 and 6 both used 4 requests, which makes the cleanest comparison: 155,814 vs 103,058, or 52,756 fewer tokens (about 13,200 per request).

The test file was unchanged in all 6 runs, checked by hash.

## With plugins and connectors on

An earlier test kept a real setup in place, with one plugin that adds its own context and two connectors. These were single headless runs on Haiku.

| Version | Artifact on | Artifact off | Saved |
|---|---:|---:|---:|
| 2.1.316 | 44,914 | 31,525 | 13,389 |
| 2.1.284 | 35,207 | 26,429 | 8,778 |

Savings differ by version, so check your own with `/context`.

## What else uses context

These were also measured on Haiku with Artifact already off. They depend on your setup.

| Turned off | Tokens saved |
|---|---:|
| Claude Docs connector (8 tools) | 1,715 to 1,925 |
| Figma plugin (16 skills) | 2,454 to 2,654 |
| Caveman plugin | 1,923 |

## Limits

- **Small sample:** each arm had 3 runs per test, and the coding task was tiny. This doesn't show how code quality holds up on large projects.
- **Headless only:** interactive sessions may differ, so run `/context` before and after.
- **Isolated setup:** the Opus tests removed hooks and connectors to isolate the tool. Your total will be higher.
- **Cost is an estimate:** figures are Claude Code's `total_cost_usd`. Subscription users don't pay per token, and the effect on usage limits was not measured.
- **Claude Docs connector:** with `enableArtifact: false`, its tools were missing at startup in many `claude -p` runs. The cause is unproven, and they worked in the one interactive session checked. Scripts that need them should verify they loaded.
- **First attempt discarded:** one run set showed 15,070 saved, but the connector had loaded only in the on arm, so it was not a clean comparison.

## Run it yourself

Your user settings can force Artifact off, so the commands skip them. Run each several times in an empty folder and compare the usage numbers:

```bash
# Artifact on
claude -p "Reply with OK." --model claude-opus-5-5 --effort high \
  --setting-sources project,local --settings '{"enableArtifact":true}' \
  --strict-mcp-config --no-session-persistence --output-format stream-json --verbose

# Artifact off
claude -p "Reply with OK." --model claude-opus-5-5 --effort high \
  --setting-sources project,local --settings '{"enableArtifact":false}' \
  --strict-mcp-config --no-session-persistence --output-format stream-json --verbose
```

The `init` event lists the loaded tools. Each `assistant` event has the usage numbers. These runs use your Claude quota.
