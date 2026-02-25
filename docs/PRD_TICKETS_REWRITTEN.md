# PadhAI UPSC — PRD for Devs (Rewritten Tickets)

All tickets rewritten with a consistent structure: **Title**, **Description** (numbered sections), **Acceptance**. Use the ticket **Title** as the issue title and **Description** plus **Acceptance** as the issue description in Linear (or your tracker).

---

## Event-to-flow reference (unchanged)

| Flow | Trigger | Schedule (WA / SMS) |
|------|---------|---------------------|
| Payment Failure | `payment_failed_screen_shown` or gateway failure callback. Emit to backend. | WA instant, SMS +1h, WA +24h. Cancel rest if user pays later. |
| Payment Success | `payment_success_screen_shown` or gateway success. Emit `subscription_activated` with duration. | WA instant + SMS instant, then WA Day 3, WA Day 7. |
| Subscription Expiry | `subscription_ended` (backend when period ends). | WA -2d, SMS -1d, WA instant, SMS +24h, WA +48h. |
| Cancellation | `subscription_cancelled` or checkout drop-off (no success within e.g. 4h). | WA +4h, SMS +24h, WA +48h. |
| Onboarding Drop-Off | Signup but no second `app_open` or no `trial_activated` by +6h. | WA +6h, SMS +24h, WA +48h. |
| Short-Term Win-Back | No `app_open` / engagement for 7 days (`last_activity`). | WA Day 7, SMS Day 10, WA Day 14. |
| Weekly to Annual | `subscription_activated` with duration = weekly. | WA Day 1, SMS Day 3, WA Day 6 (within first 7 days). |
| Long-Term Win-Back | 30d inactive, or app update day, or 180+ days. | Monthly: WA Day 30, SMS +24h, WA +48h. Update: WA, SMS +24h, WA +48h. Final: WA only. |
| Feature Paywall (Phase 2) | `feature_paywall_abandoned` with feature (news \| pyq \| ai_explain \| mock). | WA +3h (feature-specific), SMS +24h, WA +48h. |

---

## Phase 1 — Notifications

### Phase 1 · P0-1 — Payment Failure

**Title**  
`[P0] Payment Failure — Register templates, CleverTap journey, event ingestion and delivery`

**Description**

1. **Template registration (priority 1)**  
   Register and get approval for Payment Failure templates.  
   - **WhatsApp (Meta utility):** instant and +24h. Use exact copy from Template Library; all links in copy must use variable `{link}`.  
   - **SMS (TRAI DLT, PDAIUP-S Service Implicit):** +1h. Submit DLT first (longer lead time), then Meta.

2. **Backend and events**  
   When payment fails (app event `payment_failed_screen_shown` or web/app payment gateway failure callback), do the following.  
   - Persist the event.  
   - Send user identity and event payload to CleverTap so the journey can trigger.  
   - Ensure gateway failure from PhonePe (web) and PhonePe/Google Play (app) both result in the same event or equivalent payload to backend.  
   - Cancel any scheduled messages for this user if they complete payment later (e.g. on `payment_success` or `subscription_activated`).

3. **CleverTap journey**  
   Build one journey for Payment Failure.  
   - **Trigger:** user enters segment or receives event (payment failed).  
   - **Steps:** send WA instant (approved template, CTA `{link}`); wait 1 hour; send SMS (+1h template); wait 23 hours; send WA (+24h template).  
   - **Exit condition:** user completes payment (`subscription_activated` or `payment_success`).  
   Connect WA Business API and SMS gateway to CleverTap. Respect opt-out and DND before each send.

**Acceptance**

- WA and SMS templates for Payment Failure submitted and approved (DLT + Meta).  
- Backend emits/pushes failure event to CleverTap; app and web failure both trigger the same flow.  
- CleverTap journey sends WA instant, SMS +1h, WA +24h; journey exits on payment success.  
- Links use `{link}`; opt-out/DND honoured.

---

### Phase 1 · 1 — Payment Success

**Title**  
`Payment Success — Register templates, CleverTap journey, dual message (WA + SMS instant) and Day 3 / Day 7`

**Description**

1. **Template registration (priority 2)**  
   Register and get approval for Payment Success templates.  
   - **WhatsApp (Meta utility):** instant welcome, Day 3, Day 7. Copy from Template Library; links as `{link}`.  
   - **SMS (TRAI DLT PDAIUP-S):** instant. Submit DLT then Meta.

