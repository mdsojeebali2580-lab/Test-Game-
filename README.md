# Puzzle Quest — Full Telegram Mini App project

Premium 3×3 sliding puzzle, Telegram authentication, SQLite database, server-side score records, leaderboard, Monetag TMA rewarded-interstitial integration with server postback verification, admin dashboard, and basic privacy/terms pages.

> This is a deployable starter, not a guarantee of Monetag/Adsterra approval or account safety. Networks make independent moderation decisions. Do not turn on ads until the actual Mini App/domain is approved.

## Stack
- Node.js 20+
- Express API
- SQLite (`better-sqlite3`) with WAL
- Telegram Mini App `initData` HMAC validation
- Basic-auth protected admin panel

## 1. Run locally
```bash
cp .env.example .env
# Edit .env; set BOT_TOKEN and strong unique secrets
npm install
npm start
```
Open `http://localhost:3000` for browser preview. Telegram login and database score submission require launching the deployed HTTPS app from Telegram. For real Telegram testing, use an HTTPS tunnel or deployment.

## 2. Deploy
Deploy the entire project to a Node.js host that supports persistent disk storage (Render persistent disk, VPS, etc.). Set all `.env.example` values as environment variables. Ensure `DB_FILE` points to a persistent writable volume; the default `data.sqlite` sits in the app folder and may be lost on ephemeral hosts/redeploys. Terminate HTTPS at the host/proxy. Do not commit `.env`, database files, bot token, admin password or postback secret.

Set `BASE_URL` to the public HTTPS origin. Create a Telegram bot using @BotFather, then configure the Mini App URL to `https://your-domain/`. Do not put the bot token in frontend code.

## 3. Admin panel
Visit `https://your-domain/admin`. Browser will request HTTP Basic Authentication:
- Username: `ADMIN_USERNAME`
- Password: `ADMIN_PASSWORD`

Change both from the sample values before deployment. Dashboard includes user/game/ad event summaries, recent events, CSV export, and Monetag zone settings.

## 4. Monetag for Telegram Mini Apps — supported path
Monetag publishes a dedicated Telegram Mini App SDK with three formats: Rewarded Interstitial, Rewarded Popup and In-App Interstitial. This project implements the optional **Rewarded Interstitial** path. It:
1. Creates a unique `ymid` record on the backend before showing the ad.
2. Calls the official SDK `show_<MAIN_ZONE_ID>({ ymid, requestVar })`.
3. Does not grant points from a client-side click or merely from a resolved Promise.
4. Grants the bonus only after a server-to-server Monetag postback says `reward_event_type=valued`.
5. Uses a database status transition keyed by unique `ymid` so postback retries cannot credit the same event twice.

### Configure
A. Get your Telegram Mini App approved/added in your Monetag publisher dashboard. Obtain the **main SDK zone ID** and the exact SDK script URL/tag from the dashboard. Do not guess or substitute an SDK URL.

B. Set `MONETAG_POSTBACK_SECRET` in the server environment to a long random secret.

C. In the admin panel, set:
- Monetag main zone ID
- Official SDK script URL
- Enable rewarded points (only after the postback is configured and tested)

D. In Monetag SSP, open the main SDK zone's Postback URL field and use this exact pattern, replacing the origin and secret:
```text
https://YOUR_DOMAIN/api/postback/monetag?secret=YOUR_LONG_SECRET&ymid={ymid}&zone_id={zone_id}&event_type={event_type}&reward_event_type={reward_event_type}&estimated_price={estimated_price}
```
The secret must match `MONETAG_POSTBACK_SECRET`. Keep the endpoint HTTPS and do not add redirects. The app uses a unique server-created UUID for each ad attempt. Postbacks for `valued` events credit once; `non_valued` events do not.

E. Test inside the actual Telegram Android/iOS WebView using a real approved ad zone. Check Monetag postback logs and the Admin panel. A successful ad load is not proof of a paid/valued event.

## 5. Adsterra — do not force or incentivize ads
This project deliberately does **not** auto-inject Adsterra Popunder, Social Bar, Direct Link or other generic website tags into the Telegram Mini App. Adsterra's published publisher guidance says traffic must be real/human and that bot or incentivized traffic is prohibited; its publisher terms also restrict use of tags to approved publisher websites and prohibit artificial/fraudulent impressions, clicks, incentives, proxies and emulated devices. Adsterra should not be used to reward users for viewing/clicking ads unless Adsterra gives you explicit written approval for the exact format and reward model.

