# Margin plugins

Margin inside your coding agent: see what each AI job costs, which steps are worth a face-off, and the verdicts. The tools only read. Face-offs and pull requests stay in the [Margin console](https://trymargin.io).

You need a Margin ingest key (`mgk_…`) from the console or [trymargin.io/start](https://trymargin.io/start).

## Claude Code

```
/plugin marketplace add trymargin/margin-plugins
/plugin install margin@margin
```

Claude Code asks for the key when the plugin is enabled and stores it in your system's credential store, never in a settings file. Four skills come with it: `margin:connect`, `margin:costs`, `margin:check`, `margin:status`.

## Cursor

Copy `cursor/mcp.json` into `.cursor/mcp.json` and `cursor/margin.mdc` into `.cursor/rules/`. Cursor reads the key from `MARGIN_INGEST_KEY`.

## Codex

Append `codex/config.toml` to `~/.codex/config.toml`. Codex reads the key from `MARGIN_INGEST_KEY`.

## DeepSeek Harness

Append `dsh/cordis.patch.yml` to your profile's `cordis.patch.yml`, or run `dsh <profile> --patch dsh/cordis.patch.yml`. DSH reads the key from `MARGIN_INGEST_KEY`. In Creator Mode you can also paste the prompts in `dsh/CREATOR_MODE.md`: one connects Margin, one builds a cost panel in the sidebar.

## What it reads

The MCP server at `https://trymargin.io/api/mcp` returns your project's own cost, outcome and recommendation data, scoped to the key. It never sees your prompts. Docs: [trymargin.io/docs](https://trymargin.io/docs).

MIT licensed.
