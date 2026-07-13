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
**My position:** Default watchlists to public=False (private).

**Reasoning:**
Watchlist content is more personal than collection content. A collection is what you've watched — retrospective, curated, part of your public taste. A watchlist is what you're thinking about watching. That includes late-night curiosity, intimate or romance content, therapy-related films, guilty pleasures, and films a friend recommended that you want to check out privately before deciding if it's for you. Private-by-default respects that intent gap.

CineLog's watchlist is closer to a personal to-do list than a broadcast feed. Netflix's "My List" is private. Goodreads' "Want to Read" defaults to private. Those are the closer analogs for a queue of future intent. A pure social platform like Letterboxd's activity feed can lean public because the whole point is broadcast. The watchlist is not that.

Private-by-default is also a one-way ratchet. If a user wants more visibility, they toggle it and nothing bad happens. If we default to public and a user gets accidentally exposed, that's a trust-breaking moment they can't undo. The safer default is the one where the low-regret path leads to a good outcome.

**Tradeoff acknowledged:**
The social discovery loop is weaker on day 1. Users have to opt in to sharing, and some will never toggle it. CineLog will feel less social by default than it could. The mitigation is a per-item visibility toggle, which is easy to add later and restores the discovery loop without forcing every user through a "wait, that's public??" moment first.

## Comment 5 — Sort order
**My position:** Agree with the reviewer. Changed the default sort from alphabetical to date-added descending (newest first).

**Reasoning:**
Watchlists are a queue of future intent, not a reference catalog. When I open a watchlist to decide what to watch tonight, the film I'm most likely to reach for is the one I added most recently — usually because a friend just recommended it or something caught my eye today. Newest-first surfaces recent intent, which is the highest-signal item for the "what should I watch right now" question.

Alphabetical sort is useful when you're searching a fixed reference set — a library catalog, a phone book. Watchlists aren't reference material. Sorting A-Z buries whatever I added last week under whatever old film starts with A.

Every platform users encounter for this pattern already defaults to recency, not alphabetical. Netflix My List, YouTube Watch Later, Amazon wishlist, Letterboxd's own watchlist — all use recency. Following the convention users already have muscle memory for reduces friction on day 1.

**Engagement with reviewer's point:**
The reviewer said "most users want to see what they added recently" — this matches my own experience and every real-world example I could think of. Alphabetical was a default I picked without thinking through the actual use case. Changing it.

Small extension worth flagging: sort should eventually be a query parameter (?sort=date_added as default, with ?sort=title or ?sort=rating as opt-in). That way the default stays aligned with the common case, but power users who want a specific ordering can get it. Not scoped for this PR — flagging for a follow-up ticket.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
