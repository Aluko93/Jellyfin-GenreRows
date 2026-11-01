# GenreRows 1.4.6 — Smart Collections

Custom genre rows for Jellyfin Web, optional ActorFlix actor discovery and an optional session-protected private-media module. Developed primarily with AI tools, with automated tests and user testing; not an official Jellyfin plugin.

## 1.4.6 actor filmography grouping

Actor profiles now have separate Movies and TV series groups. A series appears once with its own poster and normal Jellyfin details link, even if the actor has hundreds of episode credits. Episodes remain an internal source of credits and are not displayed as individual filmography cards. Matching seasons are combined across credit pages and libraries.

When episode credits cover only some locally available authorized seasons, a compact Season/Seasons label identifies those seasons, including Specials when applicable. If all known locally available seasons are covered, no season list is shown. Series-level cast alone does not claim detailed season coverage. Missing or incomplete season metadata keeps conservative labels; an episode without an authorized parent series is omitted instead of exposing parent metadata. This uses Jellyfin metadata, not an external filmography service.

Genre/year filters, sorting, recent movies, favorites and Back focus remain available. Counts refer to movies plus unique series, not episode totals. Poster nodes remain stable while additional credits arrive. All six module languages include the new labels.

## 1.4.5 artwork and loading fix

Fixes a 1.4.4 metadata-query error that omitted ImageInfos and returned null cover tags. Queries now load the existing Primary artwork, with no user-data join or unrelated fields. Simple rules request only the required result count; actor-AND and all-user unplayed rules keep bounded 128-candidate batches.

Home paints the first 12 cards and continues in batches of at most 12. Covers are requested close to the viewport with at most four active downloads; existing decoded images and all displayed cards survive refresh while the replacement page set is still loading. Card geometry matches the ordinary genre rows. Horizontal remote moves use adjacent cards without scanning the geometry of the entire gallery.

The optional Home modules are ordered after the genre rows: Smart Collections, Actors, Adult. This order is maintained when sections appear asynchronously. Existing collection definitions and all other plugin settings are preserved.

## Smart Collections

An optional module inside GenreRows, disabled by default. Enable it in Dashboard > Genre Rows — Smart Collections, add collections, select rules and save. Collections are dynamic Home rows and plugin-owned paged views. Clicking a movie or episode opens its normal Jellyfin details page. These are read-only views, not persistent Jellyfin box sets, and they do not modify metadata or playback history.

Each collection can select authorized libraries (an empty selection means all libraries available to the viewing user), Movie/Series/Episode content, actors, one genre, an inclusive minimum/maximum production year, minimum community rating, watched state, audio language and 4K. All conditions must match. Up to eight selected actors are combined with AND: the title must credit every selected actor as Actor or GuestStar, not only one of them. Actor search uses the existing local permission-scoped ActorFlix catalog and does not require its Home gallery to be enabled. A cold actor catalog can initially return no suggestions; retry after it finishes.

Use the genre suggestions from the actual Jellyfin library metadata. Missing metadata does not match a restricted year/rating/audio/4K rule. Audio/4K apply to movies or episodes, not series-folder records; select Episodes for technical filters on television content. Audio uses existing Jellyfin language tags and 4K uses Jellyfin's indexed Is4K filter; no media reprobe or transcoding is performed.

Examples:

- Leonardo DiCaprio AND Brad Pitt: add both actors from the local picker. They must exist in your accessible library credits.
- Science-fiction movies released after 2015: select the actual science-fiction genre and set From year to 2016.
- Highly rated films: set Minimum rating to 7.5 (inclusive).
- Not watched by me: uses the viewing user's Jellyfin played flag.
- Never watched by any current user: excludes titles marked played or with a positive existing play count for any currently existing server user. It does not recover deleted-user history or information removed from Jellyfin. No per-user viewing records are sent to the browser. This aggregate rule is configured by the administrator and may indirectly reveal that a title has been watched somewhere on the server.
- Italian 4K films: choose Italian audio and 4K only.

