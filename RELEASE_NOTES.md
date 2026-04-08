---
title: Release notes
---

# Release notes

## 1.1.1 - update

- **Date**: 0001-01-01
- **Version**: 1.1.1

### Summary

This release includes the changes listed in the sections below.

### New Features

```json
{
  "Additional Changes": [
    "The stacks plugin handling now supports a configurable cache directory so plugin downloads can be reused across runs. This adds a new CLI flag for the cache path, persists lookup state, strips `-plugin-cache-dir` args before invoking the plugin, increases allowed plugin files to 3, and renames the stacks plugin binary to `tfstacks`.",
    "Built-in providers are now always available without requiring explicit configuration. Terraform now maintains a shared built-in providers registry and treats built-ins as implicitly required, with tests covering omission and explicit declaration cases.",
    "Provider list resources now use a nested `config` block and support additional request controls. Terraform now parses and schema-validates a nested list-resource `config`, supports query block `limit`, and wires a `limit` field through tfplugin5/tfplugin6 descriptors and `ListResourceRequest` construction.",
    "Backend configuration handling was refactored to be more consistent across backends and state stores. Terraform now uses `Backend.SetConfig` directly with nil-receiver checks and introduces reusable config-state types for backend and `state_store` with schema-based validation and updated planfile handling.",
    "Terraform’s module test system now has richer and more deterministic graph execution and variable handling. This includes dependency-aware parallel teardown, run output/state tracking in the eval context, per-test-file variable definitions and resolution order changes, and graph-based variable nodes that can depend on other variables and run outputs.",
    "Terraform’s plan and protocol serialization for action invocations was expanded and reorganized as the feature evolved. Internally, action invocation data moved into nested `Changes`, hook identities and JSON rendering were updated for trigger metadata, and protobuf field-number adjustments were made to keep tfplugin5/tfplugin6 compatibility.",
    "Terraform updated various project infrastructure and build dependencies to stay current. This includes dependency refreshes (such as `dario.cat/mergo`, `go-getter`, `google.golang.org/protobuf`), workflow pin updates for GitHub Actions, and Go toolchain updates including a move to Go 1.25 with accompanying upgrade notes."
  ],
  "Breaking Changes": [
    "Terraform’s workspace-related backend APIs now return diagnostics rather than a single error so callers can handle warnings and multiple failures consistently. Specifically, backend `Workspaces`/`DeleteWorkspace` and related command/workspace code paths were updated to return and consume `tfdiags.Diagnostics`, checking `HasErrors()` and reporting `diags.Err()` as needed.",
    "Remote state backends now report failures using diagnostics rather than returning a single error value. The remote state `Get`/`Put`/`Delete` APIs were migrated from `error` returns to `tfdiags.Diagnostics`, and all implementations, callers, and tests were updated to preserve warning information and standardized reporting."
  ],
  "Bug Fixes": [
    "Terraform OSS backend proxy resolution is now more reliable in environments that require proxies. Terraform now uses `httpproxy`-based proxy resolution for the OSS backend path and includes tests to prevent regressions.",
    "Azure remote state backend configuration errors now surface as clear validation feedback instead of failing silently. The Azure backend now emits `backendbase.ErrorAsDiagnostics` diagnostics for missing credential inputs and for `lookup_blob_endpoint` usage without `resource_group_name`, and the backend implementation was moved onto the current `backendbase.Base`/`configschema.Block` framework.",
    "`terraform test` no longer risks hanging when run parallelism is set to 1. Module test graph semaphore acquisition now skips subgraph nodes via a `Subgrapher` interface and a run-parallelism test flag to prevent deadlocks.",
    "`terraform test` now exits successfully when no tests are found instead of failing unexpectedly. The test command now returns exit code 0 for the zero-tests case and includes regression coverage for the corrected behavior.",
    "Terraform test and evaluation now fail safely instead of panicking when references are incomplete or invalid. The evaluator/expander now guards incomplete reference resolution and returns controlled diagnostics, with new tests covering these error paths.",
    "Terraform no longer prints sensitive variable contents in mismatch warnings during apply. Planned and applied values are now marked sensitive before rendering diagnostics so output redacts the raw values, with fixtures validating the behavior.",
    "List queries now handle provider diagnostics more safely and produce clearer output when no results are returned. Terraform now stops processing on error diagnostics, skips warning-only events that lack identity, suppresses empty per-block output, and emits a consolidated warning and structured `QueryComplete` JSON payload.",
    "Linked-resource diagnostics and planning behavior now report and decode values correctly. Provider gRPC wrappers now format linked-resource decode failures using the correct index and properly decode and pass through linked-resource `planned_state` for both v5 and v6 protocol implementations.",
    "Resource identities are now preserved more consistently across plan and apply, including failure scenarios. Terraform now carries identity through failed destroy operations and updates identity correctly when attribute sensitivity changes."
  ],
  "Deprecations": [
    "Terraform’s configuration generation now favors provider-driven and schema-aware extraction paths over older mark-based filtering behaviors. Internally, config generation was reworked to extract values at the state-value level (including computed/deprecated/sensitive handling) via helpers like `BlockByPath` and `ExtractLegacyConfigFromState`, reducing reliance on legacy schema filters and value marks."
  ],
  "New Features": [
    "Terraform now supports provider-driven state storage via an experimental `state_store` block so you can store state outside of traditional backends. Under the hood, Terraform parses `state_store`, enforces mutual exclusivity with `backend`/Terraform Cloud, plans a dedicated `StateStore` config state, and wires schema-based validation/configuration through tfplugin6 provider interfaces and gRPC stubs/mocks.",
    "The provider plugin protocol now supports listing and deleting stored provider-managed states. This adds tfplugin6 `GetStates` and `DeleteState` RPCs and threads them through provider interfaces, gRPC wrappers, mocks, and stacks stubs so integrations can enumerate and remove state by `type_name` and `state_id`.",
    "Terraform now includes an experimental `terraform query` command to run provider-backed list/query operations and display results. Internally, this introduces query-mode planning/execution for `list` blocks, new human and JSON output renderers, and gRPC `ListResource` request/response handling that is restricted to query plans and excluded from normal apply graphs.",
    "Providers can now expose actions through the plugin protocol so Terraform can plan and invoke them. This introduces provider action schemas and Plan/Invoke RPC support in tfplugin5/tfplugin6, adds action lifecycle hooks and UI/JSON plumbing, and wires action config validation via `ValidateActionConfig` with schema-aware diagnostics.",
    "Terraform can now ask providers to generate configuration for existing infrastructure so you can bootstrap configuration from imported state. This adds `GenerateResourceConfig` to tfplugin5/tfplugin6 (including capability flags, RPCs, mocks, and gRPC handling) and refactors generated-config handling around `ImportGroup`/`ResourceImport`, using provider-supplied config when available with legacy extraction as a fallback.",
    "Terraform now supports targeting actions in plan/apply so you can invoke only specific actions when desired. This adds dedicated parsing/validation for action and action-instance target addresses and propagates `ActionTargets` through the operation pipeline, graph transforms, and planning context."
  ],
  "Performance": [
    "Terraform now avoids redundant comparisons when correlating planned set changes, reducing unnecessary work during planning. The planner skips equality checks when values are already matched, and this optimization is covered by new changelog-noted tests."
  ],
  "Security": [
    "Terraform updated cryptography dependencies to include upstream security and correctness fixes. Notably, `github.com/cloudflare/circl` was updated to v1.6.1 and other `golang.org/x` dependencies were refreshed across modules."
  ]
}

```