2. **Backend and events**  
   On payment success (`payment_success_screen_shown` or gateway success callback from PhonePe/Google Play):  
   - Persist subscription and emit `subscription_activated` with duration (weekly or target_plan).  
   - Send user identity and event to CleverTap so the journey can trigger.  
   - Ensure both app and web success paths result in the same event or equivalent payload.

3. **CleverTap journey**  
   Build one journey for Payment Success.  
   - **Trigger:** `subscription_activated` (or equivalent success event).  
   - **Steps:** send WA instant (welcome template, CTA `{link}`); send SMS instant (same user, approved DLT template with `{link}`); wait until Day 3 from subscription start; send WA (Day 3 template); wait until Day 7; send WA (Day 7 template).  
   - No exit condition for Day 3/7 once started; optional exit if user unsubscribes.  
   Connect WA and SMS to CleverTap; respect opt-out/DND.

**Acceptance**

- WA and SMS templates for Payment Success submitted and approved.  
- On success, backend persists subscription and pushes event to CleverTap.  
- CleverTap sends 2 messages immediately (WA + SMS), then WA Day 3 and WA Day 7.  
- `subscription_activated` emitted with duration. All links `{link}`.

---

### Phase 1 · 2 — Subscription Expiry

**Title**  
`Subscription Expiry — Register templates, CleverTap journey, pre- and post-expiry sequence`

**Description**

1. **Template registration (priority 3)**  
   Register Subscription Expiry templates.  
   - **WhatsApp (Meta utility):** -2d, instant at expiry, +48h. Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** -1d, +24h. Submit DLT then Meta.

2. **Backend**  
   When a subscription period ends (weekly or annual):  
   - Emit `subscription_ended` with subscription type and end date.  
   - Subscription end date must be available (e.g. from billing or entitlement store) so CleverTap can schedule -2d and -1d relative to that date.  
   - Send event and user to CleverTap.

3. **CleverTap journey**  
   Build one journey for Subscription Expiry.  
   - **Trigger:** `subscription_ended` or scheduled trigger based on known end date.  
   - **Steps:** at T-2 days send WA (renew in 2 days); at T-1 day send SMS (renew tomorrow); at T (expiry) send WA instant (expired, renew); wait 24h send SMS (+24h); wait 24h send WA (+48h).  
   Use approved templates; all links `{link}`. Respect opt-out/DND.

**Acceptance**

- Templates for Subscription Expiry submitted and approved.  
- Backend emits `subscription_ended` with end date.  
- CleverTap sends WA -2d, SMS -1d, WA instant, SMS +24h, WA +48h at correct times.

---

### Phase 1 · 3 — Cancellation Recovery

**Title**  
`Cancellation Recovery — Register templates, CleverTap journey, trigger on cancel or checkout drop-off`

**Description**

1. **Template registration (priority 4)**  
   Register Cancellation Recovery templates.  
   - **WhatsApp (Meta utility):** +4h, +48h. Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** +24h. Submit DLT then Meta.

2. **Backend and events**  
   - Emit `subscription_cancelled` when user cancels from profile or when cancellation is confirmed via PhonePe/Play backend.  
   - For checkout drop-off: if user reached payment (e.g. `subscribe_now_clicked` or `payment_inprogress`) but never `payment_success_screen_shown` within threshold (e.g. 4 hours), treat as cancellation and emit same or equivalent event to CleverTap.  
   - Send user and event to CleverTap.

3. **CleverTap journey**  
   Build one journey for Cancellation.  
   - **Trigger:** `subscription_cancelled` (or checkout drop-off event).  
   - **Steps:** wait 4h; send WA (+4h template); wait 20h; send SMS (+24h); wait 24h; send WA (+48h).  
   - **Exit:** if user completes payment (`subscription_activated`).  
   Use approved templates; links `{link}`; respect opt-out/DND.

**Acceptance**

- Templates for Cancellation submitted and approved.  
- Explicit cancel and checkout drop-off (after threshold) both trigger journey.  
- CleverTap sends WA +4h, SMS +24h, WA +48h; journey exits on payment success.

---

### Phase 1 · 4 — Onboarding Drop-Off