Collections refresh their results and configuration every minute while Home is visible; each row also has a manual Refresh button. Newly indexed films and updated ratings/play flags are evaluated on the next query, without rebuilding another database. Preview tests the first matching titles without saving. Up to 20 collections and 1–60 Home cards per collection are supported. Full collection views load additional results on request. Sorting supports recent additions, title, year and rating. Rules referring to inaccessible libraries return no items; an empty authorized scope never becomes a global query.

Each server request examines at most five batches of 128 metadata records; sparse actor-AND or global-unwatched rules continue via cursors. Home yields between continuation requests, and the full view has an explicit Load more control. Broad rules on large libraries or servers with many user profiles can take longer. Pagination reads live metadata: additions/removals during paging can shift offsets; the client deduplicates item IDs and Refresh starts a new view.

Configuration, actor selection and unsaved previews require Jellyfin administrator authorization. Collection catalog/results require authenticated Jellyfin access and recheck the current user's library scope. Responses are no-store. The module uses existing indexed normal libraries only; the external Adult directory is not scanned or registered. It makes no external service requests and introduces no playback events. Six interface languages are included.

## ActorFlix

Enable the existing Actors switch in Dashboard > Genre Rows — Actors. The Home row defaults to Discover. The first 30 cards appear without waiting for the complete catalog. Metadata loads in small background pages and portraits are requested only near the visible area. Background updates reuse existing card/image nodes, preserve focus/scroll and do not reshuffle already exposed actors. Explicit sorting/search/filtering recalculates order; the next visit uses all currently cached Discover scores. This keeps browsing stable when popularity data or a new cache revision arrives. A–Z, Z–A and explicit name search remain available. The search keyboard opens only on click/OK. Actor cards reuse Jellyfin's existing portraits, with a bottom gradient, a soft theme-colored light underneath and a 250 ms 25% enlargement on hover/focus. Reserved slots prevent overlap or layout shifts; no permanent actor outline is used. Reduced-motion preferences disable the transition. Mouse, keyboard, remote-direction keys and narrow screens are supported in the web interface.

Click an actor for a dedicated overlay with their existing Jellyfin photo, name, short/expandable biography, favorite button, recently added movies, separate groups of accessible movies and TV series, and genre/year filters. Sort titles by name, newest/oldest year, stored Jellyfin community rating or recently added. Missing years/ratings are placed last in their respective sorts. Movie cards open normal Jellyfin details. Back/Escape closes the profile and restores gallery focus. Recently added means the latest DateCreated values in accessible movies; it is not a notification service. Missing biographies remain unavailable instead of triggering external metadata requests.

Favorites use Jellyfin's native user-data service and are separated by authenticated account. The Home Favorites button filters the actor gallery. Profile/credits/favorites endpoints derive the user from authentication and verify accessible actors and library IDs.

## Discover ranking

The score is 70% normalized TMDB popularity, 20% normalized number of distinct movies in the user's accessible libraries, and 10% deterministic daily variation. Counts use log scaling to prevent huge libraries from dominating. Duplicate credits do not count twice; series and episodes do not inflate the movie count. Variation is stable for one UTC day and Jellyfin user. These are GenreRows design choices, not Netflix's algorithm, and the same most-famous actor may still rank highly on consecutive days.

When no positive TMDB score is available, Discover uses movie count and cast-order prominence as a local approximation. The interface labels this library-only state. No hardcoded celebrity list is used. Actor records are still derived only from accessible real titles, excluding orphan people, inaccessible libraries, missing/virtual items and audio-only credits. Actor metadata snapshots are permission-scoped and persist on the server.

## TMDB source and cache

TMDB publishes public daily person-ID exports containing popularity. No API key is required by this export service at the time tested. GenreRows downloads the normal person export from the fixed HTTPS files.tmdb.org host, caches it on the server for one week and retains numeric scores only for locally requested TMDB person IDs. There is one export download per cache refresh, not a request for each actor. Library titles, user identities, search terms and selected actor IDs are not sent to TMDB.

