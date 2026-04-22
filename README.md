# Coding Harness Assistant Rules for sandgardenhq/mono Repository

## Quick Reference

**CRITICAL** rules - check these first:

- **ALWAYS** run linting and tests before committing any changes
- **ALWAYS** use commit format: `package/path: action description`
- **ALWAYS** use `go test -short` by default (full E2E only when explicitly requested)
- **NEVER** commit secrets, credentials, or sensitive values
- **NEVER** modify generated files directly - use `go generate ./...`
- **NEVER** assume libraries exist - check `go.mod`, `package.json` first
- **NEVER** create mock files manually - use mockery configuration
- **ALWAYS** declare new mocks in `sfs/.mockery.yaml` or `kernel/.mockery.yaml`
- **ALWAYS** use code generation for ID types, API models, and endpoints (see [Code Generation Guide](docs/CODE_GENERATION.md))
- **PREFER** editing existing files over creating new ones

---

## Red Lines (NEVER DO)

These are hard prohibitions. Violating these causes immediate problems:

| Prohibition                                                      | Reason                                                                                  |
| ---------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **NEVER** commit secrets or API keys                             | Security breach risk                                                                    |
| **NEVER** modify generated files directly                        | Changes will be overwritten                                                             |
| **NEVER** skip linting and tests before committing               | Lint errors break CI                                                                    |
| **NEVER** use `log.*` for application code                       | Use `slog` for structured logging                                                       |
| **NEVER** create new files when editing existing ones would work | Reduces code sprawl                                                                     |
| **NEVER** create mock files manually                             | Use mockery config files; manual mocks get overwritten                                  |
| **NEVER** use `_ = variable` to silence unused variable warnings | Indicates dead code; either use the variable or remove the code that produces it        |
| **NEVER** duplicate production logic in test helpers             | Tests should call production code, not reimplement it; use mocks for isolation          |
| **NEVER** keep unused code "for later"                           | Delete unused props, parameters, exports immediately; don't leave dead code             |
| **PREFER** private functions over public ones                    | Bias towards smaller public package interfaces; only expose what's absolutely necessary |

---

## Project Context and Structure

This is a monorepo containing multiple interconnected projects:

### Backend Applications

