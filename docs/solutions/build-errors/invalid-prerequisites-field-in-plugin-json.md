---
title: "Invalid prerequisites field in plugin.json"
date: 2026-02-12
category: build-errors
module: plugin-schema
tags: [plugin-json, schema, marketplace, claude-code]
symptoms:
  - plugin.json contains unrecognized fields
  - plugin fails validation or behaves unexpectedly
severity: medium
---

# Invalid `prerequisites` field in plugin.json

## Problem

When creating or moving a `.claude-plugin/plugin.json`, a `prerequisites` field was included listing plugin dependencies. This field does not exist in the Claude Code plugin schema and was silently accepted but meaningless.

```json
{
  "prerequisites": [
    "compound-engineering (>=2.30.0) - https://github.com/EveryInc/compound-engineering-plugin",
    "ralph-loop (optional) - official Claude plugin"
  ]
}
```

## Root Cause

The original `plugin.json` was authored with an invented `prerequisites` field during initial plugin creation. When restructuring the repo as a marketplace, the field was blindly copied forward instead of being validated against the actual schema.

## Solution

Remove the field. Encode dependency information in the `description` field instead, since there is no programmatic dependency declaration in plugin.json.

### Valid plugin.json fields

```json
{
  "name": "string",
  "version": "string",
  "description": "string (include dependency info here)",
  "author": { "name": "string", "url": "string" },
  "repository": "string",
  "license": "string",
  "keywords": ["array", "of", "strings"]
}
```

### Fix applied

```json
{
  "description": "End-to-end development pipeline: brainstorm, plan, deepen, review, implement, compound. One command to rule them all. Requires compound engineering >=2.30.0 (https://github.com/EveryInc/compound-engineering-plugin) and (optional) ralph-loop (official Claude plugin)"
}
```

## Prevention

- Do not invent JSON schema fields — only use fields that exist in the actual spec
- When copying or moving config files, validate each field against the schema
- Treat "no error on unknown fields" as silent failure, not acceptance
