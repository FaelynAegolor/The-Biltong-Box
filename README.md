# The Biltong Box

A drying tracker for biltong: log each strip's starting weight, check suggested
"ready" weights, and record daily temperature/humidity readings. Works as a
completely local, offline file — or you can wire it up to Firebase so it runs
as a real website that syncs live across every device you open it on.

This guide assumes you're starting from scratch: no Firebase project, no
GitHub repo yet. It'll take about 15–20 minutes, once.

## What you'll end up with

- A free GitHub Pages website (e.g. `https://yourusername.github.io/the-biltong-box/`)
- A free Firebase project storing your data in Firestore
- Live sync: open the site on your phone in the garage next to the biltong
  box, enter today's weights, and it's instantly there when you check from
  your laptop.

If you'd rather skip all of this, you don't have to — just open `index.html`
directly in a browser (double-click the file). It works fully offline, saving
to a local file on your computer instead. The Firebase setup below is only
needed for the "sync across devices" part.

---

## Part 1 — Create the Firebase project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com) and sign in with a Google account.
2. Click **Add project**. Give it any name (e.g. "biltong-box"). You can
   disable Google Analytics for this project — it's not needed.
3. Once the project is created, you'll land on the project overview page.

### Enable Firestore

4. In the left sidebar, click **Build → Firestore Database**.
5. Click **Create database**.
6. Choose **Start in production mode** (not test mode — we'll set our own
   rules below), pick any location close to you, and click **Enable**.

### Enable anonymous sign-in

7. In the left sidebar, click **Build → Authentication**.
8. Click **Get started**.
9. Under **Sign-in method**, find **Anonymous** in the provider list, click
   it, toggle **Enable**, and click **Save**.

   This lets the app quietly sign visitors in without asking them for a
   password — it's what the security rules below check for.

### Set the security rules

10. Back in **Firestore Database**, click the **Rules** tab.
11. Replace the contents with what's in `firestore.rules` (included in this
    package) — or just paste this:

    ```
    rules_version = '2';
    service cloud.firestore {
      match /databases/{database}/documents {
        match /biltong/data {
          allow read, write: if request.auth != null;
        }
        match /{document=**} {
          allow read, write: if false;
        }
      }
    }
    ```

12. Click **Publish**.

    This means: anyone who opens your site can read/write your biltong data
    (since the app signs them in anonymously automatically) — but nothing
    else in your Firestore project is reachable. That's a reasonable
    trade-off for a personal hobby tracker that isn't linked anywhere public.
    If you want it locked to just you, see the note at the bottom of
    `firestore.rules`.

### Get your web config

13. Click the **gear icon** next to "Project Overview" (top left) → **Project settings**.
14. Scroll to **Your apps**. Click the **`</>`** (web) icon to add a web app.
15. Give it a nickname (e.g. "biltong-box-web"), skip Firebase Hosting (we're
    using GitHub Pages instead), and click **Register app**.
16. You'll see a `firebaseConfig` object like this — copy the whole thing,
    you'll need it in Part 2:

    ```js
    const firebaseConfig = {
      apiKey: "AIza...",
      authDomain: "biltong-box.firebaseapp.com",
      projectId: "biltong-box",
      storageBucket: "biltong-box.appspot.com",
      messagingSenderId: "123456789",
      appId: "1:123456789:web:abc123"
    };
    ```

    These values are safe to make public (they identify your project, they
    don't authorize anything by themselves) — the security rules from step
    11 are what actually protects your data.

---

## Part 2 — Fill in the config

17. Open `index.html` in a text editor.
18. Near the very top of the `<script>` section, find this block:

    ```js
    var FIREBASE_CONFIG = {
      apiKey: "",
      authDomain: "",
      projectId: "",
      storageBucket: "",
      messagingSenderId: "",
      appId: ""
    };
    ```

19. Paste in your values from step 16, so it looks like:

    ```js
    var FIREBASE_CONFIG = {
      apiKey: "AIza...",
      authDomain: "biltong-box.firebaseapp.com",
      projectId: "biltong-box",
      storageBucket: "biltong-box.appspot.com",
      messagingSenderId: "123456789",
      appId: "1:123456789:web:abc123"
    };
    ```

20. Save the file. As long as `apiKey` is filled in, the app will connect to
    Firebase automatically on load — no other code changes needed. (Leave it
    blank and the app just runs locally instead, exactly as before.)

---

## Part 3 — Put it on GitHub Pages

21. Go to [https://github.com/new](https://github.com/new) and create a new
    repository (public is fine, and required for free GitHub Pages on a
    personal account). Name it whatever you like, e.g. `the-biltong-box`.
22. Upload `index.html` to the repo — either via the GitHub web UI
    ("Add file → Upload files", drag `index.html` in, commit), or via git:

    ```
    git clone https://github.com/yourusername/the-biltong-box.git
    cd the-biltong-box
    cp /path/to/index.html .
    git add index.html
    git commit -m "Add The Biltong Box"
    git push
    ```

    (You don't need to upload `firestore.rules` or this README to the repo —
    they're just reference docs. Fine to include them if you want.)

23. In the repo, go to **Settings → Pages**.
24. Under **Build and deployment → Source**, choose **Deploy from a branch**.
25. Under **Branch**, choose `main` (or `master`) and `/ (root)`, then **Save**.
26. Wait a minute or two, then refresh — GitHub will show you the live URL,
    something like `https://yourusername.github.io/the-biltong-box/`.

Open that URL. You should see The Biltong Box load, and within a second or
two the header should show **"🟢 Synced live via Firebase"**. Add a batch on
one device, open the same URL on another — it'll show up there too.

---

## Troubleshooting

- **Header shows "Firebase error — check console & config"** — open the
  browser's developer console (F12) for the actual error. Almost always
  either a typo in `FIREBASE_CONFIG`, Firestore not enabled yet, or the
  security rules not published.
- **Nothing happens / stuck on "Connecting to Firebase…"** — check that
  Anonymous sign-in is enabled (Part 1, step 9) and the rules are published
  (step 12).
- **"Missing or insufficient permissions"** in the console — the rules
  weren't published, or don't match `biltong/data` exactly.
- **Changes on one device don't show on another** — make sure both are
  loading the *same* GitHub Pages URL (not a locally opened copy of
  `index.html`, which runs Firebase too, but a locally-opened file and the
  deployed site are still the same app talking to the same Firestore
  document, so this should just work — double check you edited and
  deployed the version with your real config filled in).

## Updating the site later

Any time you want to change the tracker itself (new features, tweaks), just
replace `index.html` in the repo with the new version and push — GitHub
Pages redeploys automatically within a minute or two. Your data stays in
Firestore, untouched, independent of the code.