**Title**  
`Onboarding Drop-Off — Register templates, CleverTap journey, trigger on signup without engagement`

**Description**

1. **Template registration (priority 5)**  
   Register Onboarding Drop-Off templates.  
   - **WhatsApp (Meta utility):** +6h, +48h. Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** +24h. Submit DLT then Meta.

2. **Backend and definition**  
   - Define signup (e.g. account created timestamp or first `app_open`).  
   - By +6 hours from signup, if there is no second `app_open` (or no `trial_activated`, or no meaningful engagement as defined), push user and event to CleverTap to start this flow.  
   - Ensure CleverTap can segment or receive “signup, no engagement by +6h” users.

3. **CleverTap journey**  
   Build one journey for Onboarding Drop-Off.  
   - **Trigger:** user in segment “signed up, no second session / no trial by +6h” (or equivalent event).  
   - **Steps:** at +6h send WA; wait 18h send SMS (+24h); wait 24h send WA (+48h).  
   - **Exit:** if user opens app or activates trial (if applicable).  
   Use approved templates; links `{link}`; respect opt-out/DND.

**Acceptance**

- Templates for Onboarding Drop-Off submitted and approved.  
- Users who sign up and do not engage by +6h receive WA +6h, SMS +24h, WA +48h.  
- Journey exits on engagement.

---

### Phase 1 · 5 — Short-Term Win-Back

**Title**  
`Short-Term Win-Back — Register templates, CleverTap journey, trigger on 7-day inactivity`

**Description**

1. **Template registration (priority 6)**  
   Register Short-Term Win-Back templates.  
   - **WhatsApp (Meta utility):** Day 7, Day 14. Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** Day 10. Submit DLT then Meta.

2. **Backend and data**  
   - Ensure `last_activity` or last `app_open` timestamp is available per user (e.g. from app events or CleverTap).  
   - CleverTap must be able to segment users whose `last_activity` is exactly 7 days ago (or 10 and 14 for subsequent steps).  
   - Dedupe so a user does not re-enter this flow multiple times for the same inactivity spell (e.g. once per 7-day-inactive entry).

3. **CleverTap journey**  
   Build one journey for Short-Term Win-Back.  
   - **Trigger:** user enters segment “last_activity 7 days ago” (and not already in this journey).  
   - **Steps:** send WA (Day 7); wait 3 days send SMS (Day 10); wait 4 days send WA (Day 14).  
   Use approved templates; links `{link}`. Respect opt-out/DND. Ensure no duplicate sequences per user per qualifying period.

**Acceptance**

- Templates for Short-Term Win-Back submitted and approved.  
- Users inactive 7 days get WA Day 7, SMS Day 10, WA Day 14.  
- No duplicate sequences per user per spell.

---

### Phase 1 · 6 — Weekly to Annual Upsell

**Title**  
`Weekly to Annual — Register templates, CleverTap journey, 3 messages in first 7 days for weekly subscribers`

**Description**

1. **Template registration (priority 7)**  
   Register Weekly to Annual templates.  
   - **WhatsApp (Meta utility):** Day 1, Day 6. Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** Day 3. Submit DLT then Meta.

2. **Backend and events**  
   - When `subscription_activated` is emitted, include duration (weekly or target_plan).  
   - CleverTap must receive this so it can start the upsell journey only for users with duration = weekly.  
   - When user upgrades to annual (`subscription_activated` with duration = target_plan or equivalent), journey must exit and cancel remaining messages.

3. **CleverTap journey**  
   Build one journey for Weekly to Annual.  
   - **Trigger:** `subscription_activated` with duration = weekly.  
   - **Steps:** send WA (Day 1); wait 2 days send SMS (Day 3); wait 3 days send WA (Day 6). All within first 7 days of subscription.  
   - **Exit condition:** user upgrades to annual (duration change).  
   Use approved templates; links `{link}`; respect opt-out/DND.

**Acceptance**

- Templates for Weekly to Annual submitted and approved.  
- Only weekly subscribers get the sequence (Day 1, 3, 6); sequence stops when user upgrades to annual.

---

### Phase 1 · 7 — Long-Term Win-Back

**Title**  
`Long-Term Win-Back — Register templates, CleverTap journeys (Monthly, App Update, Final)`

**Description**

