# Run the MCP server

## Overview

This document describes how to execute the `cat-facts-mcp` CLI to start the MCP (Model Context Protocol) server.

The server connects over stdio using `StdioServerTransport`.

## Prerequisites

- Node.js `>=18.0.0`
- `npm`

## Run modes

### 1. Run the published CLI

`cat-facts-mcp` publishes a binary named `cat-facts-mcp` that points to `./bin/run.js`.

1. Install the package.

   ```sh
   npm install -g cat-facts-mcp
   ```

2. Execute the CLI.

   ```sh
   cat-facts-mcp
   ```

### 2. Run from source (development)

The repository includes `./bin/dev.js`, which executes the oclif command runner with the `ts-node` ESM loader.

1. Install dependencies.

   ```sh
   npm install
   ```

2. Execute the development entrypoint.

   ```sh
   node ./bin/dev.js
   ```

## Verification

When the server successfully connects to stdio, stderr includes the following line:

```text
Server running on stdio!
```
