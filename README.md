# ft

Release binaries for `ft`, the [Freshtimes](https://freshtim.es) command line: exact time with its
error bound, market hours and phases, overlap windows, time conversion and holidays. It also runs an
MCP server for agents (`ft mcp`).

```sh
brew install arenzana/tap/ft
```

The same binary is installed as `ft` and `freshtimes`. Builds for macOS and Linux (arm64 and amd64)
are on the [releases page](https://github.com/arenzana/freshtimes-cli/releases).

```sh
ft now Madrid Tokyo
ft markets --asset futures
ft window lse nyse --zone Madrid
ft mcp
```

This repo holds releases only; issues and source are not public.

## Agent skill

[`skills/freshtimes`](skills/freshtimes/SKILL.md) teaches an AI agent when and how to use `ft` and the MCP server.
For Claude Code, copy the folder into `~/.claude/skills/`.
