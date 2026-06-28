# Charaivati — setup & sync

**चरैवेति** ("keep moving forward"). A tiny offline-first task app. One task at a time. Data lives on your device; optional **free** live sync keeps laptop ↔ phone in step via Firebase.

---

## A. Run it

- **Desktop now:** open `index.html` in a browser. (The app runs; the service worker / "install" needs HTTPS — see B.)
- **Phone (free, installable):** host the folder over HTTPS, then on the phone use the browser's **Add to Home Screen**. Free hosting options:
  - **GitHub Pages:** push this folder to a repo → Settings → Pages → deploy from `main`. You get an `https://<you>.github.io/<repo>/` URL.
  - **Netlify:** drag the folder onto app.netlify.com → instant HTTPS URL.

---

## B. Turn on laptop ↔ phone sync (Firebase — free)

One-time, ~10 minutes. You click through the console; the code is already written.

### 1. Create the project
1. Go to <https://console.firebase.google.com> → **Add project** → name it `momentum` → you can disable Analytics → **Create**.

### 2. Add a Web app
1. On the project home, click the **`</>`** (Web) icon → register app (nickname `momentum`) → **Register app**.
2. It shows a `firebaseConfig = { … }` block. Copy the values for `apiKey`, `authDomain`, `projectId`, `appId`.
3. Open `index.html`, find the **PASTE YOUR FIREBASE CONFIG HERE** block (near the bottom), and replace the four `PASTE_ME` values.
   > The `apiKey` is **not** a secret — it's safe in a public repo. Security is enforced by the rules in step 4.

### 3. Enable Google sign-in
1. Console → **Build → Authentication → Get started**.
2. **Sign-in method** tab → **Google** → enable → pick a support email → **Save**.
3. **Settings → Authorized domains** → **Add domain** → add your hosting domain (e.g. `yourname.github.io`). `localhost` is already allowed for desktop testing.

### 4. Create the database + lock it down
1. Console → **Build → Firestore Database → Create database** → **Start in production mode** → pick a region → **Enable**.
2. **Rules** tab → replace everything with this, then **Publish**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{uid} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```

   This means: only *you*, signed in, can read/write *your* tasks. No one else can touch them.

### 5. Use it
1. Open the app on your **laptop** → tap **🔗 Sign in to sync** → sign in with Google. Status shows **✅ Synced**.
2. Open the app on your **phone** → sign in with the **same Google account**. Both now share one task list, live — add a task on one, it appears on the other within seconds.
3. Works offline too: edits made with no signal sync automatically when you're back online.

---

## How it works (so future-you remembers)
- **Local-first:** tasks are always in the browser's `localStorage`; the app works with zero network.
- **Sync:** when signed in, every change is pushed (debounced) to `users/{your-uid}` in Firestore, and a live listener pulls remote changes back. On first sign-in on a device, local + cloud tasks are **merged** (nothing lost); after that, the cloud is the source of truth.
- **Conflict note:** if you edit the *same* data on two devices while *both* are offline, the last one to reconnect wins. For solo use this is rarely an issue.

## Free-tier headroom
Firestore free (Spark) plan: 1 GB stored, 50K reads + 20K writes/day. A single person's task app uses a rounding error of that. Cost: **$0**.
