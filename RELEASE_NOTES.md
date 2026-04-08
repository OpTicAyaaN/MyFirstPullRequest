---
title: Release notes
---

# Release notes

## 1.1.1 - update

- **Date**: 0001-01-01
- **Version**: 1.1.1

### Summary

This release includes experimental provider actions, provider-managed state capabilities, and query support, along with diagnostics API updates and reliability improvements.

### New Features

- Added experimental `state_store` support for provider-driven state storage outside traditional backends.
- Added tfplugin6 support for listing and deleting stored provider-managed states via `GetStates` and `DeleteState`.
- Added experimental `terraform query` command for provider-backed list/query operations with human and JSON output.
- Added provider action support in tfplugin5/tfplugin6 with plan and invoke RPCs and schema-aware `ValidateActionConfig`.
- Added provider-driven configuration generation for existing infrastructure via `GenerateResourceConfig` in tfplugin5/tfplugin6.
- Added action targeting support in plan/apply via action and action-instance target addresses.

### Breaking Changes

- Updated workspace backend APIs (`Workspaces`, `DeleteWorkspace`) to return `tfdiags.Diagnostics` instead of `error`.
- Updated remote state backend APIs (`Get`, `Put`, `Delete`) to return `tfdiags.Diagnostics` instead of `error`.

### Bug Fixes

- Fixed OSS backend proxy resolution by using `httpproxy`-based proxy handling.
- Fixed Azure remote state backend validation to emit diagnostics for missing credentials and invalid `lookup_blob_endpoint` usage.
- Fixed `terraform test` deadlocks when run parallelism is set to `1`.
- Fixed `terraform test` to exit with code `0` when no tests are found.
- Fixed evaluator/expander handling of incomplete references to return diagnostics instead of panicking.
- Fixed apply-time mismatch warnings to redact sensitive variable values.
- Fixed list query handling to stop on error diagnostics and to improve output for empty results.
- Fixed linked-resource diagnostics indexing and `planned_state` decoding for protocol v5 and v6.
- Fixed identity propagation across plan/apply, including failed destroy operations and attribute sensitivity changes.

### Performance

- Updated the planner to skip redundant equality checks when correlating planned set changes.

### Security

- Updated cryptography dependencies to include upstream security and correctness fixes.

### Deprecations

- Updated configuration generation to favor provider-driven, schema-aware extraction and to reduce reliance on legacy mark-based filtering.

### Additional Changes

- Added stacks plugin cache directory support via a configurable cache path for reusable plugin downloads.
- Updated built-in provider handling to treat built-ins as implicitly required without explicit configuration.
- Updated provider list resources to use a nested `config` block and to support query `limit`.
- Updated backend configuration handling for consistent `Backend.SetConfig` usage and schema-based validation across `backend` and `state_store`.
- Updated module test graph execution to improve determinism and variable handling.
- Updated plan and protocol serialization for action invocations to use nested `Changes` and updated hook identity rendering.
- Updated project infrastructure and build dependencies, including a move to Go 1.25.