The first refresh runs in the background: the gallery remains usable with local ranking, then receives TMDB scores in the background without moving the portraits already shown. Reapply Discover or revisit the Home row to rank the visible batch with the latest scores. The tested 2026-10-29 export was 78,098,877 compressed bytes (about 75 MiB); size can change. A cold download/parsing pass can take time. Disable TMDB in Dashboard > Genre Rows — Actors for local-only operation. Offline/errors retain cached scores or fall back locally, with retry backoff. Match quality depends on existing TMDB person IDs in Jellyfin.

Data source: https://developer.themoviedb.org/docs/daily-id-exports
Popularity explanation: https://developer.themoviedb.org/docs/popularity-and-trending
The About section contains TMDB's approved unmodified logo and attribution notice. This product uses the TMDB API but is not endorsed or certified by TMDB. TMDB data is used only by ordinary ActorFlix libraries, not by the private-media module.

## Existing functionality retained

Ordinary genre rows, existing settings, the private-media player, password/session protections, user visibility, TV navigation, category covers and generated previews are retained. Adult, Smart Collections, ordinary rows, plugin configuration and integration files match 1.4.5 byte for byte. The 1.4.6 production changes are confined to the ActorFlix title service and profile renderer. The ordinary renderer now favors different titles in the leading 15 positions across earlier rows of the same media type, including across custom groups. It requests at most 100 recent candidates per selected library/genre and preserves the configured row length. Repeated titles remain later in the row; when there are insufficient fresh candidates they can still occupy leading positions. Rows retain configured group/genre order even if requests finish out of order. Configuration and the number of displayed cards are unchanged. Private video cover matching remains: My film.mp4 plus My film.jpg/png/jpeg/webp in the same folder. Existing private data remains external to Jellyfin libraries.

## Requirements and tested scope

This installer targets Jellyfin Server 12.2.0 / .NET 10. Local tests used the official 12.2.0 packages, ASP.NET HTTP fixtures, Chromium desktop/narrow viewports and synthetic FFmpeg media. The preceding 1.4.1 was reported working on the user's 12.2.0 Podman server. This 1.4.6 has not been installed on production by the developer. A full Jellyfin metadata database and physical TV were not available for this release; other Jellyfin versions are not claimed compatible by this package. See VALIDATION.md.

## Installation and upgrade

Back up your existing plugin folder and configuration. Extract the complete installer archive. It contains package/GenreRows_1.4.6.0/Jellyfin.Plugin.GenreRows.dll, meta.json and SHA256SUMS. For a manual installation, stop Jellyfin, retain/remove the previous GenreRows folder from the active plugins directory, copy the new package directory into your plugins directory and start Jellyfin. Plugin ID remains f6a47162-497f-472f-aadf-323c766ac182; install only one GenreRows version at a time. Existing configuration is preserved.

For an existing Podman installation, run bash update-podman.sh CONTAINER from the extracted directory. It verifies version/hashes, retains the previous plugin outside plugins and prints the exact rollback command. For a first Podman installation use install-podman.sh. Restart scripts allow 30 seconds for a graceful stop before Podman applies its fallback. The included scripts assume the conventional /config container layout. See the actual script output if custom paths differ. Reload the web client after restart.

## Rollback and troubleshooting

The update script prints bash restore-previous-podman.sh CONTAINER BACKUP_PATH. Retain that exact path. Do not guess a backup directory. Manual rollback: stop Jellyfin, move the new plugin out of active plugins, restore your retained plugin folder and restart. Configuration XML is not deleted by these scripts.

An initially library-only Discover label is expected during the first export refresh or with missing provider IDs/network access. Actor metadata can be reloaded using Dashboard > Genre Rows — Actors > Reload actors. For missing biographies/photos, check the existing Jellyfin metadata. For a missing private cover, verify identical video/image stems, server-visible files and the installed DLL checksum. Standard/Extended versions sharing this plugin ID cannot be active together.

## Build and license

Source archive: src/, tests/, NuGet.Config, build.sh, scripts and documentation; no build caches, test videos or credentials. Build src/GenreRows.csproj in Release with .NET 10. Distributed plugin is GPL-3.0; preserve notices and include corresponding source. hls.js retains its Apache-2.0 license. TMDB logo/data retain their respective owners' terms and attribution. See LICENSE and THIRD-PARTY-NOTICES.txt.