1. **Template registration (priority 8)**  
   Register Long-Term Win-Back templates (three sub-flows).  
   - **WhatsApp (Meta utility):** Day 30, +24h, +48h for Monthly; update day, +24h, +48h for App Update; 180+ days single WA for Final with KEEP/STOP.  
   - **SMS (TRAI DLT):** +24h for Monthly and App Update. Copy from Template Library; links `{link}`. Submit DLT then Meta.

2. **Backend and data**  
   - `last_activity` (or last `app_open`) must be available.  
   - For App Update sub-flow: app version or release event must be available so CleverTap can target “inactive users” at update release.  
   - For Final (180+ days): segment users inactive 180+ days; single WA only, no SMS.

3. **CleverTap journeys**  
   Build three sub-flows or one journey with branches.  
   - **(a) Monthly:** trigger when last_activity = 30 days ago; send WA Day 30, wait 24h send SMS, wait 24h send WA.  
   - **(b) App update:** trigger on app release; target inactive users; send WA on update day, SMS +24h, WA +48h.  
   - **(c) Final:** trigger when last_activity 180+ days; send single WA (KEEP/STOP).  
   Use approved templates; respect opt-out/DND.

**Acceptance**

- Templates for Long-Term Win-Back submitted and approved.  
- Monthly, App Update, and Final sub-flows run with correct triggers and schedules; last_activity and release event available.

---

## Phase 1 — Non-notification (Infrastructure & product)

### Phase 1 · 8 — Web landing and checkout

**Title**  
`Web landing and checkout (PhonePe only) — pages, auth, gateway, and CleverTap triggers`

**Description**

1. **Landing and plan selection**  
   - Build PadhAI Pro landing page with plan selection (e.g. weekly, annual).  
   - Display pricing and CTA to checkout.  
   - Ensure mobile and desktop usable.

2. **Auth and checkout**  
   - Implement OTP-based phone auth (or equivalent) so user is identified before payment.  
   - Capture verified phone for WA/SMS delivery and for CleverTap identity.  
   - Integrate PhonePe as the only payment method on web. Handle redirect to PhonePe and return to our domain.  
   - Capture success and failure callbacks from PhonePe; persist transaction result and user identity.

3. **Success and failure pages and CleverTap**  
   - **Success page:** On payment success callback, show confirmation page. Activate Premium for the user (see Subscription API ticket for sync to app). Emit or push event to CleverTap so Payment Success journey triggers (`subscription_activated` with duration). Ensure CleverTap receives user identity (e.g. phone/email) and event payload.  
   - **Failure / retry page:** On payment failure callback, show retry page with option to try again. Emit or push failure event to CleverTap so Payment Failure journey triggers. Persist failure event for analytics; do not block retry.  
   - All CTAs on success/failure pages that point to app or store must use the same `{link}` variable resolution as notification templates (backend resolves per context).

**Acceptance**

- User can complete or fail payment on web; success triggers Payment Success CleverTap flow; failure triggers Payment Failure CleverTap flow.  
- Gateway callbacks captured and persisted; CleverTap receives correct events and user identity.  
- OTP auth and PhonePe-only checkout working.

---

### Phase 1 · 9 — Subscription API and link variable resolution

**Title**  
`Subscription API and link variable resolution — Premium sync, {link} resolution, no raw deeplinks`

**Description**

1. **Premium activation and sync**  
   - On web payment success: backend must activate Premium for the user (entitlement or subscription record).  
   - Sync this state to the app (e.g. via API or existing user profile) so that when the user opens the app, Premium features are unlocked.  
   - Record `subscription_activated` with duration (weekly/target_plan), platform (web), and any required identifiers for CleverTap.

2. **Link variable resolution**  
   - Every notification (WA and SMS) uses the placeholder `{link}` in template copy; no raw deeplinks (e.g. padhai.app/...) in approved template text.  
   - Backend (or the system that injects content into CleverTap messages) must resolve `{link}` at send time to the correct destination: app deep link for in-app users, web checkout or app-store link for web payers or where appropriate.  
   - Resolution logic must account for context (e.g. flow type, user source) so the right URL is used.

3. **Consistency**  
   - Ensure all flows that send WA/SMS use the same resolution mechanism so links are correct for payment retry, renewal, onboarding, win-back, and upsell.

**Acceptance**

