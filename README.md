# neeshdle

Guess the song from as little audio as possible.

You hear 0.5 seconds of a track. Miss, and you get 2 seconds, then 8, then the full clip. Fewer seconds needed means a better round.

**Play it:** [link to live site]

## Why I made it

I've always loved song-guessing games, and when Heardle shut down I wanted an alternative. This is a passion project: my own take on a style of game I love as a music fan, built the way I wanted to play it, with my own playlists and any artist I'm into.

## Ways to play

- **No login.** A small hand-picked pool plus Apple's live top songs chart. Works for anyone.
- **Spotify.** Log in and play from your own playlists (Premium required for playback). Right now this only works for accounts I've added to the app's allowlist. Spotify keeps new apps in Development Mode, limited to a small set of approved users, and has made it much harder to get approved for public access, which is now aimed at established businesses rather than individual developers. If you want to try it, reach out and I'll add you.
- **neeshdle account.** Sign up and your stats follow you across devices instead of living in one browser.

You can also type what you want to play, like "the strokes room on fire" or "2000s pop punk", and it builds a pool of songs for you.

## How the prompt-to-songs feature works

The first version asked the model for a list of songs. It didn't work. Asked for Slayyyter, it returned Dua Lipa and The Weeknd. Real songs, wrong artist.

So the model no longer picks songs at all. It only classifies the request:

- a specific album: `{ "type": "album", "artist": "...", "album": "..." }`
- a specific artist: `{ "type": "artist", "artist": "..." }`
- a vibe or genre: `{ "type": "vibe", "artists": ["...", "..."] }`

Every actual song then comes from the iTunes catalog. The model handles the part it's good at (understanding a sentence) and never the part it's bad at (remembering a tracklist). Responses are validated before use, and the endpoint is rate limited.

## Things that were harder than expected

- **Clip timing.** With the Spotify SDK, every play/pause is a network round trip, so "play 0.5 seconds" was never exactly 0.5 seconds. Timing is measured against the player's own reported position, not just when the JS timer fired.
- **Silent intros.** Some songs start with silence, so the first 0.5 seconds is nothing. Spotify's audio streams are DRM protected, so there's no way to detect this automatically. Those tracks get a manually calibrated start offset.
- **Same song, different IDs.** One song can exist as the single, the album version, and a remaster, each with its own ID. Guesses match on normalized title and artists, not just exact ID.
- **Search coverage.** Apple's free search index lags behind new releases by a month or two. A song that can't be searched can never be guessed, so every track in the curated pool is checked against search before it's added.

## Stack

React + Vite on the front end, Vercel serverless functions for the API, Postgres (Neon) for accounts and stats, Spotify Web Playback SDK and iTunes Search API for audio, and Claude Haiku for the prompt classifier.

Auth is built from scratch: scrypt password hashing, server-side sessions in httpOnly cookies, and constant-time login checks so response timing doesn't reveal whether a username exists.

## Running locally

```bash
npm install
npm run dev
```

Create a `.env.local` with:

```
POSTGRES_URL=
ANTHROPIC_API_KEY=
SESSION_SECRET=
```

For Spotify mode, add your Spotify app's client ID and playlist ID in `src/config.js` and register `http://127.0.0.1:5173/` as a redirect URI in the Spotify dashboard. The database tables are created automatically on the first request.
