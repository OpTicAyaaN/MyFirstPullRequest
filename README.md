# cat-facts mcp documentation

## overview

This repository contains instructional documentation for `cat-facts-mcp`, an MCP (Model Context Protocol) server implemented in TypeScript.

## documentation

- [Run the MCP server](./docs/running-the-server.md)
- [MCP tools](./docs/tools.md)

## schemas

The following TypeScript types are defined in `src/types.ts`.

```ts
import {JSONSchema7} from 'json-schema';

export type LayerEnvironment = 'development' | 'production' | 'staging';

export interface Tool {
    description?: string;
    inputSchema: JSONSchema7;
    name: string;
    url?: string;
}

export interface SearchResult {
    name: string;
    type: string;
    url?: string;
    value: string;
}

export interface FlaggableAuthDetails {
    description?: string;
    name: string;
}
```