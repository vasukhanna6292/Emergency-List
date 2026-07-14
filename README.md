# Emergency List — iOS Concept & Working Prototype

A proof-of-concept for a built-in iOS emergency feature: **if your phone is lost or stolen, you can reach your trusted contacts and start recovery from any borrowed iPhone.**

Most people only remember their own number, not their family's. Emergency List lets a pre-configured user authenticate on a stranger's iPhone (passcode → Face Check) and immediately call an emergency contact, who can then share the lost phone's live location — cutting the time between losing a phone and acting on it.

This repository contains a **fully working, self-contained HTML prototype** — no build step, no backend to run.

---

## Live demo / deployment

The entire app is a single file: **`index.html`**. Host it on any static HTTPS host.

> ⚠️ **HTTPS is required.** Face recognition (camera), the live voice call (microphone), and location sharing use browser APIs that only work over `https://` (or `localhost`) — **not** from a downloaded `file://`.

### Deploy on Netlify (fastest)
1. Go to <https://app.netlify.com/drop>
2. Drag this repo's folder (or just `index.html`) onto the page.
3. Netlify gives you an `https://…netlify.app` link. Open it on your phone.

### Or connect this GitHub repo to Netlify
1. Netlify → **Add new site → Import an existing project → GitHub** → pick this repo.
2. Build command: **none.** Publish directory: **`/`** (root).
3. Deploy. (`netlify.toml` in this repo already sets this.)

### Or GitHub Pages
Repo **Settings → Pages → Deploy from branch → `main` / root**. Your site publishes at `https://<user>.github.io/<repo>/`.

---

## Features

### Recovery flow (single phone)
- **Home** — "I lost my phone" and "You're protected" (edit your setup).
- **Setup / Your Emergency List**
  - Your identity: phone number with **country-code picker** + **pull from Contacts**.
  - **Dynamic passcode** — choose **4 or 6 digits**, iOS-style.
  - Three-factor recovery: passcode (2nd factor) + **Face Check** (3rd factor).
  - **Emergency contacts** — add/edit/remove, each with country code or Contacts import.
  - **Per-contact location sharing** — switch on exactly which contacts may receive your location.
- **Lost-phone flow** — passcode → **Face Check** → case hub → call a contact → shared location on a map → put on hold / receive callback → **close case** (animated secure erase) → done.
- **Staged demo** (purple button) — auto-plays the full flow hands-free for recordings. Not part of the product.

### Real face recognition
- Uses [face-api.js](https://github.com/justadudewhohacks/face-api.js) entirely **on-device**.
- **Enroll** once (Setup → Face Check enrollment): the camera captures a 128-D face descriptor stored in this browser's `localStorage`.
- **Face Check** in the lost-phone flow runs the live camera and matches against the enrolled descriptor. Falls back to a simulated pass when no camera/enrollment is available (e.g. during the demo).

### Live two-phone mode (real, peer-to-peer)
Run the same page on **two phones at once** to do a genuine recovery test.
- **Zero setup** — no accounts, no backend, no config. Signaling goes through the free public [PeerJS](https://peerjs.com) broker; the call itself is **direct phone-to-phone WebRTC**.
- **Roles:** *Borrowed iPhone* (you) generates a **4-digit code**; *Family member* (your contact) enters it to pair.
- **Real WebRTC voice call** between the phones.
- **Live GPS location** shared from the family phone to the borrowed phone (with a manual-pin fallback if location is blocked), shown on a map with an "Open in Maps" link.
- **Session log**, hold/callback, and **close case** that ends the session on both phones.

To run: host over HTTPS → open on both phones → **Live two-phone test** → pick roles → pair with the code. An in-app **"How the live test works"** screen walks through it.

---

## Tech & architecture

- **Single-file app** — all logic, styles, and markup inlined in `index.html`. No framework install, no bundler.
- **Face recognition:** face-api.js (TinyFaceDetector + landmark + recognition nets), loaded from CDN, runs locally. Descriptor persisted in `localStorage`; nothing uploaded.
- **Live link:** PeerJS for WebRTC signaling over its public broker; a **data channel** relays session state (status, location, events) and a **media channel** carries live audio. Fully peer-to-peer — no server of yours.
- **Source of truth:** the prototype is authored as a component and compiled into the standalone `index.html`. Edit the source in the design tool and re-export; treat `index.html` as the build artifact.

### Privacy notes
- Face descriptor and setup data live only in the browser on that device.
- The live session is temporary; closing the case erases session data from both phones.
- The public PeerJS relay is fine for a **personal test**, not a production-hardened deployment.

### Honest limitations
- The setup **Contacts** list is a realistic mock — browsers cannot read a phone's real address book. Country-code + manual entry is fully real.
- Live voice/location/camera require **HTTPS**, **internet on both phones**, and the user granting **mic/camera/location** permission.
- Single-phone "call", the map pin outside live mode, and the callback are simulated for demo purposes.

---

## Repository layout
