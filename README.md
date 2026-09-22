# Georgia Tech Scavenger Hunt App

A React Native/Expo mobile app for running location-based scavenger hunts on Georgia Tech's campus. Users join a hosted session, walk to real-world landmarks ("artifacts"), and check in to earn points on a live leaderboard. Firebase (Realtime Database) for game data, and Firebase Auth for managing accounts. 

Current status: Full lifecycle implemented (sign-up to end game) with listeners for live-data changes. However, still wrapping up adding artifacts + testing before publishing (hopefully by the end of Fall 2026?)

[See demo from Fall 2025 here](https://drive.google.com/file/d/1y3kmOH5Z1rpWbfgBAcqEXlGRrK9hH4I9/view?usp=sharing)

### Game lifecycle flow

1. **Auth** — `WelcomeScreen` → `LogInScreen` / `SignUpScreen` (Firebase Auth)
2. **Join or create a session** — `JoinSessionScreen` (join by session code) or `CreateGameScreen` (host a new hunt)
3. **Lobby / preview** — `GamePreviewScreen` shows session details before the host starts the game
4. **Play** — `HomeScreen` hosts the in-game tab views under `screens/home_screens/`:
   - `LocationsScreen` — map of artifact locations
   - `ArtifactsScreen` — artifacts found/available
   - `LeaderboardScreen` — live participant rankings
   - `SettingsScreen`
5. **Artifact detail** — `ArtifactInfoScreen`, `LocationInfoScreen` for hints, images, and other media per artifact
6. **End of game** — `EndOfGameScreen` / `GameResultScreen` show final scores after a game; `PastResultsScreen` lists a user's history


## Database Schema (Firebase Realtime Database)

This is the area I mostly worked on (in addition to some UI). The schema, constraints, and CRUD rules are defined in [`types/updated_database.ts`](types/updated_database.ts) and implemented by the service layer in [`updated_services/`](updated_services/) (wired into the app via [`contexts/ServiceContext.js`](contexts/ServiceContext.js)). Full CRUD documentation lives in [`types/updated_readme.md`](types/updated_readme.md). The major difference between this and the deprecated "database.ts" is the removal of the Teams entity (we decided to keep gameplay individual, rather than teams-based).

> Note: `types/database.ts` and `services/` are an earlier version of the schema (included `Team`/`Verification` concepts) and are no longer used by the app — `types/updated_database.ts` and `updated_services/` are current.


### Service layer

| Service | Responsibility |
|---|---|
| [`BaseService`](updated_services/BaseService.ts) | Shared Firebase primitives: `getRef`, `exists`, `getData`, `setData`, `removeData`, `subscribe`. All other services extend this. |
| [`UserService`](updated_services/UserService.ts) | User CRUD, profile setters, session membership (`addUserToSession`/`removeUserFromSession`), `currentSession`. |
| [`SessionService`](updated_services/SessionService.ts) | Session CRUD, `gameState` transitions, participant/points management, found-artifact tracking, leaderboard queries (`getSessionLeaderboardEntries`), live subscriptions (`subscribeToSession(s)`). |
| [`ArtifactService`](updated_services/ArtifactService.ts) | Artifact CRUD, coordinates/media setters, session-membership checks before deletion. |

Services are instantiated once with a shared `baseNode` (from `config/config.js`, e.g. for environment isolation) and exposed app-wide through `useServices()` (`contexts/ServiceContext.js`).

## Getting started

```bash
npm install
npm start        # expo start, scan QR code with expo go app
npm run ios      # or android / web
```

Requires Firebase connection; firebase project config lives in [`firebase_config.js`](firebase_config.js). Seed/test data utilities are in [`scripts/`](scripts/) (`upload_artifact_catalog.ts`, `testServices.ts`, `updatedSchemaTests.ts`).
