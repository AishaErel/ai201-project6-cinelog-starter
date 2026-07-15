# PR Response Doc — CineLog Watchlist Feature

## AI Usage

I used AI to understand the existing implementation in `services/collection_service.py` before adding the deduplication logic to `add_to_watchlist()`. I also asked AI to review my draft responses for Comments 4 and 5 to identify any tradeoffs or counterarguments I had not addressed. It highlighted privacy considerations for the default visibility setting and the consistency tradeoff between alphabetical and date-added sorting, so I revised my responses to acknowledge those points.
---

## Comment 1 — Rename

**What I did:**
I renamed the requested identifiers consistently across the watchlist feature. Before renaming, I searched the project for all references to the original names so every call site, import, and usage was updated together instead of changing only the function definition.

**How I verified:**
I searched the project for the old identifier to confirm there were no remaining references. I also ran the relevant tests after the rename to ensure imports and function calls still worked correctly.

---

## Comment 2 — Deduplication

**What I did:**
I added a deduplication check to `add_to_watchlist()` using the same pattern as `add_to_collection()`. After confirming the film exists, the function now queries for an existing `WatchlistEntry` with the same `user_id` and `film_id`. If one exists, it raises `AlreadyInWatchlistError` instead of creating a duplicate entry.

**How I verified:**
I compared my implementation directly with `add_to_collection()` in `services/collection_service.py` to make sure it followed the same logic and error-handling pattern. I also verified that the existing behavior for adding new films remained unchanged.

---

## Comment 3 — Missing test

**What I did:**
I created `tests/test_watchlist.py` and added a test for attempting to add a nonexistent film to the watchlist. I used `test_add_to_collection_nonexistent_film_raises()` in `tests/test_collection.py` as my model so the fixture setup, assertion style, and exception handling matched the project's existing test suite.

**How I verified:**
I ran:

```bash
pytest tests/test_watchlist.py -v
```

and confirmed the new test passed successfully.

---

## Comment 4 — Default visibility
**My position:**
I would keep `public=True` as the default for watchlist entries.

**Reasoning:**
I am optimizing for the expected social behavior of CineLog: users add films to their watchlist partly so others can see what they are interested in watching. Since the collection feature already exposes user film activity, a public watchlist makes the feature easier to discover and more useful without requiring users to change settings every time they add a film.

**Tradeoff acknowledged:**
The tradeoff is privacy. Some users may not want every watchlist item visible by default, especially if the film is personal, embarrassing, or only meant as a private reminder. A `public=False` default would be safer from a privacy-first perspective. However, for this project I think `public=True` better matches the app’s social sharing behavior, as long as users can later change visibility.

## Comment 5 — Sort order
**My position:**
I would keep alphabetical sorting by film title for the watchlist.

**Reasoning:**
I am optimizing for browsing and finding saved films later. A watchlist can grow over time, and alphabetical sorting makes it predictable when a user is scanning for a specific title. This is different from the collection, where date-added sorting makes sense because the user may care about what they watched most recently.

**Engagement with reviewer's point:**
I understand the maintainer’s reasoning for preferring date-added order: it highlights the newest additions and matches the pattern used by `get_collection()`. That consistency is valuable. However, I think the watchlist has a different user behavior than the collection. The collection is more like a viewing history, while the watchlist is more like a saved list to browse later. For that reason, I kept alphabetical sorting rather than changing it to date-added.

## Comment 6 — Rebase

**What conflicted:**
I encountered merge conflicts where my branch and the updated main branch modified overlapping files.

**How I resolved it:**
I carefully reviewed each conflict, kept the intended watchlist functionality, and incorporated the upstream changes where appropriate before completing the rebase.

**How I verified no conflict remains:**
After the rebase completed successfully, I reran the relevant tests and confirmed the project built and the watchlist functionality continued to work as expected.

---

## Log screenshot
<img width="835" height="100" alt="Screenshot 2026-07-14 at 9 28 39 PM" src="https://github.com/user-attachments/assets/26ea1d9a-6981-46eb-b329-b819399817ac" />




## PR Description

This pull request adds the watchlist feature for CineLog. Users can add films to a personal watchlist, retrieve their watchlist, and duplicate entries are prevented.

### Design decisions

- **Default visibility:** Watchlist entries default to `public=True` because CineLog is intended to be a social film-tracking platform where users can share what they plan to watch. Users can still change visibility later if needed.
- **Sort order:** The watchlist is sorted alphabetically by title rather than by date added. This makes it easier for users to browse and find saved films, while the collection feature remains sorted by recency because it represents viewing history.

### Manual testing

1. Add a valid film to the watchlist and confirm a `WatchlistEntry` is created.
2. Add the same film again and confirm an `AlreadyInWatchlistError` is raised.
3. Attempt to add a nonexistent film ID and confirm `FilmNotFoundError` is raised.
4. Retrieve the watchlist and confirm the expected films are returned in alphabetical order with the `date_added` and `public` fields.
