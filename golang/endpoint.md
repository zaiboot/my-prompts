# Role
You are a Senior Go Software Engineer 1.22 and PGSQL13 developer working in this repo.

# Objectives
- Add a new authenticated GET endpoint at `POST /api/transactions` that will load the transactions of the current user.
- If the database loads an error, returns **424** failed dependency with message: `Try again later`
- Returns **200 OK** with the a list of transactions. Limit it to TOP 10 most recent transactions.
- Returns **204 No content** if there's nothing to return or it contains empty values.
- transaction filter are by username. This is mandatory.
- Uses our `httpx` helper for known HTTP responses/errors where possible.
- Binds `ctx := r.Context()` once and uses `ctx` throughout (don’t call `r.Context()` again).

# Constraints & Style
- Do not introduce new external deps.
- If new functions have unused parameters removed them. Does not apply for existing code.
- Keep changes minimal and localized; no breaking changes.
- Inspect other handlers and look for patterns.
- Follow existing router conventions (gorilla/mux if present).
- Prefer handler signature `func( http.ResponseWriter, r *http.Request)` and pass `ctx` to downstream calls.
- Propagate errors via `httpx` helpers (e.g., `httpx.JSON`, `httpx.BadRequest`, etc.) consistent with how the repo uses them.

# Repo discovery first
1. Locate router setup and add route registration for `POST /api/transactions
2. Find any auth helpers/middleware that extract claims; reuse them.
   - Preferred claim keys: `username`.
3. Inspect `.vscode/launch.json` and ensure we can run the app from there.
4. Review database struct for table `tourney_tickets_store.transaction`

# Implementation details
- New handler name: `<HTTP_VERB><ENDPOINT_URL>`.
- Pseudocode:

  ```go
    type request struct {
		Id         int
		Name       string
		TemplateId string
		AmountRequired int
		Until      time.Time
	}
    type response struct {
		Id         int
		Name       string
		TemplateId string
		balanceRequired int
		Until      time.Time
	}
  /// Swagger documentation on request and returns 
  func GetName(/* deps if needed */) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
      ctx := r.Context()

      // 1) Extract Bearer token (reuse existing middleware/util if present).
      // 2) Parse claims and fetch username (prefer "username").
      // 3) If missing/empty -> httpx.BadRequest(ctx, w, "missing username")
      // 4) query `database.transaction` and get the transactions for username
    }
  }
