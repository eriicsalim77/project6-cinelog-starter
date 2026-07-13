# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how I used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed save_to_watchlist to add_to_watchlist in services/watchlist_service.py and updated the one call site in routes/watchlist/watchlist.py. Followed the verb_to_noun pattern add_to_collection uses.

**How I verified:** Ran a project-wide grep for save_to_watchlist to catch every call site before renaming. After changes, grepped again for both names to confirm no orphans. Ran pytest tests/ -v — all tests pass.

## Comment 2 — Deduplication
**What I did:** Added a dedup check to add_to_watchlist that mirrors the query pattern in add_to_collection. Before inserting a new WatchlistEntry, query for an existing entry with the same user_id and film_id. If one exists, return it instead of creating a duplicate.

The one deliberate difference from add_to_collection: it raises AlreadyInCollectionError, but I return the existing entry. The watchlist route has no error handling wrapper, so raising would 500 the request. Returning the existing entry keeps the endpoint consistent with itself (same 201 status either way) and is safe because the caller can't tell whether the entry was newly created or already existed.

**How I verified:** Read the dedup block in add_to_collection first (lines 47-53) to match the query pattern. Then ran a manual test in a Python shell — called add_to_watchlist twice with the same user and film, queried WatchlistEntry.query.filter_by, confirmed only one row exists. Ran pytest tests/ -v — all 4 tests still pass.

## Comment 3 — Missing test
**What I did:** Created tests/test_watchlist.py with test_add_to_watchlist_nonexistent_film_raises. Mirrored the structure of test_add_to_collection_nonexistent_film_raises exactly — same fixtures, same imports, same pytest.raises pattern.

**How I verified:** Ran pytest tests/test_watchlist.py -v to confirm the new test passes. Ran pytest tests/ -v to confirm all tests still pass (5 total now — 4 collection + 1 watchlist).

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