- Premium is activated and synced to app on web payment success.  
- All notification templates use `{link}` only; backend resolves to correct URL per context.  
- No raw deeplinks in template copy.

---

### Phase 1 · 10 — Opt-out and conversion tracking

**Title**  
`Opt-out and conversion tracking — STOP/KEEP, DND, and flow-level metrics`

**Description**

1. **Opt-out handling**  
   - Implement STOP/KEEP handling for WhatsApp (user replies STOP to unsubscribe from WA notifications).  
   - Respect DND for SMS.  
   - When user opts out, update preference in backend and in CleverTap so no further messages are sent for that channel.  
   - Before each send in CleverTap, check opt-out and DND status and skip if user has opted out.

2. **Conversion tracking**  
   - Ensure each notification flow is measurable for conversion.  
   - Use CleverTap (or existing analytics) to track: which users entered each journey, which step they received, and whether they converted (e.g. payment success, `subscription_activated`, `app_open`) after a message.  
   - UTM or equivalent parameters on `{link}` where applicable so traffic from WA vs SMS can be distinguished.  
   - No separate ticket for implementing Mixpanel; use CleverTap events and existing product analytics for flow performance.

**Acceptance**

- STOP/KEEP and DND honoured; opt-out state synced to CleverTap and backend.  
- Conversion and channel (WA vs SMS) measurable per flow via CleverTap or existing tooling.

---

## Phase 2 — Feature Paywall Abandonment (notification)

### Phase 2 · 1 — Emit feature_paywall_abandoned event

**Title**  
`Emit feature_paywall_abandoned event — app and backend`

**Description**

1. **Definition**  
   User is in “abandoned” state when they (a) saw a feature paywall modal (News MCQ/PYQ, PYQ answer verification, AI Explain blur, or Mock results), and (b) closed the app or navigated away without subscribing (no `payment_success` or `subscription_activated` after viewing the paywall).

2. **App**  
   - When the user dismisses the paywall (e.g. back, close, or leaves screen) without tapping subscribe or completing payment, emit event `feature_paywall_abandoned`.  
   - Payload must include: `feature` (string: news | pyq | ai_explain | mock), and any required user identity (e.g. CleverTap identity).  
   - Send to backend or directly to CleverTap per existing event pipeline.

3. **Backend**  
   - If events are routed via backend, persist or forward `feature_paywall_abandoned` to CleverTap with the same payload so the Phase 2 journey can trigger and can select feature-specific template variant.

**Acceptance**

- Event `feature_paywall_abandoned` fires when user sees paywall and leaves without subscribing.  
- Parameter `feature` is one of news, pyq, ai_explain, mock.  
- Event reaches CleverTap (or backend that forwards to CleverTap) with correct identity.

---

### Phase 2 · 2 — Feature Paywall Abandonment journey

**Title**  
`Feature Paywall Abandonment — Register templates (feature-specific), CleverTap journey`

**Description**

1. **Template registration**  
   Register Feature Paywall Abandonment templates (same pattern as Phase 1: register templates and build journey in one ticket).  
   - **WhatsApp (Meta utility):** +3h and +48h.  
   - **SMS (TRAI DLT PDAIUP-S):** +24h.  
   - Message copy must be feature-specific: four variants (News, PYQ, AI Explain, Mock) so the right copy is sent based on `feature_paywall_abandoned.feature`. Use Template Library for copy; all links as `{link}`. Submit DLT then Meta. If Meta/DLT require one template per variant, register all required variants.

2. **CleverTap journey**  
   Build one journey for Feature Paywall Abandonment.  
   - **Trigger:** event `feature_paywall_abandoned`.  
   - **Steps:** wait 3h; send WA using template variant that matches event.feature (news/pyq/ai_explain/mock); wait 21h; send SMS (+24h, feature-specific variant); wait 24h; send WA (+48h, feature-specific variant).  
   - Resolve `{link}` per user/context. Respect opt-out/DND. Exit if user subscribes (`subscription_activated`).

**Acceptance**

- Feature-specific WA and SMS templates for Feature Paywall submitted and approved.  
- CleverTap journey sends WA +3h, SMS +24h, WA +48h with copy that matches the feature user abandoned; links use `{link}`; journey exits on subscribe.

---

## Phase 2 — Product paywalls (non-notification)

