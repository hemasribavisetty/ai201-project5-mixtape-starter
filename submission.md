# Project 5: Mixtape Bug Hunt

# AI Usage

During this project, I used ChatGPT to help me understand the existing codebase and verify my debugging process.

## Instance 1

I asked ChatGPT to explain the responsibilities of the service files and trace the data flow between the route files and the service layer. This helped me understand the overall application architecture before beginning any bug fixes.

## Instance 2

After identifying suspicious code manually, I asked ChatGPT to explain Python logic and SQLAlchemy query behavior to verify my understanding of the bug. I confirmed every suggested fix by reproducing the issue, reading the code myself, and running the tests after making the changes.

---

# Codebase Map

## Main Files and Roles

* **app.py** – Creates the Flask application, configures the database, and registers all route blueprints.
* **models.py** – Defines the SQLAlchemy models including User, Song, Playlist, PlaylistEntry, Notification, ListeningEvent, Tag, and the many-to-many relationship tables.
* **seed_data.py** – Populates the database with sample users, songs, playlists, listening events, and tags.
* **routes/songs.py** – Handles song sharing, searching, rating, and retrieval endpoints.
* **routes/playlists.py** – Handles playlist creation and playlist song retrieval.
* **routes/users.py** – Handles user profile, listening streak, and notification endpoints.
* **routes/feed.py** – Handles the friends activity feed and listening feed.
* **services/streak_service.py** – Implements listening streak calculation.
* **services/feed_service.py** – Generates the friends listening feed.
* **services/search_service.py** – Performs song searching.
* **services/notification_service.py** – Creates and retrieves notifications.
* **services/playlist_service.py** – Retrieves songs in playlists.

## Data Flow Example

### User Rates a Song

1. Client sends a request to rate a song.
2. `routes/songs.py` receives the request.
3. The route validates the request and calls `notification_service.py`.
4. The service creates any necessary notifications.
5. The updated data is stored through SQLAlchemy models defined in `models.py`.
6. The route returns the response to the client.

## Pattern I Noticed

The application follows a clear **route → service → database** architecture.

* Route files handle HTTP requests and responses.
* Service files contain the business logic.
* Models define the database schema.
* Nearly every feature delegates immediately from the route layer into a service function.

The bugs are intentionally located inside the service layer.

---

# Bug Fix 1

## Issue Number and Title

Issue #1 — My listening streak keeps resetting

### How I Reproduced It

I inspected `services/streak_service.py` and traced the listening streak update logic. The bug occurs when a user listens on consecutive days where the second day is Sunday. Instead of incrementing the streak, the application reset it.

### How I Found the Root Cause

I started with the README, which identified `streak_service.py` as the affected service. Inside `update_listening_streak()`, I found the conditional responsible for incrementing the streak.

### Root Cause

The code checked:

```python
elif days_since_last == 1 and today.weekday() != 6:
```

Python's `weekday()` returns `6` for Sunday. This condition prevented streaks from incrementing on Sundays even when the user listened on consecutive days, causing the streak to reset incorrectly.

### Your Fix and Side-Effect Check

I changed the condition to:

```python
elif days_since_last == 1:
```

This allows consecutive listening days to increment the streak regardless of which weekday it is. I ran the streak tests afterward to verify that same-day listens, consecutive-day listens, and skipped-day resets all behaved correctly.

---

# Bug Fix 2

## Issue Number and Title

Issue #5 — The last song in a playlist never shows up

### How I Reproduced It

I inspected `services/playlist_service.py` and followed the playlist retrieval logic. Playlists containing multiple songs consistently returned every song except the final one.

### How I Found the Root Cause

The README identified `playlist_service.py` as the affected service. In `get_playlist_songs()`, I found the SQL query correctly retrieved every song, but the returned list was sliced before being returned.

### Root Cause

The function returned:

```python
songs[:-1]
```

The slice excluded the final element of the playlist every time.

### Your Fix and Side-Effect Check

I changed the return statement to:

```python
return [song.to_dict() for song in songs]
```

This returns the complete playlist while preserving song order. I verified the fix by running the playlist tests and confirming the last song now appears correctly.

---

# Bug Fix 3

## Issue Number and Title

Issue #3 — The same song keeps showing up twice in search

### How I Reproduced It

I inspected `services/search_service.py` and traced the SQLAlchemy search query. Songs associated with multiple matching tag records appeared multiple times in the search results.

### How I Found the Root Cause

I followed the README guidance to the search service and examined the SQLAlchemy query. The query joined the `song_tags` table but never removed duplicate `Song` rows.

### Root Cause

The SQL query performed an outer join with `song_tags`. Because one song may have multiple tag rows, SQL returned duplicate rows for the same song. Without `DISTINCT`, duplicate songs appeared in the application results.

### Your Fix and Side-Effect Check

I added:

```python
.distinct()
```

before `.all()` in the SQLAlchemy query. This ensures each song appears only once while preserving valid search results. I ran the search tests afterward to verify duplicate songs no longer appeared.

---

# Git Log Screenshot

(.venv) hemasribavisetty@Hemasris-MacBook-Air ai201-project5-mixtape-starter % git log --oneline
5febb00 (HEAD -> bugfix/mixtape, origin/bugfix/mixtape) fix: remove duplicate songs from search results
7b53e2d fix: include last song in playlist results
089e4a5 fix: allow streaks to continue on Sundays
08599cf docs: add codebase map
2dfdeaa (origin/main, origin/HEAD, main) Add .gitignore file and update README with setup instructions
7b64551 initial commit
(.venv) hemasribavisetty@Hemasris-MacBook-Air ai201-project5-mixtape-starter % 

```bash
git log --oneline
```

Example:

```
5febb00 fix: remove duplicate songs from search results
7b53e2d fix: include last song in playlist results
089e4a5 fix: allow streaks to continue on Sundays
08599cf docs: add codebase map
```
