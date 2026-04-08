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
    "The stacks plugin handling now supports a configurable cache directory so plugin downloads can be reused across runs.",
    "Built-in providers are now always available without requiring explicit configuration.",
    "Provider list resources now use a nested `config` block and support additional request controls.",
    "Backend configuration handling was refactored to be more consistent across backends and state stores.",
    "Terraform’s module test system now has richer and more deterministic graph execution and variable handling.",
    "Terraform’s plan and protocol serialization for action invocations was expanded and reorganized as the feature evolved.",
    "Terraform updated various project infrastructure and build dependencies to stay current."
  ],
  "Breaking Changes": [
    "Terraform’s workspace-related backend APIs now return diagnostics rather than a single error so callers can handle warnings and multiple failures consistently.",
    "Remote state backends now report failures using diagnostics rather than returning a single error value."
  ],
  "Bug Fixes": [
    "Terraform OSS backend proxy resolution is now more reliable in environments that require proxies.",
    "Azure remote state backend configuration errors now surface as clear validation feedback instead of failing silently.",
    "`terraform test` no longer risks hanging when run parallelism is set to 1.",
    "`terraform test` now exits successfully when no tests are found instead of failing unexpectedly.",
    "Terraform test and evaluation now fail safely instead of panicking when references are incomplete or invalid.",
    "Terraform no longer prints sensitive variable contents in mismatch warnings during apply.",
    "List queries now handle provider diagnostics more safely and produce clearer output when no results are returned.",
    "Linked-resource diagnostics and planning behavior now report and decode values correctly.",
    "Resource identities are now preserved more consistently across plan and apply, including failure scenarios."
  ],
  "Deprecations": [
    "Terraform’s configuration generation now favors provider-driven and schema-aware extraction paths over older mark-based filtering behaviors."
  ],
  "New Features": [
    "Terraform now supports provider-driven state storage via an experimental `state_store` block so you can store state outside of traditional backends.",
    "The provider plugin protocol now supports listing and deleting stored provider-managed states.",
    "Terraform now includes an experimental `terraform query` command to run provider-backed list/query operations and display results.",
    "Providers can now expose actions through the plugin protocol so Terraform can plan and invoke them.",
    "Terraform can now ask providers to generate configuration for existing infrastructure so you can bootstrap configuration from imported state.",
    "Terraform now supports targeting actions in plan/apply so you can invoke only specific actions when desired."
  ],
  "Performance": [
    "Terraform now avoids redundant comparisons when correlating planned set changes, reducing unnecessary work during planning."
  ],
  "Security": [
    "Terraform updated cryptography dependencies to include upstream security and correctness fixes."
  ]
}

```