### Phase 2 · 3 — Paywall UI — News (MCQ/PYQ lock)

**Title**  
`Paywall UI — News (MCQ/PYQ lock) — article free, tap MCQs/PYQs shows modal`

**Description**

1. **Free experience**  
   - News article is fully readable without subscription.  
   - All linked MCQs and PYQs related to the article are listed or tappable.

2. **Paywall trigger and modal**  
   - When user taps to open or attempt a linked MCQ or PYQ (beyond any free allowance if defined), show the upgrade modal.  
   - Modal copy must be feature-specific for News (e.g. “Unlock News MCQs and PYQs” / “Subscribe to practice MCQs linked to this article”). Do not show paywall for Duel (Duel remains free forever).  
   - Upgrade modal must include CTA to subscribe. Payment options: PhonePe or Google Play (in-app).  
   - On dismiss or back without paying, ensure `feature_paywall_abandoned` can fire with feature = news (see Phase 2 · 1).

3. **Shared components**  
   - Use shared upgrade modal and paywall event tracker from Phase 2 · 7 so feature and screen are recorded for CleverTap.

**Acceptance**

- Article free; tap on linked MCQ/PYQ shows paywall with News-specific copy; payment via PhonePe or Google Play; dismiss triggers `feature_paywall_abandoned` with feature=news. Duel not gated.

---

### Phase 2 · 4 — Paywall UI — PYQ (answer verification lock)

**Title**  
`Paywall UI — PYQ (answer verification lock) — Q1 free, Q2+ gated`

**Description**

1. **Free experience**  
   - First PYQ (Q1) in a session or list is fully free: question, options, and answer verification (check/correct) visible.  
   - User can attempt and verify one question without subscription.

2. **Paywall trigger and modal**  
   - From Q2 onward (or when user tries to view answer verification for the second question), show paywall overlay.  
   - Copy must be feature-specific for PYQ (e.g. “Verify answers on all PYQs with Premium”). Lock only the verification step; question and options may remain visible per product decision.  
   - Upgrade modal with PYQ-specific copy; payment via PhonePe or Google Play.  
   - On dismiss without paying, fire `feature_paywall_abandoned` with feature = pyq.

3. **Shared components**  
   - Use shared modal and event tracker; record feature=pyq.

**Acceptance**

- Q1 free with full verification; Q2+ verification gated; paywall shows PYQ-specific copy; payment options available; abandon triggers feature=pyq.

---

### Phase 2 · 5 — Paywall UI — AI Explain (blur)

**Title**  
`Paywall UI — AI Explain (blur) — first lines visible, rest blurred`

**Description**

1. **Free experience**  
   - When user asks for “Explain the answer” (or equivalent), show the first few lines of the AI-generated explanation in full.  
   - Rest of the explanation is blurred (or truncated) with a clear boundary.

2. **Paywall trigger and modal**  
   - User must subscribe to unlock the full explanation. Show subscribe CTA on the blur/truncation.  
   - Copy must be feature-specific for AI Explain (e.g. “Get full AI explanation with Premium”). Tapping “Subscribe” or “Unlock” opens upgrade modal.  
   - Upgrade modal with AI Explain-specific copy; payment via PhonePe or Google Play.  
   - On dismiss without paying (e.g. user leaves screen or closes app), fire `feature_paywall_abandoned` with feature = ai_explain.

3. **Shared components**  
   - Use shared blur/lock UI and modal; event tracker records feature=ai_explain.

**Acceptance**

- First few lines of explanation visible; rest blurred with subscribe CTA; payment unlocks full explanation; abandon triggers feature=ai_explain.

---

### Phase 2 · 6 — Paywall UI — Mock test results

**Title**  
`Paywall UI — Mock test results — full mock free, results behind paywall`

**Description**

1. **Free experience**  
   - User can take the full mock test (answer all questions) without subscription.  
   - Submission is allowed.

2. **Paywall trigger and modal**  
   - After submission, results (score, rank, topic-wise analysis, comparison) are behind a paywall.  
   - Show a results gate overlay (e.g. “Your results are ready — subscribe to view”) with feature-specific copy for Mock. User cannot see score/rank/analysis until they subscribe or have an active subscription.  
   - Upgrade modal with Mock-specific copy (e.g. “View your mock results and detailed analysis”); payment via PhonePe or Google Play.  
   - On dismiss or back without paying, fire `feature_paywall_abandoned` with feature = mock.

