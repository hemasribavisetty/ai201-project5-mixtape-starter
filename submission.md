# Project 5: Mixtape Bug Hunt

## AI Usage

## Codebase Map

### Main Files and Roles

- `app.py`: Creates the Flask application, configures the database, and registers the route blueprints.
- `models.py`: Defines the SQLAlchemy models used by the app, including users, songs, playlists, playlist-song relationships, notifications, and tags.
- `seed_data.py`: Populates the database with test users, songs, playlists, tags, and sample activity.
- `routes/songs.py`: Handles song-related HTTP endpoints such as sharing songs, searching songs, and rating songs.
- `routes/playlists.py`: Handles playlist creation and playlist song management routes.
- `routes/users.py`: Handles user profile, streak, and notification routes.
- `routes/feed.py`: Handles feed-related routes such as friends listening now and recent activity.
- `services/streak_service.py`: Contains listening streak calculation logic.
- `services/feed_service.py`: Contains logic for building the friends listening now feed.
- `services/search_service.py`: Contains song search logic.
- `services/notification_service.py`: Contains notification creation and retrieval logic.
- `services/playlist_service.py`: Contains playlist song retrieval logic.

### Data Flow Example: Rating a Song

A user rates a song through a route in `routes/songs.py`. The route reads the request data, finds the song, and delegates notification-related behavior to `services/notification_service.py`. The service layer creates notification records when needed, and those records are stored using the SQLAlchemy models from `models.py`.

### Pattern I Noticed

The app is organized using a route/service pattern. The route files handle HTTP request parsing and response formatting, while the service files contain the main business logic. The known bugs are concentrated in the `services/` layer, so debugging should start by tracing from the route to the related service function.

## Bug Fix 1

## Bug Fix 2

## Bug Fix 3

## Git Log Screenshot