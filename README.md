# Best Bird of 2026 — MontCo Birders Rare Bird Poll

A mobile-first poll for voting on the club's best rare bird sightings of the
year: full-screen photos that flip to sighting details, swipe between
sightings, rank your top 3, and see live results. Backed by Firebase
Firestore so sightings and votes sync in real time for everyone with the
link — open it up now and keep adding sightings all the way through
December, no expiration built in.

## 1. Create a Firebase project

1. Go to https://console.firebase.google.com → **Add project** (the free
   "Spark" plan is plenty for this). Use a new project separate from any
   other app you've built — keeps this poll's data cleanly on its own.
2. Once created, click the **</> (Web)** icon to register a web app. Give it
   any nickname — you don't need Firebase Hosting, just the config.
3. Copy the `firebaseConfig` values it shows you (apiKey, authDomain, etc.).

## 2. Turn on Firestore

1. In the left sidebar: **Build → Firestore Database → Create database**.
2. Choose **Start in test mode** for now (you'll tighten this below).
3. Pick any region close to your group.

### Security rules

Go to the **Rules** tab of Firestore and paste this in:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /sightings/{sightingId} {
      allow read: if true;
      allow write: if true;
    }
    match /votes/{voteId} {
      allow read: if true;
      allow write: if true;
    }
    match /settings/{docId} {
      allow read: if true;
      allow write: if true;
    }
  }
}
```

**Worth knowing:** this keeps things simple for a casual, unlisted-link poll,
but it means anyone who finds the link (or opens their browser's dev tools)
could technically write to the database directly — the "editor passcode" in
the app is a friendly gate, not real security, since it lives in the
JavaScript that ships to every visitor. If you ever want real access
control, that means adding Firebase Authentication and rules based on
signed-in users — a bigger change, happy to help with it if you get there.

## 3. Configure the app locally

```bash
cd montco-rare-birds
cp .env.example .env
```

Open `.env` and fill in the six values from step 1:

```
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
```

Then also open `src/App.jsx` and change this line near the top to your own
passcode:

```js
const EDITOR_PASSCODE = "rarebird2026";
```

## 4. Run it locally

```bash
npm install
npm run dev
```

Visit the local URL it prints — add a sighting or two, try voting, confirm
it all works before you deploy.

## 5. Push to GitHub

```bash
git init
git add .
git commit -m "MontCo Birders rare bird poll"
git remote add origin <your-repo-url>
git push -u origin main
```

(`.env` is already in `.gitignore` — your Firebase keys won't get committed.)

## 6. Deploy on Netlify

1. Netlify → **Add new site → Import an existing project** → pick your repo.
2. Build command and publish directory are already set via `netlify.toml`
   (`npm run build`, `dist`) — Netlify should detect them automatically.
3. Before the first deploy, go to **Site configuration → Environment
   variables** and add the same six `VITE_FIREBASE_*` values from your
   `.env` file.
4. Deploy. Share the resulting `*.netlify.app` URL with the club.

## Adding a sighting (photo workflow)

There's no automated photo lookup in this version — you add each sighting by
hand from **Manage**, which matches how you'd actually curate a "best of"
list anyway:

1. Find the sighting on the eBird checklist or Macaulay Library.
2. On the Macaulay Library asset page, click **Embed** (Cornell's official
   tool for reusing photos elsewhere, sanctioned for non-commercial use —
   this is exactly that). Copy the image URL from what it gives you.
3. Paste that into the **Photo URL** field. Always fill in **Photo credit**
   with the photographer's name too — required by Macaulay's usage terms,
   and it's just the right thing to do for people's photos.
4. If an asset shows as "restricted," it isn't available this way — you'd
   need to ask the photographer directly, or use a different photo of that
   sighting.
5. Write up the sighting details yourself in the **Sighting details** field —
   species, date, location, and whatever makes it notable. This is the
   "synopsis" people read when they flip the card.

If you'd rather crop or edit a photo yourself before adding it: aim for
roughly 1200px on the long edge, JPEG at ~80% quality, so it loads quickly on
phones. Then host it anywhere with a stable public URL (an image host, or
even just its Macaulay/eBird link) and paste that URL into the same field —
no upload step built into the app.

## How it works day to day

Every visitor gets asked their name once (stored on their own device), and
anyone who knows your editor passcode can open **Manage** to add, edit, or
remove sightings — swipe right from the left edge of a sighting in that list
to delete it, or tap the pencil to edit one in place. Everyone else browses
the sightings, flips each card to read the details, and ranks their top 3
once they've seen them all. Rankings save automatically as people tap — no
submit button, and tapping a pick again clears it. Results tally live
(1st = 3 points, 2nd = 2, 3rd = 1) and update for everyone in real time.

## Results reveal

By default, the Results tab is hidden from everyone except the manager (the
device that's unlocked Editor mode). In **Manage**, under "Results reveal,"
you can:

- Set a **scheduled reveal time** — once that moment passes, Results
  unlocks automatically for everyone, no action needed from you at the time.
- **Reveal now** — flips it open immediately, overriding any scheduled time
  (handy if you want to reveal it live at a get-together instead of waiting
  on a timer).
- **Re-hide results** — flips it back closed if you change your mind, even
  after a scheduled time has passed.

The manager's own device always sees results, regardless of these settings —
useful for double-checking the poll is working, less useful if you want the
reveal to be a surprise for yourself too. There's no way to hide it from the
manager currently; if that's ever wanted, it'd need its own toggle.
