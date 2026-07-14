# Type Faster

A typing test app with personalized practice drills and live multiplayer racing —
built as a React + Vite web app and shipped natively to iOS and Android via Capacitor.

> This repository is a portfolio case study. The app is closed-source; screenshots,
> architecture notes, and engineering write-ups live here so the work can be reviewed
> without exposing the codebase.

## Screenshots

<p align="center">
  <img src="screenshots/01-onboarding.png" width="250" />
  <img src="screenshots/02-live-typing.png" width="250" />
  <img src="screenshots/03-results-smartcoach.png" width="250" />
</p>
<p align="center">
  <img src="screenshots/04-race-hub.png" width="250" />
  <img src="screenshots/05-race-matchmaking.png" width="250" />
</p>

## Tech Stack

- **React 18, Vite, Tailwind CSS** — the web client
- **Capacitor** — native iOS and Android wrappers over the same web codebase, with platform-specific release signing
- **Supabase** — Postgres + Row Level Security, Realtime channels, and Edge Functions as the backend
- **Security-definer RPCs** — `get_leaderboard`, `get_wpm_percentile`, `username_available` run server-side so anonymous clients never query raw tables directly
- **Realtime presence + broadcast** — race rooms, matchmaking queues, and live opponent positions all run over Supabase Realtime channels
- **i18n from the ground up** — 6 typing languages including Arabic, with a fully separate app-UI localization layer (EN/AR) that mirrors layout, not just text, in RTL

## Architecture

```mermaid
graph TD
    subgraph Client
        Test[Typing Test]
        Coach[Smart Coach]
        RaceHub[Race Hub]
        Results[Results + Leaderboard]
    end

    subgraph Supabase
        DB[(Postgres: profiles, test_results, friends, race_invites)]
        RPC[Security-definer RPCs]
        Realtime[Realtime: race:room, race:queue]
        EdgeFn[Edge Function: login-with-identifier]
    end

    Test --> Coach
    Test --> Results
    Results --> RPC
    RPC --> DB

    RaceHub --> Realtime
    Realtime --> DB
    RaceHub -.presence + bots.-> RaceHub

    Login[Username Login] --> EdgeFn
    EdgeFn --> DB
```

**Race matchmaking** elects a leader per room over Realtime presence, then fills empty
seats with skill-matched bots that join the room the same way a real player would —
same presence events, same avatar broadcast — so a race never sits waiting for
players who aren't coming.