- **sfs/** - Go backend serving Document Writer SaaS Doc Holiday
- **kernel/** - Shared Go utilities and libraries for all backend apps

### Frontend Applications

- **js/** - Root NextJS workspace using pnpm
- **js/apps/doc.holiday/** - Frontend web application for Document Writer SaaS Doc Holiday
- **js/packages/** - Shared components, hooks, and utilities

### Ignored Directories

- **ssl/** - Ignore this directory
- **vendor/** - Ignore this directory

---

## Coding Style

### Go

**Comments**: Do NOT add unnecessary comments. Leave existing comments alone. Add explanatory comments only for complex logic that needs clarification.

**Logging**: Use `slog` for application code:

```go
slog.Info("operation complete", "user_id", userID)
slog.Error("failed to create object", "error", err)
slog.Warn("deprecated endpoint called", "endpoint", r.URL.Path)
```

**Struct Tags**: Always use `db:"fieldname"` tags for database-mapped fields:

```go
type User struct {
    ID        int64     `db:"id"`
    Name      string    `db:"name"`
    CreatedAt time.Time `db:"created_at"`
}
```

**Error Handling**: Use concrete error types from `kernel/errtype` package when possible.

### Go + SQL

When writing Go code that embeds SQL queries, follow these conventions:

**SQL Formatting**:

- Write SQL as raw string literals (backtick-quoted)
- Place opening backtick immediately after function argument
- Indent SQL one level deeper than function body
- Vertically align fields/values in INSERT statements
- List each column on its own line in SELECT statements

**INSERT Example**:

```go
row, err := pgxplus.CollectExactlyOneRow[idRow](ctx, t.db, `
    INSERT INTO text_documents ( content,  created_at)
    VALUES                     (@content, @created_at)
    RETURNING id
`,
    pgx.StrictNamedArgs{
        "content":    content,
        "created_at": time.Now(),
    },
)
```

**SELECT Example**:

```go
qb := pgxplus.NewQuery(`
    SELECT
        id,
        content,
        created_at
    FROM
        text_documents
    WHERE
        search_vector @@ plainto_tsquery('english', @query)
`,
    "query", query,
)
```

**Database Functions**: Use these pgxplus patterns:

- `pgxplus.CollectOneRow` - single row, may not exist
- `pgxplus.CollectExactlyOneRow` - single row, must exist
- `pgxplus.CollectAll` - multiple rows
- `pgxplus.Exec` - no return value

### Testing Conventions

**Subtest Names**: Use camelCase for subtest names:

```go
// DO:
t.Run("missingOrgID", func(t *testing.T) {
t.Run("invalidInput", func(t *testing.T) {

// DON'T:
t.Run("missing org ID", func(t *testing.T) {
t.Run("Missing_Org_ID", func(t *testing.T) {
```

**Benchmarks**: Use `b.Loop` pattern:

```go
func BenchmarkFoo(b *testing.B) {
    for b.Loop() {
        // benchmark code
    }
}
```

**Cleanup**: Prefer `t.Cleanup()` over `defer`:

```go
// DO:
t.Cleanup(func() {
    db.Close()
})

// DON'T (unless cleanup order matters):
defer db.Close()
```

**Test Mode**: Use `go test -short` by default. Only run full E2E tests when explicitly requested.

**Test Helper Anti-Pattern**: Never write test helper functions that reimplement production logic. If test code duplicates the same algorithm as production code:

- Tests are testing the test code, not actual behavior
- Production and test must be kept in sync manually (error-prone)
- Instead: Extract shared logic into production code that tests can call directly

### Mock Generation

All test mocks are generated via [mockery](https://github.com/vektra/mockery) using the matryer template.

**Configuration Files:**

- `sfs/.mockery.yaml` - Mocks for SFS package interfaces
- `kernel/.mockery.yaml` - Mocks for kernel utilities (pgxplus, etc.)

**Adding New Mocks:**

1. Add interface configuration to the appropriate `.mockery.yaml` file
2. Run `go generate ./sfs ./kernel` to generate the mock files
3. Never edit generated mock files directly

**Mock Naming Convention:**

- Use lowercase struct names: `structname: "myInterfaceMock"`
- Place mocks in `*_mocks_test.go` files
- Set `skip-ensure: True` in template-data

---

## Code Generation (MANDATORY)

**Full documentation: [docs/CODE_GENERATION.md](docs/CODE_GENERATION.md)**

This project uses extensive code generation. You MUST use the appropriate generator when:

| Creating This                     | Use This Generator | Command                                     |
| --------------------------------- | ------------------ | ------------------------------------------- |
| New database entity/table with ID | ID Type Generator  | `go generate ./sfs/internal/models/ids/...` |
| API request/response types        | Model Generator    | `go generate ./sfs ./kernel`                |
| New HTTP endpoint                 | Endpoint Generator | `go generate ./sfs/sfs/internal/gen/...`    |
| Interface mock for testing        | Mockery            | `go generate ./sfs/...`                     |
| Span generation                   | spanlint           | `go run ./sfs/cmd/spanlint -fix ./...`      |

### When You MUST Use Code Generation

**ID Types** - Use when:

- Creating a new database table with a primary key
- Adding a new domain entity that needs type-safe ID handling
- Any type that will be referenced by ID

**Model Generation** - Use when:

- Creating HTTP request types (structs parsed from JSON body or query params)
- Creating HTTP response types (structs serialized to JSON)
- Any type the frontend/TypeScript needs to consume

**Endpoint Generation** - Use when:

- Adding new API endpoints
- Modifying endpoint paths, methods, or types
- Changing authentication for endpoints

### Quick Commands

```bash
# After adding ID type to ids/gen/main.go:
go generate ./sfs/internal/models/ids/...

# After adding model to models.go with GenStructs registration:
go generate ./sfs ./kernel

# After adding endpoint to endpoints.go:
go generate ./sfs/sfs/internal/gen/...

# Full regeneration (when in doubt):
go generate ./...
```

### Files You Must NEVER Edit Manually

- `sfs/internal/models/generated.go`
- `sfs/internal/models/generated_test.go`
- `sfs/internal/models/ids/*.go` (except the gen/ directory)
- `sfs/sfs/generated_serve_*.go`
- `sfs/sfsclient/generated_*.go`
- `js/packages/sfs-client/src/client/types_generated.ts`
- `*_mocks_test.go` files

---

## Database and Migrations

**Full documentation: [docs/MIGRATIONS.md](docs/MIGRATIONS.md)**

```bash
# Generate a new migration
go run kernel/cmd/dbutils/main.go generate "migration name" --project sfs

# Apply all pending migrations
go run sfs/cmd/sfs/main.go migrate

# Rollback last migration
go run sfs/cmd/sfs/main.go migrate down 1
```

---

## Development Workflow

### Quality Control

**ALWAYS** run these checks before committing:

```bash
# Run linter (required)
go run github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.8.0 run --fix

# Run tests (required)
go test -short ./sfs/... ./kernel/...

# Regenerate generated code (if you modified models or interfaces)
go generate ./...

# Regenerate spans
go run ./sfs/cmd/spanlint -fix ./...
```

**Additional linters** (run these for thorough validation):

```bash
# Check for incorrect log.Print* usage (should use slog)
go run ./kernel/cmd/lint-println ./sfs/... ./kernel/...

# Validate template formatting
go run ./kernel/cmd/templatefmt -verbose -check

# Check JSON struct tags
./kernel/cmd/lint-json/find-json-tags.sh
```

If linting reports whitespace errors like:

```
sfs/internal/logic/file.go:726:2: unnecessary trailing newline (whitespace)
```

Use `cat -n` to see line numbers and compose accurate edit patterns.

### Commit Messages

Format: `package/path: action description`

Think: "This commit modified **[target]** to **[action]**"

**Examples**:

```
sfs/internal/dao: add Connections DAO
sfs/internal/logic: fix NPE in user handler
kernel/pgxplus: update CollectAll signature
js/apps/doc.holiday: add dark mode toggle
```

**Rules**:

- Focus on package/directory, not individual files
- Use lowercase action verbs: add, fix, update, remove, refactor
- Keep descriptions concise (under 50 chars ideally)
- Always refer to https://go.dev/wiki/CommitMessage

---

## AI-Friendly Development Practices

**Generated Files**: Always update through `go generate ./...`. **NEVER** edit generated files directly.

**Code Clarity**:

- Use descriptive variable and function names
- Follow existing patterns in the codebase
- Optimize for clarity and maintainability

---

## Security and Best Practices

- **NEVER** expose or log secrets and keys
- **NEVER** commit sensitive information to the repository
- **DO** use proper input validation and sanitization
- **DO** implement proper authentication and authorization
- **DO** follow security best practices for each language/framework

---

## Behavioral Guidelines

| Guideline                      | Action                                                       |
| ------------------------------ | ------------------------------------------------------------ |
| Before making changes          | **READ** existing code to understand patterns                |
| Before using a library         | **CHECK** go.mod, package.json to confirm it exists          |
| When writing code              | **FOLLOW** existing code style and conventions               |
| When adding functionality      | **EDIT** existing files; don't create new ones unnecessarily |
| When touching code             | **MAINTAIN** consistency with codebase architecture          |
| When making functional changes | **UPDATE** related documentation                             |
| When editing files             | **PRESERVE** existing comments unless they're wrong          |

---

## Configuration and Dependencies

### Configuration Management

- **DO** use environment variables for all configuration
- **NEVER** hardcode sensitive values

### Dependency Management

- **Python**: Use `uv` and virtual environments
- **TypeScript**: Use `pnpm` for package management
- **Go**: Use standard Go modules

### Code Quality Commands

**Go**:

```bash
go run github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.8.0 run --fix
go mod tidy
```

**TypeScript**:

```bash
cd js && pnpm lint && pnpm check-types
```

---

## Documentation Requirements

- Use **English** for all code and documentation
- **Python**: Follow PEP 257 conventions for docstrings
- **Go**: Add godoc comments to exported functions/types (but don't over-comment)
- **TypeScript**: Add typedoc comments to exported code
- **DO** update existing docstrings when modifying code
- **DO** keep existing comments when editing files
