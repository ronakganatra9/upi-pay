# UPI Pay

> A lightweight, mobile-first Progressive Web App (PWA) for scanning UPI QR codes, entering a payment amount, and launching a standard UPI payment intent.

[![PWA](https://img.shields.io/badge/PWA-installable-111827)](#install-as-a-pwa)
[![Vanilla JS](https://img.shields.io/badge/JavaScript-vanilla-F7DF1E)](#tech-stack)
[![No Backend](https://img.shields.io/badge/backend-none-2ea44f)](#architecture)

## Overview

UPI Pay is a static, backend-free web app designed for mobile devices. It can:

- Scan a compatible UPI QR code using the device camera.
- Parse the recipient UPI ID and merchant name from the QR payload.
- Accept a payment amount.
- Automatically prepare a single payment for amounts up to ₹2,000.
- Automatically prepare a multi-payment sequence for amounts above ₹2,000, using ₹1,999.99 installments and the exact remaining balance for the final installment.
- Launch one standard `upi://pay` intent through a single **Pay via any UPI App** button.
- Let the operating system/UPI environment present compatible installed UPI apps where supported.
- Track installment progress and ask the user to confirm completion after returning from the payment app.
- Keep local payment history in browser storage.
- Install as a PWA on supported browsers.

> **Important:** Every installment is a separate UPI transaction. This project does not receive or verify bank/payment-provider transaction status. The completion state shown by the app is user-confirmed local state.

## Screens / Flow

```text
Scan QR
   ↓
Recipient confirmation
   ↓
Enter amount
   ↓
┌──────────────────────────┐
│ Amount ≤ ₹2,000          │ → One payment
└──────────────────────────┘

┌──────────────────────────┐
│ Amount > ₹2,000          │ → Automatic installments
└──────────────────────────┘
   ↓
Pay via any UPI App
   ↓
UPI app chooser / compatible UPI app
   ↓
Return to UPI Pay
   ↓
Confirm completion
   ↓
Next installment, if any
```

## Automatic payment calculation

The current UI uses this deterministic calculation:

- `₹2,000` or less → one payment.
- Above `₹2,000` → repeated `₹1,999.99` installments until the remaining amount is less than `₹1,999.99`.
- The final installment is the exact remaining amount.

Examples:

| Total | Automatic sequence |
|---:|---|
| ₹1,500 | ₹1,500 |
| ₹2,000 | ₹2,000 |
| ₹2,000.01 | ₹1,999.99 + ₹0.02 |
| ₹5,000 | ₹1,999.99 + ₹1,999.99 + ₹1,000.02 |
| ₹10,000 | ₹1,999.99 × 5 + ₹0.05 |

These are separate payment transactions, not one transaction divided internally.

## Tech stack

- HTML5
- CSS3
- Vanilla JavaScript
- Web Camera API (`getUserMedia`)
- `jsQR` for QR decoding
- Web App Manifest
- Service Worker
- Browser `localStorage`
- Standard `upi://pay` intent

There is no database, API server, authentication system, or payment backend.

## Project structure

```text
.
├── index.html
├── app.js
├── style.css
├── manifest.json
├── sw.js
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
├── README.md
├── LICENSE
├── SECURITY.md
└── .gitignore
```

## Run locally

You can run the static site with Python:

```bash
python3 -m http.server 8080
```

Open:

```text
http://localhost:8080
```

For camera access and PWA installation on a phone, use an HTTPS URL. `localhost` is treated as a secure development origin by browsers, but a phone accessing your Mac over a normal LAN HTTP address is not the same secure-origin setup.

## Test on a phone

For quick local testing, use an HTTPS tunnel such as Cloudflare Tunnel:

```bash
cloudflared tunnel --url http://localhost:8080
```

Open the generated `https://*.trycloudflare.com` URL on the phone.

## Deploy to Cloudflare Pages

This is a static site, so there is no build process.

For a Git-connected Cloudflare Pages project, use:

```text
Framework preset: None
Build command: exit 0
Build output directory: .
```

The directory containing `index.html` must be the deployed output/root. If `index.html` is nested inside another directory, the root `*.pages.dev` URL can return 404.

## Install as a PWA

### Android / Chrome

Open the HTTPS deployment URL and choose **Install app** or **Add to Home screen** when offered.

### iOS / Safari

Open the HTTPS deployment URL in Safari, then use **Share → Add to Home Screen**.

## UPI intent behavior

The app intentionally uses one generic payment button instead of hard-coding individual UPI apps:

```text
upi://pay?pa=merchant@upi&pn=Merchant&am=1999.99&cu=INR
```

The operating system/browser/UPI environment decides how the intent is handled and which compatible UPI apps can be offered.

Browser behavior differs between Android versions, browsers, installed UPI apps, and device manufacturers. The project therefore should be tested on the target Android devices before production use.

## Payment verification limitation

A browser PWA cannot universally receive a trusted payment-success callback from every UPI app. The app therefore does **not** claim that a payment was verified by a bank or PSP. It records the user's explicit completion choice locally.

For production-grade automatic reconciliation, a merchant/payment-provider backend and an appropriate payment-status API/webhook would be required.

## Privacy

The application does not have a backend and does not intentionally transmit UPI IDs, amounts, or local payment history to a project server.

Browser storage is local to the user's device/browser. Clearing site data can remove local history.

The QR decoding library is currently loaded from jsDelivr. If a fully self-contained/offline distribution is required, vendor the library locally instead of using the CDN.

## Security considerations

Do not enter UPI PINs, card details, bank passwords, OTPs, or other credentials into this application. A UPI PIN should only ever be entered inside the user's trusted UPI application.

Always verify the recipient and amount before authorizing a payment in the UPI app.

## Contributing

Pull requests and issue reports are welcome. Keep the project backend-free unless a backend is explicitly introduced for a documented feature such as payment reconciliation.

## License

Released under the MIT License. See [`LICENSE`](LICENSE).
