# Stub — screenshot → event → Google Calendar

A single HTML file, no build step, no backend. It runs entirely in your browser:

1. You upload a screenshot of an event (invite, poster, Instagram post, email...).
2. It's sent to the **free** Gemini API, which extracts title / date / time / location / description.
3. You check the extracted "ticket stub," edit anything that's wrong, and add it to your Google Calendar with one tap — using Google's own sign-in, no server involved.

The Google OAuth client ID is already baked into `index.html` (it's not a secret — it just identifies the app to Google, same way every app's client ID is visible in its own network requests). The only thing you personally enter is your Gemini key, which is stored only in your browser's `localStorage` and never sent anywhere except directly to Google's Gemini API.

## 1. Get a free Gemini API key

1. Go to **https://aistudio.google.com/apikey**
2. Sign in, click **Create API key**.
3. Copy it — you'll paste it into the app's settings panel.

The free tier (Gemini 2.5 Flash) comfortably covers casual personal use — a handful of screenshots a day is nowhere near the daily limit.

## 2. Create a Google OAuth client ID

This is what lets the app ask *your* Google account for permission to add calendar events — no backend or client secret needed for this flow.

1. Go to **https://console.cloud.google.com/** and create a new project (or reuse one).
2. **APIs & Services → Library** → search **Google Calendar API** → Enable.
3. **APIs & Services → OAuth consent screen**:
   - User type: External
   - Fill in app name (e.g. "Stub"), your email for support/developer contact
   - Scopes: you can skip adding scopes here, the app requests them itself
   - **Test users**: add your own Google account email — while the app is "in testing" only test users can sign in, which is exactly what you want for a personal tool
4. **APIs & Services → Credentials → Create credentials → OAuth client ID**:
   - Application type: **Web application**
   - Name: anything
   - **Authorized JavaScript origins**: add the exact URL you'll open the app from (see hosting step below) — e.g. `http://localhost:8000` or `https://your-app.netlify.app`
   - Leave redirect URIs empty — not needed for this flow
   - Create, then copy the **Client ID** (looks like `123...apps.googleusercontent.com`)

## 3. Host the file somewhere

Google's sign-in won't work from a `file://` path, so the page needs a real URL — but it doesn't need a real server, just static hosting.

**Easiest for phone access — Netlify Drop (free, no account needed for a quick test):**
1. Go to **https://app.netlify.com/drop**
2. Drag `index.html` onto the page
3. You'll get a URL like `https://random-name-123.netlify.app`
4. Go back to Google Cloud Console → your OAuth client → add that exact URL to **Authorized JavaScript origins**
5. Open that URL on your phone or laptop — bookmark it / add to home screen for quick access

**For local testing on your computer:**
```bash
cd stub-app
python3 -m http.server 8000
```
Then open `http://localhost:8000`, and add `http://localhost:8000` as an authorized origin.

## 4. Use it

1. Open the app, tap the ⚙ icon, paste in your Gemini key, save.
2. Upload or paste a screenshot.
3. Tap **Extract event details** — Gemini reads the image and fills in the ticket.
4. Fix anything it got wrong (it's all editable).
5. **Sign in with Google**, then **Add to Google Calendar**.

## Notes

- The consent screen will show an "unverified app" warning since you haven't submitted it for Google review — that's expected for a personal tool. Click "Advanced → Go to Stub (unsafe)" to proceed; only you (as a listed test user) can sign in anyway.
- If Gemini can't find a clear date/time, it'll leave those blank rather than guess — always double-check the ticket before adding.
- The app calls `gemini-flash-latest`, Google's self-updating alias for its current flash model, specifically so you don't have to keep editing this file every time a dated model version (like `gemini-2.5-flash`) gets retired — Google has been retiring those every few months in 2026. If `gemini-flash-latest` ever 404s too, check **https://ai.google.dev/gemini-api/docs/models** for the current alias/model name and swap it in `index.html` (search for `generateContent`).
