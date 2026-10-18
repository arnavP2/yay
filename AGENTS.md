<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

## Architecture
- Cargo volume is optional in load storage and validated as positive finite cubic metres; it does not change matching without truck volume-capacity data — why: preserve older loads and avoid inventing capacity guarantees.
- Broadcast and tracking maps share the browser-only Google Maps loader in `src/lib/google-maps.ts`; invoke it only from effects — why: avoid duplicate scripts and preserve SSR-safe map initialization.
- Intercity map traffic is a presentation-only simulation sourced from mock data, separate from vendor eligibility and booking state — why: illustrative movement must never imply live GPS or change who receives a load.
- All mock data lives in `src/data/mock.ts` and matching rules in `src/lib/matching.ts` (tested); app state is a single React context in `src/lib/store.tsx` — why: easy swap to a Flask REST API later.
- Shared multi-device state lives in the connected backend tables `bh_loads`, `bh_load_statuses`, `bh_quotes`, `bh_bookings`, `bh_events`; every mutation goes through a server function in `src/lib/app.functions.ts` and the store re-syncs via realtime — why: a vendor on another device must see the shipper's actions live. Vendors are identified per device by phone match (no auth) stored in localStorage; vendor identities stay in `src/data/mock.ts`, never in the database.
- Public database reads and realtime for the shared demo board are limited to the latest load and its related rows using `backhaul_private.bh_current_load_id()`; keep this fixed-search-path helper outside exposed API schemas — why: preserve phone-match cross-device updates without exposing historical records or causing recursive load policies.
- The WhatsApp bot is a per-phone state machine in `src/lib/whatsapp-bot.ts` (sessions in `bh_wa_sessions`, message log in `bh_wa_messages`, both service-role only); Twilio calls `src/routes/api/public/whatsapp/webhook.ts`, replies go out via the Twilio connector gateway in `src/lib/whatsapp.ts` — why: webhook replies must be async API sends so vendor broadcasts and shipper notifications work outside the request/response turn. The sender number is `TWILIO_WHATSAPP_FROM` (defaults to the Twilio sandbox).
- Registered users live in `bh_users` (phone PK, role vendor/shipper, region), trucks in `bh_trucks` (region, type, capacity, optional volume), per-load auction config in `bh_auction_settings`; all writes go through `src/lib/users.functions.ts` server functions — why: phone-match demo has no auth, so the service role is the only writer.
- Auction bids reuse `bh_quotes`; the undercut rule (bid ≤ lowest − min decrement) is pure logic in `src/lib/auction.ts` with tests — why: one tested rule shared by server enforcement and UI hints.
- The AI assistant (`src/lib/assistant.functions.ts`, route `/assistant`) calls the Lovable AI Gateway with `openai/gpt-6-astra` via AI SDK Responses — why: user asked for Grok but no Grok/xAI model exists in the gateway catalog; default model is the fallback.
