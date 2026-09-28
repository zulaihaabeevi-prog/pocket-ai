# Packet AI — Yearly Budget Planner

A single-page app (`index.html`) with:
- Multi-user login (email + password)
- A database so each person's data syncs across devices
- A full 12-month plan (auto-fill all months from one month, then edit any month)
- A "Goals" tab for things you're saving up to buy, with progress tracking

It uses **Firebase** (Google's free backend) for login and storage, and is ready to host on **GitHub Pages**.

---

## 1. Create your free Firebase project (5 min)

1. Go to https://console.firebase.google.com → **Add project** → give it a name → finish setup.
2. In the left sidebar, click **Build → Authentication → Get started**.
   - Under **Sign-in method**, enable **Email/Password**.
3. In the left sidebar, click **Build → Firestore Database → Create database**.
   - Choose **Start in test mode** for now (you'll lock it down in step 3 below).
   - Pick any region close to you.
4. Click the **gear icon (⚙) → Project settings**, scroll to **Your apps**, click the **`</>`** (web) icon, register an app (any nickname), and **skip** the "Firebase Hosting" checkbox.
5. Firebase will show you a `firebaseConfig` object like:
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "yourapp.firebaseapp.com",
     projectId: "yourapp",
     storageBucket: "yourapp.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```
   Copy it.

## 2. Paste your config into the app

Open `index.html`, find this block near the top of the `<script>` section, and replace the placeholder values with the ones you copied:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

It's fine that this key is visible in your public HTML — Firebase web API keys aren't secret. What actually protects each person's data is the security rule in the next step.

## 3. Lock down the database (important — do this before real use)

In Firebase Console → **Firestore Database → Rules**, replace the default rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

This makes sure a logged-in person can only ever read or write their **own** budget data, never anyone else's. Click **Publish**.

## 4. Test it locally

Just open `index.html` in a browser. Sign up with an email/password, build a month's plan, add a goal — then reload the page and log back in to confirm your data is there.

## 5. Host it on GitHub Pages

1. Create a new GitHub repository (e.g. `packet-ai-budget`).
2. Upload `index.html` (with your Firebase config filled in) to the repo — either via the GitHub web UI ("Add file → Upload files") or:
   ```bash
   git init
   git add index.html
   git commit -m "Packet AI budget planner"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/packet-ai-budget.git
   git push -u origin main
   ```
3. In the repo, go to **Settings → Pages**.
4. Under **Build and deployment → Source**, choose **Deploy from a branch**, branch **main**, folder **/ (root)** → **Save**.
5. After a minute, GitHub shows your live URL:
   `https://YOUR_USERNAME.github.io/packet-ai-budget/`

One more Firebase step once you have that URL: in Firebase Console → **Authentication → Settings → Authorized domains**, click **Add domain** and add `YOUR_USERNAME.github.io`. (Without this, login will be blocked on the live site even though it worked locally.)

That's it — anyone can now open the link, sign up, and get their own private budget year.

---

### How the data is organized (for reference)
Each user gets one Firestore document at `users/{their-login-id}` containing:
- `base` — their last-used monthly inputs (used as the starting point for a new month)
- `months` — one entry per month (`"2026-01"`, `"2026-02"`, …) with that month's inputs and computed plan
- `goals` — the list of things they're saving up to buy, with target amount, target date, and amount saved so far
- `year` — which year is currently shown in the Year Plan tab