3. **Shared components**  
   - Use shared modal and event tracker; record feature=mock.

**Acceptance**

- Full mock free to take; results (score, rank, analysis) behind paywall; overlay and modal Mock-specific; abandon triggers feature=mock.

---

### Phase 2 · 7 — Shared paywall components

**Title**  
`Shared paywall components — modal, event tracker, blur/lock UI, gating`

**Description**

1. **Upgrade modal**  
   - Reusable modal component that displays feature-specific copy (News, PYQ, AI Explain, Mock). Copy and CTA text must be configurable by feature.  
   - Modal must include primary CTA to subscribe (PhonePe / Google Play) and dismiss/back.  
   - On show, modal should receive feature type so it can (a) render the right copy and (b) pass feature to the event tracker when user dismisses without paying.

2. **Paywall event tracker**  
   - When user hits a paywall (sees the modal or blur/CTA) and leaves without subscribing, record the feature (news | pyq | ai_explain | mock) and optionally screen/source.  
   - This data must be available to emit `feature_paywall_abandoned` (Phase 2 · 1) with the correct feature parameter.  
   - Integrate with app’s event pipeline so CleverTap or backend receives the event.

3. **Blur/lock UI and gating**  
   - **Blur/lock UI:** Reusable component for “partial content + blur + CTA” (used by AI Explain and any similar flows). Configurable line or character threshold for what is visible vs blurred. CTA triggers open of upgrade modal with the correct feature.  
   - **Free-tier state manager:** Central place or logic that determines whether the user has access to a given feature (News MCQ, PYQ verification, AI full explain, Mock results). Must integrate with subscription state (Premium active or not) so paywalls show only for free users and unlock after subscription.

**Acceptance**

- All four paywalls (News, PYQ, AI Explain, Mock) use the shared modal, event tracker, and gating logic; copy is feature-aware; dismiss without pay results in `feature_paywall_abandoned` with correct feature.

---

## Summary — Final ticket list

| # | Phase | Title (use as issue title) |
|---|--------|----------------------------|
| P0-1 | 1 | [P0] Payment Failure — Register templates, CleverTap journey, event ingestion and delivery |
| 1 | 1 | Payment Success — Register templates, CleverTap journey, dual message (WA + SMS instant) and Day 3 / Day 7 |
| 2 | 1 | Subscription Expiry — Register templates, CleverTap journey, pre- and post-expiry sequence |
| 3 | 1 | Cancellation Recovery — Register templates, CleverTap journey, trigger on cancel or checkout drop-off |
| 4 | 1 | Onboarding Drop-Off — Register templates, CleverTap journey, trigger on signup without engagement |
| 5 | 1 | Short-Term Win-Back — Register templates, CleverTap journey, trigger on 7-day inactivity |
| 6 | 1 | Weekly to Annual — Register templates, CleverTap journey, 3 messages in first 7 days for weekly subscribers |
| 7 | 1 | Long-Term Win-Back — Register templates, CleverTap journeys (Monthly, App Update, Final) |
| 8 | 1 | Web landing and checkout (PhonePe only) — pages, auth, gateway, and CleverTap triggers |
| 9 | 1 | Subscription API and link variable resolution — Premium sync, {link} resolution, no raw deeplinks |
| 10 | 1 | Opt-out and conversion tracking — STOP/KEEP, DND, and flow-level metrics |
| 2·1 | 2 | Emit feature_paywall_abandoned event — app and backend |
| 2·2 | 2 | Feature Paywall Abandonment — Register templates (feature-specific), CleverTap journey |
| 2·3 | 2 | Paywall UI — News (MCQ/PYQ lock) — article free, tap MCQs/PYQs shows modal |
| 2·4 | 2 | Paywall UI — PYQ (answer verification lock) — Q1 free, Q2+ gated |
| 2·5 | 2 | Paywall UI — AI Explain (blur) — first lines visible, rest blurred |
| 2·6 | 2 | Paywall UI — Mock test results — full mock free, results behind paywall |
| 2·7 | 2 | Shared paywall components — modal, event tracker, blur/lock UI, gating |

---

*For Linear: use the **Title** as the issue title and **Description** plus **Acceptance** as the issue description.*
