# Role
You are a senior Go 1.22 developer working in this repo.

# Objectives
Add a new authenticated GET endpoint at `GET /api/name` that:
- Returns **200 OK** with the **username** extracted from the Bearer token.
- If the username cannot be obtained, returns **400 Bad Request**.
- Uses our `httpx` helper for known HTTP responses/errors where possible.
- Binds `ctx := r.Context()` once and uses `ctx` throughout (don’t call `r.Context()` again).

# Constraints & Style
- Do not introduce new external deps.
- Keep changes minimal and localized; no breaking changes.
- Follow existing router conventions (gorilla/mux if present).
- Prefer handler signature `func( http.ResponseWriter, r *http.Request)` and pass `ctx` to downstream calls.
- Propagate errors via `httpx` helpers (e.g., `httpx.JSON`, `httpx.BadRequest`, etc.) consistent with how the repo uses them.
- Return JSON with a stable shape: `{"username": "<value>"}` on success; for 400, return a concise `{"error": "generic message"}`.


# Repo discovery first
1. Locate router setup and add route registration for `GET /api/Name`.
2. Find any auth helpers/middleware that extract claims; reuse them.
   - Preferred claim keys: `username`.
3. Inspect `.vscode/launch.json` and ensure we can run the app from there.

# Implementation details
- New handler name: `GetName`.
- Pseudocode:
  ```go
  /// Swagger documentation on request and returns 
  type request struct {
		Id         int
		Name       string
		TemplateId string
		CpRequired int
		Until      time.Time
	}
  type response struct {
		Id         int
		Name       string
		TemplateId string
		CpRequired int
		Until      time.Time
	}
  func GetName(/* deps if needed */) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
      ctx := r.Context()

      // 1) Extract Bearer token (reuse existing middleware/util if present).
      // 2) Parse claims and fetch username (prefer "username").
      // 3) If missing/empty -> httpx.BadRequest(ctx, w, "missing username")
      // 4) Otherwise -> httpx.JSON(ctx, w, http.StatusOK, map[string]string{"username": username})
    }
  }```
