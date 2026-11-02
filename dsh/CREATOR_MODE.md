# DeepSeek Harness: Creator Mode prompts

Export your key first (`export MARGIN_INGEST_KEY=mgk_…` in the shell that starts DSH). Then open
DSH, pick **Creator mode** and paste one of these.

## 1. Connect Margin

```
Configure the MCP server at https://trymargin.io/api/mcp in this profile as `margin`, sending the header Authorization: Bearer followed by the MARGIN_INGEST_KEY environment variable, read at runtime and never written into a file. Make its tools available now, then call margin_list_projects and tell me the result.
```

## 2. A cost panel in the sidebar

```
Create a DeepSeek Harness plugin with a left sidebar entry called Margin that uses the margin MCP tools. Show my project's spend per job, the steps worth a face-off, and the latest face-off verdicts, from my real Margin data only. Keep the default page minimal: those three sections, no extra headings, no sample data. It only reads; for starting a face-off or opening a pull request, link to https://trymargin.io/console.
```

The config in `cordis.patch.yml` does the same as prompt 1 without Creator Mode.
