# WanZek

**An urban art market where graffiti writers are known by their work, not their name — with payments requested to Zcash shielded addresses.**

Built for the **Zecathon by ZKSNARKS (NODO 07)** · Track: **Wildcard** (art + privacy) · Demo project, **not commercial**.

**Live demo:** https://wanzek.vercel.app · **Code:** https://github.com/m1lesmodalespro/wanzek

> **Read this first — honesty note.** WanZek is a front-end demo. A few things are *real* and verifiable (the Zcash payment request, the in-browser authorship signature, photo metadata cleaning). Several things that a production marketplace needs — escrow, payment detection, shared accounts, chat — are **simulated or not implemented**. Everything is listed explicitly in [Real vs. simulated](#real-vs-simulated). Nothing in this repository moves real funds.

---

## Table of contents

1. [TL;DR for judges](#tldr-for-judges)
2. [The problem and the idea](#the-problem-and-the-idea)
3. [60-second walkthrough](#60-second-walkthrough)
4. [Real vs. simulated](#real-vs-simulated)
5. [Feature details](#feature-details)
6. [Payment flow in detail](#payment-flow-in-detail)
7. [Authorship signature in detail](#authorship-signature-in-detail)
8. [Order lifecycle and rules](#order-lifecycle-and-rules)
9. [Privacy model and data handling](#privacy-model-and-data-handling)
10. [Tech stack and architecture](#tech-stack-and-architecture)
11. [Run it / deploy it](#run-it--deploy-it)
12. [Testing and verification](#testing-and-verification)
13. [Known limitations](#known-limitations)
14. [Roadmap (ideas, not promises)](#roadmap-ideas-not-promises)
15. [Credits and licenses](#credits-and-licenses)

---

## TL;DR for judges

| | |
|---|---|
| **What it is** | A single-page, static web app: a marketplace where graffiti writers publish work under a pseudonym and clients buy, make offers, or commission pieces. |
| **Zcash's role** | Every order generates a **ZIP-321 payment request** (shielded address + amount + memo tying the payment to the order), shown as a link and a QR code. Transparent addresses are rejected. |
| **What actually works** | Payment-request generation (link + QR) · authorship signature (ECDSA P-256 via WebCrypto) · photo metadata stripping · role-based UI, privacy toggles, ES/EN. |
| **What is simulated** | Escrow/custody, payment confirmation, timers and refunds, shared accounts and orders, delivery of the artwork. |
| **What is not built** | Chat, video upload, accept/decline of offers, any backend. |
| **Live demo** | https://wanzek.vercel.app (static deployment of `index.html`) |
| **Backend** | None. All data lives in the visitor's browser (`localStorage`). |
| **Real-payment test** | The payment-request mechanism was **tested by the author with a real shielded payment** from a standard Zcash wallet during development. No transaction data and no wallet addresses are published in this repo or in the submission video. |

---

## The problem and the idea

Graffiti writers build their reputation under a pseudonym. Working with companies often means going through an agency and revealing a real identity — and getting paid usually exposes financial details too.

WanZek explores a different shape:

- **The client sees the work and the pseudonym**, not a legal name.
- **The artist decides what is public**: which works, which prices, whether to show a city, which links.
- **Zcash shielded addresses** keep the payment side private by design.
- **A cryptographic signature** lets a pseudonym be tied to a key, without revealing who holds it.

This is a design exploration and a working demo of the pieces that can be built client-side today. It is **not** a claim of absolute anonymity (see [Privacy model](#privacy-model-and-data-handling)).

---

## 60-second walkthrough

1. Open the app and tap through the intro. You land on the public profile of the sample artist **Senk (Zurdo Caribe)**, with a featured-work showcase and a gallery (filters: All / Available / Sold; click any work to zoom).
2. Top right → **Sign in → I'm an artist** → *Create my profile* (only a pseudonym is required; accept the rules).
3. **My studio** → toggle *Publish* / *Show price* per work, add links, then **Create my authorship key**. A private key is shown once (blurred in the video). Press **Check signature**.
4. Top right → **Switch to client** → create a client account (name, email, phone are required for clients) → press **Commission a work** → fill in details, a delivery date, an amount → accept the rules → **Send order**.
5. **My orders** → the order shows a *How to pay* panel with a **QR code and a `zcash:` link** (the address text is the one the artist configured).
6. Press **I've sent the payment**. Open **Demo mode** at the bottom and enable simulation controls → **Simulate: payment received**.
7. Switch to the artist → **Orders received** → **Mark as delivered**. Switch to the client → **Confirm receipt**. The order completes and the payout breakdown is displayed.

Steps 6–7 are **simulated** (see below).

---

## Real vs. simulated

### ✅ Real (works today, verifiable in the browser)

| Feature | What it actually does |
|---|---|
| **Zcash payment request** | Builds a standards-based [ZIP-321](https://zips.z.cash/zip-0321) URI: `zcash:<address>?amount=<ZEC>&memo=<base64url>`, renders it as a **QR code** and as an **Open in wallet** link. The memo is `wanzek-<orderId>` so the payment can be matched to the order. |
| **Shielded-address check (format only)** | Accepts addresses that start with `u1…` (unified) or `zs1…` (Sapling). **Rejects transparent addresses** (`t1…`, `t3…`) with an explanatory message. This is a *format* check — it does **not** validate the address checksum. |
| **Authorship signature** | Generates an **ECDSA P-256** key pair with the browser's **WebCrypto** API, signs a message containing the pseudonym and a timestamp, stores the *public* key + signature + fingerprint locally, and can **verify** the signature. The private key is shown once and **never stored**. |
| **Photo metadata cleaning** | When an artist uploads a work, the image is re-drawn on a canvas and re-encoded as a new JPEG (max 1600 px), which **drops EXIF data** (GPS, camera model, timestamps). |
| **Privacy toggles** | Per work: *Publish* and *Show price*. Profile: city visibility, links. Nothing is required except a pseudonym. These control what the profile page displays. |
| **Role-based interface** | Visitor / client / artist each see only their own navigation and data. |
| **Bilingual UI** | Spanish and English, switchable at any time (user-entered text is not translated). |
| **Responsive, accessible-minded UI** | Works on phones and desktops; respects `prefers-reduced-motion`; keyboard-operable lightbox. (No formal accessibility audit.) |

### 🟡 Simulated (looks and behaves like the real thing, but no real backend or funds)

| Feature | What actually happens |
|---|---|
| **Escrow / "payment held"** | **No funds are held.** The QR points at the payout address the artist typed. "Payment held" is only a UI state. A real deployment would need a custodian, multisig/threshold scheme, or another mechanism — not implemented here. |
| **Payment confirmation** | **There is no payment detection.** *I've sent the payment* is a self-report, and *Simulate: payment received* is a demo control (visible only in Demo mode). |
| **Timers (36 h cancel window, 5-day confirmation window)** | Computed from the browser clock. Demo mode can fast-forward time (**+36 h**, **+5 days**) to show each outcome. |
| **Settlements, refunds and the platform fee** | Amounts (95/5, 50/50, 10 % late-cancel, 1 % fee) are **calculated and displayed**. **No transfer is executed.** |
| **Artwork delivery and "locked until confirmed"** | There is no file exchange. The *locked* state is a placeholder that unlocks on confirmation. |
| **Accounts and orders shared between users** | **Not shared.** Data is stored per browser; one person plays both roles using *Switch role*. |
| **Client personal data** | The client form collects name/email/phone to illustrate the "client must be identifiable for disputes" rule. It is stored **only in the visitor's browser**, never transmitted. |
| **Sample catalog** | Four real artworks by Senk are embedded. **Statuses and sold prices (e.g. 0.60 / 0.45 ZEC) are illustrative**, not real sales. |

### ❌ Not implemented

- **Chat / messaging** between client and artist.
- **Video upload** for artists (links to external pages are supported; video upload is not).
- **Accepting or declining offers** — an offer is created as an order with a proposed amount; there is no artist accept/decline step.
- **Any server, database, or user authentication.**
- **Automatic payment detection** (e.g., scanning the chain with a viewing key).
- **Public registry or third-party verification** of authorship signatures.

---

## Feature details

### Visitor
- Browse the public profile, showcase, gallery and rules; switch language.
- To order, the visitor is asked to sign in as a client.

### Artist
- **Profile:** pseudonym (only required field), alternate tag name, crew, city (optional, with a visibility toggle), short bio, up to **5 external links**.
- **Payout address:** a shielded address (`u1…` / `zs1…`). Transparent addresses are rejected.
- **Authorship key:** create, copy private key once, check signature, regenerate.
- **Works table:** *Publish*, *Show price* and price per work; upload new works (metadata stripped; uploaded images live in memory for the session only).
- **Orders received:** see orders, with only the client's account type (*personal* / company name) — not their personal data. Mark an order as delivered.

### Client
- **Account:** full name, email, phone (required), personal or company account (company name optional).
- **Order types:** *Buy* (price published by the artist), *Make an offer or ask* (client proposes an amount), *Commission a work* (client describes the piece, proposes amount and delivery date).
- **My orders:** payment instructions with QR, status, cancel, confirm receipt, payout breakdown.

### Everyone
- **Showcase:** auto-advancing featured works (pauses on hover/focus; disabled with reduced-motion), swipe on touch.
- **Lightbox:** zoom, previous/next, keyboard arrows, `Esc` to close.
- **Rules page** and an optional, dismissible *How it works* guide that can be reopened from the footer.
- **Demo mode** panel (Orders page) with the simulation controls.

---

## Payment flow in detail

1. The client submits an order with an amount (ZEC, up to 8 decimals) and a delivery date.
2. On the order page, WanZek builds a payment request for the **artist's payout address**:

   ```text
   zcash:<shielded-address>?amount=<amount>&memo=<base64url("wanzek-<orderId>")>
   ```

3. The request is shown as a **QR code** and an **Open in wallet** link, with step-by-step instructions for the client (open wallet → scan / tap → verify the address is shielded and amount/memo match → send).
4. If the artist has not configured a valid shielded address, **no QR is produced** and the order page says so.
5. The client presses **I've sent the payment**. From here on, the state machine is **simulated** (see [Order lifecycle](#order-lifecycle-and-rules)).

Notes:

- **Wallet support varies.** Not every wallet opens `zcash:` links or pre-fills the memo from a QR; if one doesn't, copy the link or enter the memo manually.
- **Memos are visible only to the recipient** of a shielded payment, but they are *not* a secure messaging channel; do not put personal data in them. WanZek only places an order reference.
- **Tested manually:** the author sent a **real shielded payment** from a standard Zcash wallet during development, using an earlier standalone version of the payment-request page that builds the request with the same construction (address + amount + base64url memo), and received it. (How the memo is displayed depends on the wallet.) For privacy, **no addresses or transaction identifiers are published** here or in the submission video, which uses an obvious placeholder address (and blurs the link text).
- **Demo addresses:** the sample profile ships with an empty payout address; the submission video uses a clearly fake placeholder. **Never publish a real wallet address in screenshots** you are not comfortable sharing.

---

## Authorship signature in detail

**Goal:** let a pseudonym be bound to a key, so that "the same hand" behind a pseudonym can be recognised later, *without* revealing a real identity.

**How it works (real, in-browser):**

1. `crypto.subtle.generateKey` → **ECDSA P-256** key pair.
2. A message is signed with **SHA-256**: `wanzek:<pseudonym>:<ISO timestamp>`.
3. WanZek stores the **public key (JWK), message, signature and a fingerprint** (first 16 hex characters of the SHA-256 of the raw public key) in the browser.
4. **Check signature** re-verifies the stored signature with the stored public key.
5. The **private key is displayed once** (copy it and keep it safe) and is **never stored** by the app. If it is lost, the pseudonym cannot be re-proven with that signature.

**What it proves — and what it does not:**

- ✅ Whoever holds the private key signed that pseudonym claim at that time.
- ❌ It does **not** prove real-world identity, and it does **not** prevent someone else from creating a different key for the same pseudonym.
- ❌ There is **no public registry** and no third-party verification in this demo; verification happens locally against the locally stored public key.

A production version would publish the public key with the profile (and ideally anchor it somewhere tamper-evident) so that clients can verify it independently.

---

## Order lifecycle and rules

```text
Awaiting payment ──(I've sent the payment)──► Verifying payment ──(demo: payment received)──► Payment held
        │                                                                                         │
        └── Cancel (no funds involved)                                                            ├── Artist: Mark as delivered ──► Delivered (work locked, 5-day window)
                                                                                                  │                                   ├── Client: Confirm receipt ──► Completed
                                                                                                  │                                   └── No confirmation in 5 days ──► Expired
                                                                                                  └── Client: Cancel ──► Cancelled
```

**Rules shown to users (and accepted before sending an order):**

1. **Shielded Zcash only.** Payments are requested only to shielded addresses.
2. **Delivery date** is agreed in every order.
3. **Paid in advance and held** until the client confirms delivery.
4. **The work is revealed on confirmation.** The client has at most **5 days** from delivery to confirm.
5. **No confirmation in 5 days:** the work returns to the artist; the held amount is split **50 % artist / 50 % client**.
6. **Cancellation:** free within **36 hours of payment**. After that, **10 %** of the agreed value is deducted and goes to the artist (of which **1 % of the total** is the platform fee, so the artist receives 9 %).
7. **Platform fee:** **5 %** of each completed transaction.
8. **Data:** no personal data is requested from the artist; clients leave their details so problems can be resolved.
9. **Demo:** custody, deadlines and refunds are a simulation in this version.

> The fee model and rules illustrate a **possible** marketplace design. WanZek in this repository is **not commercial** and charges nothing.

---

## Privacy model and data handling

**Where data lives**

| Data | Stored | Transmitted |
|---|---|---|
| Artist profile, links, privacy toggles | Browser `localStorage` (`wanzek-v1`) | No |
| Client account (name, email, phone, type) | Browser `localStorage` | No |
| Orders and demo-clock offset | Browser `localStorage` | No |
| Authorship public key + signature | Browser `localStorage` | No |
| Authorship **private key** | **Never stored** (shown once) | No |
| Uploaded works | In memory only (lost on reload) | No |
| Language preference | Browser `localStorage` (`wanzek-lang`) | No |

**Network requests the page makes**

- `cdnjs.cloudflare.com` — the QR library (`qrcodejs@1.0.0`, pinned).
- `fonts.googleapis.com` / `fonts.gstatic.com` — optional web-font fallback. The fonts are **also embedded** in the file, so the page renders correctly without them.
- No analytics, no cookies, no trackers, no `fetch`/XHR to any WanZek server. You can confirm this in your browser's network tab.

**What WanZek never does**

- It **never asks for seed phrases or wallet private keys.**
- It does not hold or move funds.

**Honest limits of the privacy claim**

- Zcash shielded transactions hide sender, receiver and amount on-chain, but **off-chain metadata** (timing, how you share the payment link, IP addresses, screenshots) can still identify people. WanZek does not claim absolute anonymity.
- A pseudonym protects identity only as long as the artist keeps their private life and the key separate from it.
- Because this is a static demo, **there is no server-side privacy guarantee to make** — and none is claimed.

---

## Tech stack and architecture

- **One static file**: `index.html` (HTML + CSS + vanilla JavaScript). **No build step, no framework, no backend.**
- **WebCrypto** (`crypto.subtle`) for ECDSA P-256 — requires a secure context (HTTPS or `localhost`).
- **`qrcodejs` 1.0.0** (MIT) loaded from cdnjs for the QR code.
- **Fonts, embedded as subsetted WOFF:** *Erica One* (logotype), *Big Shoulders* (headlines), *Instrument Sans* (text), *DM Mono* (labels).
- **Images:** four artworks by Senk, embedded as data URIs (about 1.7 MB total file size).
- **State:** a single JavaScript object mirrored to `localStorage`; views are rendered from it.
- **Internationalisation:** a dictionary-based Spanish → English layer applied to the rendered DOM.

---

## Run it / deploy it

**Locally**

```bash
# any static server works, for example:
python3 -m http.server 8000
# then open http://localhost:8000
```

Opening the file directly also works in most browsers, but WebCrypto is most reliable on `localhost` or HTTPS.

**Deploy:** upload `index.html` to any static host (Vercel, Netlify, GitHub Pages, Cloudflare Pages). No configuration needed.

**Reset the demo:** clear the site's `localStorage` (or use a private window).

**Browser support:** current Chromium-based browsers, Firefox and Safari with WebCrypto. The page's default language is Spanish; use the **EN** button for English.

---

## Testing and verification

- The full flow — *visitor → client sign-up → order → payment request with QR → artist delivery → client confirmation → completed*, in both languages — was exercised with **scripted headless-Chromium runs** during development.
- The layout was checked for **horizontal overflow** at 320, 390, 768 and 1280 px widths.
- The **payment-request construction was tested with a real shielded payment** (see [Payment flow](#payment-flow-in-detail)).
- There is **no automated test suite in this repository**, and no formal security or accessibility audit.

**Verify the claims yourself**

- *Payment request:* create an order → copy the link → open it in a Zcash wallet and compare address, amount and memo.
- *Signature:* My studio → *Create my authorship key* → *Check signature*.
- *No network calls:* open the browser's network tab while using the app.
- *Metadata stripping:* upload a photo that has EXIF data, then inspect the displayed image — the re-encoded file carries none.

---

## Known limitations

- **No backend:** no accounts shared between devices, no persistence beyond the visitor's browser.
- **No payment detection or custody** — see [Simulated](#-simulated-looks-and-behaves-like-the-real-thing-but-no-real-backend-or-funds).
- **Address validation is format-only** (prefix and character set), not a checksum validation.
- **A single sample artist** is shipped; more artists and artworks would need real, permissioned content.
- **Signature verification is local** and there is no registry of public keys.
- **Wallet compatibility** with `zcash:` links and memo pre-fill varies.
- **Not audited.** Do not use this code to handle real funds.

---

## Roadmap (ideas, not promises)

- Detect payments automatically (e.g., a lightwalletd-backed service using a viewing key, or an in-browser light client).
- A real escrow design (custodian, threshold signatures, or another mechanism) with explicit trust assumptions.
- Shared accounts and orders via a minimal backend; publish artists' public keys with their profiles.
- Offer accept/decline, messaging, and video upload.
- Address validation with checksum, and wallet-compatibility notes.
- More artists, more styles, and original high-resolution photography with permission.

---

## Credits and licenses

- **Artworks and photographs:** © **Senk (Zurdo Caribe)** and the **Caribes crew** — shown with their permission for this demo. They are **not** covered by the code license; please do not reuse them without permission. They will be removed on request.
- **Submission video:** uses footage of Senk's work and screen recordings of this demo; the soundtrack is an original instrumental generated for the project.
- **Code:** MIT License (see `LICENSE`).
- **Fonts:** Erica One, Big Shoulders, Instrument Sans and DM Mono, each under the **SIL Open Font License 1.1** (see each project for its license text).
- **QR library:** [`qrcodejs`](https://github.com/davidshimjs/qrcodejs) 1.0.0 (MIT).
- **Standards:** [ZIP-321: Payment Request URIs](https://zips.z.cash/zip-0321).

---

*WanZek is a demo created for the Zecathon by ZKSNARKS. It is not a commercial service, does not process real payments, and makes no guarantee of anonymity.*
