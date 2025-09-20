Orbyta Foosball Cup 🏆

A realtime foosball tournament web app for Orbyta SRL with:
	•	Swiss-style qualifiers (1–2 rounds depending on the number of teams)
	•	Live knockout bracket (always ends with a Final)
	•	Balanced team generation (by role and skill)
	•	Seeding and Buchholz tiebreakers
	•	Google Login (Firebase Auth) and Firestore realtime sync
	•	Admin-only controls (allowlist + PIN + audit log)
	•	Clean neon UI with a built-in “How it works” modal

⸻

Table of Contents
	•	Live Demo & Repo
	•	Features
	•	Tournament Format
	•	How Standings Are Calculated
	•	Roles & Access Control
	•	Architecture
	•	Data Model
	•	Security Rules
	•	Setup & Installation
	•	Usage Guide
	•	Deployment
	•	Styling & UI
	•	Known Limitations
	•	Roadmap / Ideas
	•	License

⸻

Live Demo & Repo
	•	Live Demo: https://orbyta-foosball.web.app/

⸻

Features
	•	Google Sign-In (Firebase Auth)
Required to view the tournament (the main content is hidden until login).
	•	Admin-only actions (CRUD)
Admins must be on an allowlist in Firestore and also enter an Admin PIN in the UI. Admins can:
	•	Add players (Name; Role; Level)
	•	Generate balanced teams
	•	Build Swiss rounds
	•	Mark match winners (Swiss & Knockout)
	•	Save and Reset
	•	Set the tournament year (visible to everyone)
	•	Balanced Teams Algorithm
	•	Attacker list sorted desc by level (strongest first)
	•	Defender list sorted asc by level (weakest first)
	•	Pair i-th attacker with i-th defender → balanced pair
	•	Assign a seed based on team average (higher avg = better seed)
	•	Swiss Qualifiers (live)
	•	R1 seeded pairings (1–2, 3–4, …)
	•	R2 (if applicable) winners vs winners, losers vs losers, avoiding rematches
	•	Supports any even number of teams (2, 4, 6, 8, 10, 12, …)
	•	Bracket (always with Final)
	•	8+ teams → Top-8: Quarterfinals → Semifinals → Final
	•	4–6 teams → Top-4: Semifinals → Final
	•	2 teams → Final only
	•	Click “Vince” to advance winners → instantly moves teams forward
	•	Realtime Sync
	•	Any change made by an admin appears live to all logged-in users via onSnapshot.
	•	“How it works” modal
	•	Clean, non-intrusive info button in the header opens a modal that explains seed, Buchholz, Swiss logic, and bracket flow.

⸻

Tournament Format

To keep it fair and fast, the app uses Swiss qualifiers followed by single-elimination:
	•	2 teams total → Direct Final.
	•	4 or 6 teams → Swiss R1 (each team plays once) → Top-4 (Semifinals → Final).
	•	8+ teams → Swiss R1 + Swiss R2 (each team plays twice) → Top-8 (Quarters → Semis → Final).

This ensures:
	•	Everyone plays at least one (often two) matches before elimination
	•	There is always a Final
	•	The bracket feels competitive without being overly long

⸻

How Standings Are Calculated
	•	Seed (testa di serie)
Initial team strength. Computed from the average level of the two players. Lower seed number means stronger on paper (Seed 1 is best). Used to pair Round 1.
	•	Points
Win = 1, Loss = 0.
	•	Buchholz
Sum of the final points of your opponents in the Swiss rounds — a measure of schedule strength. If you faced strong teams (who also won), your Buchholz is higher.
	•	Swiss Ranking Order
	1.	Points (desc)
	2.	Buchholz (desc)
	3.	Seed (asc)

The top teams (Top-8 or Top-4, depending on total teams) advance to the knockout stage.

⸻

Roles & Access Control
	•	Viewer (logged-in user)
Must Login with Google to view the tournament (main content hidden until authenticated). Viewers see: year, Swiss rounds, bracket, champion — live.
	•	Admin (allowlisted + PIN)
Must:
	1.	Log in with Google
	2.	Be listed in Firestore collection admins/{uid}
	3.	Enter valid Admin PIN in the UI
Only admins can modify players, generate teams, pairings, and set winners.
	•	Audit log
Admin “enter” / “exit” is logged in adminSessions.

You can also enforce server-side read restrictions (see Security Rules) to require request.auth != null for reads.

⸻

Architecture
	•	Frontend:
	•	HTML + CSS (no framework)
	•	app.js ES module (imports Firebase SDK 10.x via CDN)
	•	Backend:
	•	Firebase Auth (Google Provider)
	•	Firestore (single document for tournament state, allowlist for admins)
	•	Realtime:
	•	Firestore onSnapshot keeps all users synchronized
	•	UI:
	•	Header with year, login controls, admin PIN
	•	Editor (admin-only) for player input and actions
	•	“Gare iniziali” card shows Swiss rounds
	•	“Tabellone eliminazione diretta” shows knockout bracket & champion
	•	Info modal (how it works)

