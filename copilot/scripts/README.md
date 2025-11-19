# Collection Management Scripts

Scripts for managing awesome-brdk collections inspired by [awesome-copilot](https://github.com/github/awesome-copilot/).

## Available Scripts

### Generate Collection Markdown Files

Generate markdown documentation from YAML collection files:

```bash
npm run generate
```

This reads all `.collection.yml` files in the `collections/` directory and generates corresponding `.md` files.

### Validate Collections

Validate all collection YAML files:

```bash
npm run validate
```

This checks that:
- Collection IDs are valid and unique
- Required fields are present
- File paths exist and match the item kind
- Tags and display settings follow the schema

### Create New Collection

Create a new collection interactively:

```bash
npm run create
```

Or with command-line arguments:

```bash
node create-collection.mjs my-collection-id
node create-collection.mjs --id my-collection --tags "tag1,tag2,tag3"
```

### Build (Validate + Generate)

Run validation and generation in one command:

```bash
npm run build
```

## Collection File Structure

Collections use YAML format (`.collection.yml`) with the following structure:

```yaml
id: my-collection-id
name: My Collection Name
description: A brief description of what this collection provides and who should use it.
tags: [tag1, tag2, tag3]
items:
  - path: prompts/my-prompt.prompt.md
    kind: prompt
  - path: instructions/my-instructions.instruction.md
    kind: instruction
  - path: agents/my-agent.agent.md
    kind: agent
    usage: |
      Optional usage instructions for this item.
      Can be multiple lines.
display:
  ordering: alpha # or "manual" to preserve order above
  show_badge: true # set to true to show collection badge
```

## Supported Item Types

- `prompt` - Prompt files (`.prompt.md`)
- `instruction` - Instruction files (`.instruction.md`)
- `agent` - Agent files (`.agent.md`)

**Note:** We use `.agent.md` instead of `.chatmode.md` for the newer agent format.

## Workflow

1. **Create a collection**: `npm run create`
2. **Edit the YAML file** to add your items
3. **Validate**: `npm run validate`
4. **Generate markdown**: `npm run generate`

Or simply: `npm run build`

## File Locations

- Collections YAML: `../collections/*.collection.yml`
- Collections Markdown: `../collections/*.md`
- Prompts: `../prompts/*.prompt.md`
- Instructions: `../instructions/*.instruction.md`
- Agents: `../agents/*.agent.md`