Before adding Adsterra:
1. Ask Adsterra support in writing whether your specific Telegram Mini App URL/domain and its in-app WebView inventory are eligible.
2. Ask which exact ad format is permitted in a Telegram Mini App and where its code may be placed.
3. Submit and obtain approval for the exact domain/app in the publisher dashboard.
4. Use only the exact tag/code generated for the approved website/app and format; do not modify it, duplicate it, hide it, wrap it in an iframe or place it on an unapproved domain.
5. If they approve a non-incentivized banner placement, add the exact supplied code only to the visible `#adsterra-slot` container, as directed by Adsterra support. Do not put it behind a button or disguise it as game controls.
6. Never ask users to click ads, never pay points for Adsterra impressions/clicks, and never use bot traffic, traffic exchanges, proxies, emulators or purchased incentivized traffic.

If Adsterra does not confirm TMA compatibility in writing, leave the placeholder in place and monetize through an approved Monetag TMA format instead.

## 6. Publisher-compliance checklist (risk reduction, not a guarantee)
- Get written approval for the app/domain and every ad format before going live.
- Send real users from disclosed, legitimate sources. Keep analytics screenshots and a list of traffic source URLs.
- Do not buy incentivized traffic, use traffic exchanges, bot farms, VPN/proxy/emulator traffic, auto-refreshes, auto-clicks, hidden frames, forced redirects or fake engagement.
- Never ask or pressure users to click ads. Do not make game access conditional on clicking an ad.
- Clearly label ads as sponsored; keep ads visually separate from game controls and rewards.
- Do not show more than one copy of a zone tag or repeatedly load scripts.
- Keep the puzzle app fast, responsive, functional and free from deceptive buttons, misleading claims, illegal content and malware.
- Provide a privacy policy and disclose third-party advertising/data processing.
- Review each provider's current Terms and policies regularly; policies and eligible ad formats can change.
- Keep publisher account information accurate and respond honestly to support/moderation questions.

## 7. Important limitations
- No implementation can guarantee a publisher account will never be suspended. Only the networks decide approval, traffic quality and account status.
- Points are in-game points with no cash value; this project has no withdrawals or cash-out.
- Telegram authentication is HMAC-validated server-side and the bot token stays on the server.
- Game completion values are basic server-validated bounds, not a fully tamper-proof anti-cheat engine. Do not attach cash value to scores without stronger game-session validation, anti-abuse controls and manual review.
- Admin panel uses HTTP Basic Auth; use HTTPS, a long unique password, and preferably protect `/admin` with an additional IP allowlist or VPN.
- SQLite is suitable for a small initial app. For high concurrency, move to managed PostgreSQL and add backups/monitoring.

## 8. Project structure
- `server.js` — API, Telegram auth, database, postback, admin endpoints
- `public/index.html`, `public/app.js`, `public/style.css` — Telegram game
- `public/admin.html`, `public/admin.js`, `public/admin.css` — admin panel
- `public/privacy.html`, `public/terms.html` — policy pages
- `.env.example` — environment template

## Official references reviewed
- Telegram Mini Apps: https://core.telegram.org/bots/webapps
- Monetag TMA Application Requirements: https://docs.monetag.com/docs/introduction/application-requirements/
- Monetag TMA Moderation Process: https://docs.monetag.com/docs/introduction/moderation-process/
- Monetag SDK Integration: https://docs.monetag.com/docs/ad-integration/sdk-integration/
- Monetag Rewarded Interstitial: https://docs.monetag.com/docs/ad-integration/rewarded-interstitial/
- Monetag Postback Configuration: https://docs.monetag.com/docs/postbacks/configuration/
- Monetag invalid-traffic suspension guidance: https://help.monetag.com/en/articles/6738465-why-was-my-account-suspended
- Adsterra Publisher Requirements: https://adsterra.com/blog/set-up-publishers-dashboard/
- Adsterra Publisher Terms: review the current Publisher Terms linked in your Adsterra publisher account before activation; Adsterra's public anti-fraud guidance: https://adsterra.com/blog/adsterra-ad-fraud-prevention-methods/
