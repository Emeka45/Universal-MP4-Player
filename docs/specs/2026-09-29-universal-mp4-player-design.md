# Universal MP4 Player Design Specification

**Date:** 2026-09-29  
**Repository:** Emeka45/Universal-MP4-Player

## Goal

Replace the current browser-based prototype with a production-grade native Android Universal MP4 Player that provides a beautiful, fast, local-first video library while supporting network playback and a modular compatibility path for additional media formats and codecs.

## Product Scope

The application is a full Universal Video Center branded as Universal MP4 Player. MP4 is the primary identity, but the player must not artificially restrict playback to MP4 when the underlying Android/Media3/compatibility stack can safely play other common video containers and codecs.

Core capabilities:
- Device video discovery through MediaStore.
- User-selected folders/files through Storage Access Framework.
- Search, filtering, sorting, grid/list library views.
- Folders, favorites, playlists, history and recently added/played media.
- Resume positions and watched state.
- Native video playback with Media3.
- Network URL playback and supported HLS/DASH streams.
- Audio-track and subtitle selection.
- External subtitle files and subtitle timing/style controls where supported.
- Playback speed, seeking, aspect-ratio controls, fullscreen, rotation, screen lock and gestures.
- Picture-in-Picture.
- Android MediaSession notification/lock-screen controls.
- Queue, previous/next, bookmarks and A-B repeat.
- Sleep timer.
- Share/open-with and Android-safe file operations.
- Clear diagnostics for unsupported or corrupted media.
- Accessibility and low-end-device optimization.

## Architecture

Use a native Kotlin Android application with Jetpack Compose for UI.

Playback architecture is Media3-first:
1. Prefer Android/device-supported decoders through Media3.
2. Use Media3 software playback facilities where applicable.
3. Route genuinely unsupported media through an isolated compatibility module only when a compatible decoder is available and legally distributable.
4. Report unsupported/corrupt media with an actionable explanation instead of a generic failure.

Use:
- MediaStore for discoverable local media.
- Storage Access Framework for explicit user-selected locations/files.
- Room for durable application metadata.
- Media3 MediaSession for playback state and system controls.
- WorkManager for bounded background indexing/thumbnail work.
- Android Picture-in-Picture APIs.
- Modular repositories/use cases so playback, library indexing, metadata and UI remain independently testable.

The existing web prototype is reference material only. It should not constrain the native architecture.

## Data Model

Persist:
- Video identity and source URI.
- Display name and normalized searchable metadata.
- Duration, size, MIME/container information and dimensions when available.
- Last played timestamp.
- Resume position.
- Watched state.
- Favorite state.
- Playlist membership.
- Bookmarks.
- User-selected folder/source information where applicable.

Do not persist raw video contents.

Use content URIs rather than assuming filesystem paths. SAF permissions must be persisted when Android grants durable access.

## User Experience

### Home
Show Continue Watching, Recently Played, Recently Added, Favorites and playlists, with an empty state that immediately explains how to add or discover videos.

### Library
Provide Videos and Folders views, global search, sorting and filtering. Thumbnail loading must be lazy and bounded.

### Player
Provide a cinematic player surface with:
- play/pause
- previous/next
- seek backward/forward
- scrubber
- volume
- fullscreen
- rotation/aspect controls
- playback speed
- subtitles
- audio tracks
- queue
- lock controls
- PiP
- bookmark
- sleep timer
- A-B repeat where supported

Touch gestures must have discoverable feedback and must not interfere with essential system gestures.

### Design Language
Use the Universal U identity, premium dark-first visual language, responsive Compose layouts, clear hierarchy, large touch targets and restrained animation. Visual effects must degrade gracefully on low-end devices.

## Performance and Compatibility

Target practical usability on Android 12 Go-class hardware.

Requirements:
- no full-library thumbnail preloading into memory;
- bounded image/thumbnail caches;
- paged/lazy library rendering;
- bounded background indexing;
- cancellation of stale scans;
- no permanent background service solely for library indexing;
- avoid loading complete video files into memory;
- graceful behavior under memory pressure;
- avoid blocking the main thread on metadata or database operations.

The application must distinguish:
- unsupported container/codec;
- malformed/corrupt media;
- inaccessible/expired URI permission;
- transient network failure;
- invalid network URL;
- DRM-protected content that the selected playback path cannot legally/decode.

## Privacy

Local playback must work without account creation. Video files must not be uploaded merely for indexing, thumbnails or playback. Network playback must be initiated by the user or an explicitly configured source.

## Error Handling

Every playback failure should resolve to a user-understandable state:
- explain what failed when determinable;
- provide retry for transient failures;
- offer “open with another app” where appropriate;
- provide file/source details when useful;
- never crash because a single media item is malformed.

## Testing

Test at unit, integration and UI levels.

Minimum coverage:
- media discovery and deduplication;
- metadata persistence;
- resume position persistence;
- favorites/playlists/history;
- search/filter/sort;
- SAF URI permissions;
- unsupported/corrupt media handling;
- playback state transitions;
- previous/next queue behavior;
- subtitle/audio selection state;
- PiP entry/exit;
- configuration change and process recreation;
- low-memory behavior;
- accessibility semantics;
- network playback error states.

Release verification must build the debug and release variants, run automated tests, and inspect the generated APK/AAB artifacts.

## Existing Repository Context

The repository currently contains a small web prototype. Recent commits include:
- Create Universal MP4 Player foundation.
- Add premium responsive MP4 Player design.
- Add MP4 library and playback engine.

The prototype currently uses browser File objects, object URLs and an HTML video element. The final native application intentionally replaces this architecture rather than incrementally expanding it.

## Success Criteria

A tester can install the application on a supported Android phone, discover local videos without manually importing each file, search and organize the library, open a video, control playback naturally, resume playback later, use subtitles/audio tracks when available, use PiP/system media controls, create/use playlists and favorites, play supported network sources, and receive understandable feedback for media the playback stack cannot handle.

The application must remain responsive on low-end hardware and must not depend on a cloud account for ordinary local video playback.
