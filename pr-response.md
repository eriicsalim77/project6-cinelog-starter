# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how I used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed save_to_watchlist to add_to_watchlist in services/watchlist_service.py and updated the one call site in routes/watchlist/watchlist.py. Followed the verb_to_noun pattern add_to_collection uses.

**How I verified:** Ran a project-wide grep for save_to_watchlist to catch every call site before renaming. After changes, grepped again for both names to confirm no orphans. Ran pytest tests/ -v — all tests pass.

## Comment 2 — Deduplication
**What I did:** Added a dedup check to add_to_watchlist that mirrors the pattern in add_to_collection. Before inserting a new WatchlistEntry, query for an existing entry with the same user_id and film_id. If one exists, return it instead of creating a duplicate.

**How I verified:** Read add_to_collection first to match the pattern exactly (same query style, same return behavior). Then ran a manual test in a Python shell — called add_to_watchlist twice with the same user and film, queried the table, confirmed only one row exists. Ran pytest tests/ -v — all 4 tests still pass.

## Comment 3 — Missing test
**What I did:**
**How I verified:**

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
