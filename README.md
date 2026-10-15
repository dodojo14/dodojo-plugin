# dodojo plugin

Bundles the [dodojo](https://www.dodojo.ai) memory connector with the skill that
teaches Claude how to use it.

- **MCP connector** — the dodojo Streamable HTTP MCP server at
  `https://www.dodojo.ai/api/mcp`, authorized with OAuth 2.1
  (authorization code + PKCE S256). Nineteen tools: eleven read-only, eight write.
- **Skill** — routing and safety instructions: which tool answers which request,
  how to write a memory that is still findable months later, and the boundaries
  the skill must not cross (encrypted and Only-Me memories are never claimed to
  be readable; an agent may only trash a memory an agent wrote).

## Install

Inside Claude Code:

```
/plugin marketplace add https://www.dodojo.ai/plugin/marketplace.json
/plugin install dodojo@dodojo
```

Then connect dodojo when Claude asks. Everything runs against your own dodojo
account through OAuth — the plugin never asks for a password, an API key, or a
database connection.

## What it can do

- Save text, links, images, documents and audio into your private space
- Find things by keyword, including text inside attached PDFs, slide decks
  and spreadsheets, and by meaning when you have turned that setting on
- Read what a memory said before it was changed, and act on the annotations you
  left on an attached document
- Organize memories into collections, set reminders, and send yourself a
  completion notice through the channels you already connected

## Privacy

Memories you encrypt or mark **Only Me** are closed to agents — their current
text and their version history alike. The MCP server, not this plugin, owns
that enforcement.

- [Privacy policy](https://www.dodojo.ai/privacy)
- [Documentation](https://www.dodojo.ai/resources/mcp-and-api)
- [Support](https://www.dodojo.ai/support)

## Source of truth

The skill in this repository is generated from `skills/dodojo` in the dodojo
product repository. Do not edit it here — edits are overwritten by the sync.
