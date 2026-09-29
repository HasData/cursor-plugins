# HasData plugins for Cursor

The Cursor marketplace source for [HasData](https://hasdata.com).

| Plugin | What it does |
| --- | --- |
| [`hasdata`](./plugins/hasdata) | Public web pages as structured JSON over one hosted MCP server, with a rule and a command per site and skills that span several |

## Layout

```
.cursor-plugin/marketplace.json   marketplace manifest
plugins/hasdata/
  .cursor-plugin/plugin.json      plugin manifest
  assets/logo.svg
  mcp.json                        the hosted MCP server
  rules/                          one .mdc per site, agent-requested
  commands/                       one slash command per site
  skills/                         cross-service workflows
  README.md
```

## Validating

The repository follows the [cursor/plugin-template](https://github.com/cursor/plugin-template) layout. To check it, run the template validator with this repository as the working directory:

```bash
node /path/to/plugin-template/scripts/validate-template.mjs
```

## Contributing

The rules and commands are written from measured API payloads rather than from documentation, so a correction is welcome when a field or a failure mode has changed. Open an issue with the tool name and the response you saw.

## License

MIT
