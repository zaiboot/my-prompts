# Role
You are a senior Go 1.22 engineer working in this repo.

# Goal
Migrate API documentation to the swaggo toolchain with minimal code churn.

# What to change
- Primary edits must be in: internal/api/docs.go  (project metadata, @BasePath, @Security, go:generate).
- Optional 1–2 line addition in the router to serve the Swagger UI.
- Only add doc comments above handlers if absolutely necessary; prefer leaving handlers untouched in this pass.

# Constraints & Style
- Keep behavior identical. No refactors, no renames, no new runtime deps beyond swaggo’s canonical ones.
- Follow existing gorilla/mux router patterns.
- Prefer non-invasive changes: comments 
- Do not use `init()` functions.
- If auth middleware enforces Bearer tokens, document it but do not modify middleware.

# Allowed dependencies (and only these)
- Dev tool: github.com/swaggo/swag/cmd/swag (via go:generate, not imported at runtime)
- UI handler: github.com/swaggo/http-swagger (one import in router if UI is exposed)
- Generated docs package lives at: internal/api/docs (committed to repo)

# Repo Discovery (do this first)
1) Locate the top-level mux router and any subrouters (PathPrefix, Methods).
2) Identify API base path (e.g., /api or /api/v1) and note it for @BasePath.
3) Detect if Bearer/JWT auth is enforced at router or middleware level; if yes, we’ll declare @securityDefinitions and use @Security BearerAuth in summaries later (no handler edits in this pass).

# Implementation Plan
1) Create/Update internal/api/docs.go:
   - Package comment with swaggo metadata:
     - @title, @version, @description, @schemes, @BasePath
     - @securityDefinitions.apikey BearerAuth (in: header, name: Authorization)
     - Optional contact/license if the repo has them
   - `//go:generate swag init -g internal/api/docs.go -o internal/api/docs`
   - In code, import the generated package as `swagdocs "your/module/internal/api/docs"`
   - `init()` sets `swagdocs.SwaggerInfo.BasePath` to the detected base path; set Title/Version if needed.

2) Router (optional, keep to 1–2 lines):
   - Add: `import httpSwagger "github.com/swaggo/http-swagger"`
   - Mount UI: `r.PathPrefix("/swagger/").Handler(httpSwagger.WrapHandler)`
   - Do not alter existing routes.

3) Do not mass-edit handlers in this pass.
   - If a handler already has useful comments, leave them.
   - If absolutely necessary to demonstrate a single endpoint, add one minimal @Summary/@Tags/@Router block to one handler as an example only.

# Acceptance Criteria
- `go generate ./...` from repo root produces `internal/api/docs` with swagger.json/yaml.
- `go build` succeeds with no behavior changes.
- If the UI route is added, GET /swagger/index.html loads the explorer and lists endpoints (even if sparse).
- Minimal diff: mainly `internal/api/docs.go`; at most a couple of lines in router; go.mod updated only to include http-swagger if UI is enabled.

# Notes
- @Tags should reflect business domains (e.g., "tickets", "auth", "bootstrap"), not technical layers.
- If a global error envelope type exists (e.g., httpx.Error), we will reference it later in handler comments; not in this pass.