⸻

Data Model

Collections
	•	foosball/state (single document for the live tournament)
	•	admins/{uid} (empty doc presence means allowlisted admin)
	•	adminSessions/{autoId} (audit entries)

foosball/state document schema

{
  "year": 2025,
  "playersText": "Name; Role; Level\n...",
  "players": [
    { "name": "Mario Rossi", "role": "Attacker", "level": 4 },
    { "name": "Luca Bianchi", "role": "Defender", "level": 2 }
  ],
  "teams": [
    {
      "id": "T1",
      "attacker": { "name": "Mario Rossi", "role": "Attacker", "level": 4 },
      "defender": { "name": "Luca Bianchi", "role": "Defender", "level": 2 },
      "avg": 3,
      "seed": 2
    }
  ],
  "swiss": {
    "rounds": [
      {
        "name": "Swiss R1",
        "matches": [
          {
            "id": "SR1-1",
            "teamA": { "id": "T1", "...": "..." },
            "teamB": { "id": "T2", "...": "..." },
            "winner": null
          }
        ]
      },
      {
        "name": "Swiss R2",
        "matches": [ /* same shape as above */ ]
      }
    ],
    "done": false
  },
  "rounds": [
    {
      "name": "Quarti" | "Semifinali" | "Finale",
      "matches": [
        {
          "id": "QF-1",
          "teamA": { "id": "T1", "...": "..." } | null,
          "teamB": { "id": "T8", "...": "..." } | null,
          "winner": null | "A" | "B"
        }
      ]
    }
  ],
  "champion": null | "Mario & Luca",
  "updatedAt": "<serverTimestamp>"
}


⸻

Security Rules

A good default (requires login to read, and only allowlisted admins to write):

rules_version = '2';

service cloud.firestore {
  match /databases/{db}/documents {

    function isAdmin() {
      return request.auth != null
        && exists(/databases/$(db)/documents/admins/$(request.auth.uid));
    }

    // Tournament state
    match /foosball/{docId} {
      allow read: if request.auth != null;  // require login to view
      allow write: if isAdmin();            // only admins can write
    }

    // Admins allowlist
    match /admins/{uid} {
      allow read: if request.auth != null;  // optional: or true
      allow write: if isAdmin();            // only an existing admin can modify
    }

    // Optional admin session logs
    match /adminSessions/{id} {
      allow read: if isAdmin();
      allow write: if isAdmin();
    }

    match /{document=**} {
      allow read, write: if false;
    }
  }
}


⸻

Usage Guide

As a logged-in user
	•	See the year, Gare iniziali (Swiss), Tabellone, and the Campioni banner when available.
	•	Click the ℹ️ Come funziona button in the header to read the format rules.

Admin flow (high level)
	1.	Login with Google (must be allowlisted).
	2.	Enter the Admin PIN.
	3.	Paste players in the format Name; Role; Level (role = Attacker/Defender, level 1–5, roles must be balanced).
	4.	Click Crea squadre to generate balanced teams + Swiss R1.
	5.	In Gare iniziali:
	•	Mark winners for R1.
	•	If 8+ teams, R2 is auto-generated. Mark R2 winners.
	•	The app builds the knockout bracket (Top-8 or Top-4) automatically.
	6.	In Tabellone, click Vince to advance winners through Semifinals and Final.
	7.	Use Salva when desired (auto-saves also on major actions).
Use Reset to start over (keeps the selected year unless you remove that line in code).

⸻

Deployment

Firebase Hosting is recommended:

npm install -g firebase-tools
firebase login
firebase init hosting
# choose your project, set "public" to the folder where index.html lives
firebase deploy

	•	Make sure your Auth domain matches your hosting domain.
	•	Your Firestore rules should already be deployed.

⸻

Styling & UI
	•	Neon, glassy look using CSS variables and gradients.
	•	Responsive grid-based layout.
	•	Subtle animations for polish.
	•	Admin-only sections are hidden by .admin-only when in readonly mode.
	•	Main content (<main>) is hidden until login (JS toggles display: none/grid based on auth).
	•	Info Modal: a ghost button in header opens a modal with clear, concise tournament rules.

⸻

Known Limitations
	•	Player input expects balanced roles (same number of Attackers/Defenders). The app enforces this to maintain the balanced pairing logic.
	•	Team count must be even; if there’s an odd pair count, add/remove a player to fix balance.
	•	Currently supports Swiss R1 or Swiss R1+R2 only (by design, for time efficiency).
