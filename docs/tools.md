# MCP tools

## Overview

This document describes the MCP tools exposed by `cat-facts-mcp`.

Tool discovery and invocation are implemented as follows:

- Tool listing: the MCP server responds to `ListToolsRequestSchema` by returning `LayerAPI.getAllTools()`.
- Tool execution: the MCP server responds to `CallToolRequestSchema` by calling `LayerAPI.callTool()`.

## Prerequisites

- A running `cat-facts-mcp` MCP server

## Tool discovery

Tool discovery returns a list that includes:

1. A built-in tool named `search_workflows_and_docs`.
2. A set of additional tools fetched from the Layer API endpoint `GET {baseUrl}/mcp/tools`.

`baseUrl` is selected from the configured environment:

- `production`: `https://api.buildwithlayer.com`
- `staging`: `https://api.staging.buildwithlayer.com`
- `development`: `http://localhost`

## Built-in tool: `search_workflows_and_docs`

### Purpose

The server defines a built-in tool named `search_workflows_and_docs` with the following description:

> ALWAYS EXECUTE THIS TOOL FIRST UNLESS THE TOOL TO BE USED IS OBVIOUS. It will return relevant workflows and documentation based on the user's query.

### Input schema notes

The tool declares an `inputSchema` object with `additionalProperties: false` and `required: ['body']`.

The schema also defines `$defs` for message and tool-call shaped objects (for example: `Message`, `ToolCall`, and `ToolMessage`).

### Execution behavior

When called, the tool issues an HTTP request:

- Method: `POST`
- Path: `{baseUrl}/chat/search`
- Headers:
  - `Accept: application/json`
  - `Layer-Api-Key: <configured API key>`
- JSON body: the MCP `arguments` passed to the tool

The result content is returned as an array of MCP `text` content items; each item contains a JSON-stringified `source` object from the `{ sources: SearchResult[] }` response.

If the HTTP request fails with a `got` `RequestError`, the tool returns `isError: true` and includes the error message as a `text` content item.

## Layer-provided tools

### Listing

Layer-provided tools are fetched from:

```text
GET {baseUrl}/mcp/tools
```

The request includes the headers `Accept: application/json` and `Layer-Api-Key`.

### Input defaulting via CLI flags

After tools are fetched, the server applies defaults into each tool's `inputSchema` using `addDefaultsToSchema(tool.inputSchema, overrides)`.

This defaulting behavior is driven by CLI flags captured by `getOverrides(flags)` and passed into `new LayerAPI(layerApiKey, environment, overrides)`.

The defaulting logic:

- Sets `schema.default` for string-typed fields whose identifier matches an override key.
- Removes overridden keys from `schema.required`.

### Execution

Layer-provided tools are invoked through:

```text
POST {baseUrl}/mcp/tools/call
```

The request includes the `Layer-Api-Key` header and uses the MCP `arguments` as the JSON body.

The response body is returned as a JSON-stringified `text` content item.

If the HTTP request fails with a `got` `RequestError`, the tool returns `isError: true` and includes the error message as a `text` content item.

## Troubleshooting

### Error: "Tool '<name>' is not available"

This error indicates that the requested tool name is not present in the server's in-memory tool list.

Resolution steps:

1. Request the tool list again.
2. Verify the tool name matches one of the returned `tools[].name` values.

### Validation errors

Before a tool is executed, the server validates `arguments` against the tool's JSON Schema using `ajv`.

If validation fails, the server throws an error containing a JSON string of `ajv` validation errors.
