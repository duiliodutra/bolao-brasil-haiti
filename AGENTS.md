# Bolão Brasil

A single-file static web app: a Brazilian-Portuguese football betting pool ("bolão") for the 2026 World Cup. Users submit a predicted final score ("palpite") for a match; predictions render in real time with a running prize pool. An admin panel (Google sign-in, restricted to hardcoded emails) sets the official result, locks/unlocks betting, and edits/deletes predictions.

The entire app is `index.html` (inline CSS + vanilla JS). There is **no build step, no package manager, and no dependency manifest**. The backend is **Firebase (Cloud Firestore + Authentication)**, loaded via CDN and hardcoded in `index.html`.

## Cursor Cloud specific instructions

### Running the app
- There is nothing to build or compile. Serve the static file from the repo root, e.g. `python3 -m http.server 8000` then open `http://localhost:8000`. (`python3` is preinstalled; no install needed.)
- The page requires internet access to load the Firebase SDK from `gstatic.com` and flag images from `flagcdn.com`.

### Firebase backend — production data caution
- `index.html` connects directly to the **live production** Firebase project `bolao-brasil-haiti-ddf6f` (Firestore + Auth). There is no staging project and no emulator wiring in the committed code.
- **Do NOT submit test predictions against the production backend** — writes land in `palpites` and pollute the real betting pool, and you cannot delete them without an admin Google account (`ADMINS` in `index.html`).
- To test the write/realtime flow safely, use the **Firebase Local Emulator Suite** instead of production (see below). The admin panel needs a real Google login, so it cannot be fully exercised against the emulator without code changes.

### Safe end-to-end testing with the Firebase emulator
The `firebase` CLI is installed on `PATH` (user prefix `~/.npm-global/bin`; also added to `~/.bashrc`). Java is preinstalled (required by the Firestore emulator). To test without touching production:
1. Copy `index.html` to a scratch dir (do **not** modify the committed file) and, right after `const auth = firebase.auth();`, add:
   - `db.useEmulator('127.0.0.1', 8080);`
   - `auth.useEmulator('http://127.0.0.1:9099');`
2. Add a `firebase.json` in that dir configuring the `firestore` (8080), `auth` (9099), and `hosting` (`"public": "."`) emulators.
3. Start with `firebase emulators:start --project bolao-brasil-haiti-ddf6f` (no auth/credentials required for emulators).
4. Seed the pool list so the form is usable — create the `config/boloes` document via the emulator REST API, e.g. `PATCH http://127.0.0.1:8080/v1/projects/bolao-brasil-haiti-ddf6f/databases/(default)/documents/config/boloes` with at least one `bolão` map entry (fields: `id`, `titulo`, `adversario`, `bandeiraAdversario`, `ordem`, ...). The production shape can be read (read-only) from the live Firestore REST API for reference.
5. Open the hosting URL (`http://127.0.0.1:5000`) and submit a prediction; verify it appears in the list and in the emulator at `/v1/projects/.../documents/palpites`.

### Firestore data model (collections)
- `config/boloes` — map of pools/games keyed by id (`titulo`, `adversario`, `bandeiraAdversario`, `grupo`, `dataHora`, `local`, `ordem`).
- `palpites` — predictions (`nome`, `brasil`, `haiti`, `bolaoId`, `timestamp`).
- `config/resultado-<bolaoId>` — official result. `config/trancado-<bolaoId>` — betting lock state (`ativo`).

### Lint / test / build
- No linter, test suite, or build system exists in this repo.
