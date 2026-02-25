# PadhAI UPSC — PRD for Devs (Rewritten Tickets)

All tickets rewritten with a consistent structure: **Title**, **Description** (numbered sections), **Acceptance**. Use the ticket **Title** as the issue title and **Description** plus **Acceptance** as the issue description in Linear (or your tracker).

---

## Dual-channel rule: WA + SMS at every step

**At every touchpoint we send both WhatsApp and SMS.** So for each timing in a flow, register **one WA template + one SMS template**. Schedules below show each step as “WA + SMS” together.

---

## Event-to-flow reference (dual-channel)

| Flow | Trigger | Schedule (at each step: WA + SMS) |
|------|---------|-----------------------------------|
| Payment Failure | `payment_failed_screen_shown` or gateway failure callback. Emit to backend. | Instant → +1h → +24h. Cancel rest if user pays later. |
| Payment Success | `payment_success_screen_shown` or gateway success. Emit `subscription_activated` with duration. | Instant → Day 3 → Day 7. |
| Subscription Expiry | `subscription_ended` (backend when period ends). | -2d → -1d → instant (expiry) → +24h → +48h. |
| Cancellation | `subscription_cancelled` or checkout drop-off (no success within e.g. 4h). | +4h → +24h → +48h. |
| Trial Activated for 7 days | `trial_activated` (when user downloads app first time and trial is activated). | Instant (one message only; post-trial engagement handled elsewhere). |
| Short-Term Win-Back | No `app_open` / engagement for 7 days (`last_activity`). | Day 7 → Day 10 → Day 14. |
| Weekly to Annual | `subscription_activated` with duration = weekly. | Day 1 → Day 3 → Day 6 (within first 7 days). |
| Long-Term Win-Back | 30d inactive, or app update day, or 180+ days. | Monthly: Day 30 → +24h → +48h. Update: update day → +24h → +48h. Final: single touchpoint (WA + SMS). |
| Feature Paywall (Phase 2) | `feature_paywall_abandoned` with feature (news \| pyq \| ai_explain \| mock). | +3h → +24h → +48h. |

---

## Templates to register (all phases) — WA + SMS at every step

For each timing below, register **one WhatsApp (Meta) template + one SMS (TRAI DLT PDAIUP-S) template**. All links use `{link}`.

| Flow | Touchpoints (each = WA + SMS) | WA templates | SMS templates | Total |
|------|------------------------------|--------------|---------------|-------|
| **Payment Failure** | Instant, +1h, +24h | 3 | 3 | 6 |
| **Payment Success** | Instant, Day 3, Day 7 | 3 | 3 | 6 |
| **Subscription Expiry** | -2d, -1d, instant (expiry), +24h, +48h | 5 | 5 | 10 |
| **Cancellation Recovery** | +4h, +24h, +48h | 3 | 3 | 6 |
| **Trial Activated for 7 days** | Instant (on trial activation) | 1 | 1 | 2 |
| **Short-Term Win-Back** | Day 7, Day 10, Day 14 | 3 | 3 | 6 |
| **Weekly to Annual** | Day 1, Day 3, Day 6 | 3 | 3 | 6 |
| **Long-Term Win-Back** | Monthly: Day 30, +24h, +48h. Update: update day, +24h, +48h. Final: 1 step. | 7 | 7 | 14 |
| **Feature Paywall (Phase 2)** | +3h, +24h, +48h (feature-specific copy) | 3 | 3 | 6 |
| **Total (Phase 1 + Phase 2 notif)** | — | **31** | **31** | **62** |

---

## Phase 1 — Notifications

### Phase 1 · P0-1 — Payment Failure

**Title**  
`[P0] Payment Failure — Register templates, CleverTap journey, event ingestion and delivery`

**Description**

1. **Template registration (priority 1)**  
   Register and get approval for Payment Failure templates. **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** instant, +1h, +24h (3 templates). Use exact copy from Template Library; all links in copy must use variable `{link}`.  
   - **SMS (TRAI DLT, PDAIUP-S Service Implicit):** instant, +1h, +24h (3 templates). Submit DLT first (longer lead time), then Meta.

2. **Backend and events**  
   When payment fails (app event `payment_failed_screen_shown` or web/app payment gateway failure callback), do the following.  
   - Persist the event.  
   - Send user identity and event payload to CleverTap so the journey can trigger.  
   - Ensure gateway failure from PhonePe (web) and PhonePe/Google Play (app) both result in the same event or equivalent payload to backend.  
   - Cancel any scheduled messages for this user if they complete payment later (e.g. on `payment_success` or `subscription_activated`).

3. **CleverTap journey**  
   Build one journey for Payment Failure.  
   - **Trigger:** user enters segment or receives event (payment failed).  
   - **Steps:** send WA + SMS instant; wait 1 hour; send WA + SMS (+1h); wait 23 hours; send WA + SMS (+24h).  
   - **Exit condition:** user completes payment (`subscription_activated` or `payment_success`).  
   Connect WA Business API and SMS gateway to CleverTap. Respect opt-out and DND before each send.

**Acceptance**

- WA and SMS templates for Payment Failure submitted and approved (DLT + Meta).  
- Backend emits/pushes failure event to CleverTap; app and web failure both trigger the same flow.  
- CleverTap journey sends WA + SMS at instant, +1h, and +24h; journey exits on payment success.  
- Links use `{link}`; opt-out/DND honoured.

---

### Phase 1 · 1 — Payment Success

**Title**  
`Payment Success — Register templates, CleverTap journey, dual message (WA + SMS instant) and Day 3 / Day 7`

**Description**

1. **Template registration (priority 2)**  
   Register and get approval for Payment Success templates. **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** instant welcome, Day 3, Day 7 (3 templates). Copy from Template Library; links as `{link}`.  
   - **SMS (TRAI DLT PDAIUP-S):** instant, Day 3, Day 7 (3 templates). Submit DLT then Meta.

2. **Backend and events**  
   On payment success (`payment_success_screen_shown` or gateway success callback from PhonePe/Google Play):  
   - Persist subscription and emit `subscription_activated` with duration (weekly or target_plan).  
   - Send user identity and event to CleverTap so the journey can trigger.  
   - Ensure both app and web success paths result in the same event or equivalent payload.

3. **CleverTap journey**  
   Build one journey for Payment Success.  
   - **Trigger:** `subscription_activated` (or equivalent success event).  
   - **Steps:** send WA + SMS instant; wait until Day 3 from subscription start; send WA + SMS (Day 3); wait until Day 7; send WA + SMS (Day 7).  
   - No exit condition for Day 3/7 once started; optional exit if user unsubscribes.  
   Connect WA and SMS to CleverTap; respect opt-out/DND.

**Acceptance**

- WA and SMS templates for Payment Success submitted and approved.  
- On success, backend persists subscription and pushes event to CleverTap.  
- CleverTap sends WA + SMS at instant, Day 3, and Day 7.  
- `subscription_activated` emitted with duration. All links `{link}`.

---

### Phase 1 · 2 — Subscription Expiry

**Title**  
`Subscription Expiry — Register templates, CleverTap journey, pre- and post-expiry sequence`

**Description**

1. **Template registration (priority 3)**  
   Register Subscription Expiry templates. **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** -2d, -1d, instant at expiry, +24h, +48h (5 templates). Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** -2d, -1d, instant at expiry, +24h, +48h (5 templates). Submit DLT then Meta.

2. **Backend**  
   When a subscription period ends (weekly or annual):  
   - Emit `subscription_ended` with subscription type and end date.  
   - Subscription end date must be available (e.g. from billing or entitlement store) so CleverTap can schedule -2d and -1d relative to that date.  
   - Send event and user to CleverTap.

3. **CleverTap journey**  
   Build one journey for Subscription Expiry.  
   - **Trigger:** `subscription_ended` or scheduled trigger based on known end date.  
   - **Steps:** at T-2 days send WA + SMS; at T-1 day send WA + SMS; at T (expiry) send WA + SMS; wait 24h send WA + SMS (+24h); wait 24h send WA + SMS (+48h).  
   Use approved templates; all links `{link}`. Respect opt-out/DND.

**Acceptance**

- Templates for Subscription Expiry submitted and approved.  
- Backend emits `subscription_ended` with end date.  
- CleverTap sends WA + SMS at -2d, -1d, instant (expiry), +24h, +48h at correct times.

**Subscription Expiry — Templates to register (complete list)**

Register these **10 templates** for Subscription Expiry. **WhatsApp:** Meta utility. **SMS:** TRAI DLT Service Implicit (PDAIUP-S). Variable: WA = `{link}` (CTA URL), SMS = `{#var#}`.

| # | Template name (use for Meta / DLT registration) | Channel | Timing | Body / copy |
|---|--------------------------------------------------|--------|--------|-------------|
| 1 | **subscription_expiry_2d_reminder** | WhatsApp | T-2 days | Your PadhAI Pro subscription ends in 2 days. Click the link below to renew and continue access. **Button:** Renew → `{link}` |
| 2 | **subscription_expiry_2d_reminder** | SMS | T-2 days | Your PadhAI Pro subscription ends in 2 days. Click the link to renew and continue access. {#var#} |
| 3 | **subscription_expiry_1d_reminder** | WhatsApp | T-1 day | Your PadhAI Pro subscription ends tomorrow. Click the link below to renew and continue access. **Button:** Renew → `{link}` |
| 4 | **subscription_expiry_1d_reminder** | SMS | T-1 day | Your PadhAI Pro subscription ends tomorrow. Click the link to renew and continue access. {#var#} |
| 5 | **subscription_expiry_ended** | WhatsApp | T (at expiry) | Your PadhAI Pro subscription has ended. Click the link below to renew and restore access. **Button:** Renew → `{link}` |
| 6 | **subscription_expiry_ended** | SMS | T (at expiry) | Your PadhAI Pro subscription has ended. Click the link to renew and restore access. {#var#} |
| 7 | **subscription_expiry_24h_post** | WhatsApp | T+24 hours | Your PadhAI Pro subscription has ended. Click the link below to renew and restore your access. **Button:** Renew → `{link}` |
| 8 | **subscription_expiry_24h_post** | SMS | T+24 hours | Your PadhAI Pro subscription has ended. Click the link to renew and restore your access. {#var#} |
| 9 | **subscription_expiry_48h_post** | WhatsApp | T+48 hours | Renew your PadhAI Pro subscription to restore access. Click the link below to renew. **Button:** Renew → `{link}` |
| 10 | **subscription_expiry_48h_post** | SMS | T+48 hours | Renew your PadhAI Pro subscription to restore access. Click the link to renew. {#var#} |

**Summary — Subscription Expiry templates to register (with messaging)**

**Meta (WhatsApp) — 5 templates (utility).** CTA button URL = `{link}`.

| Template name | Timing | Messaging (body + button) |
|---------------|--------|---------------------------|
| subscription_expiry_2d_reminder | T-2 days | Your PadhAI Pro subscription ends in 2 days. Click the link below to renew and continue access. Button: Renew → {link} |
| subscription_expiry_1d_reminder | T-1 day | Your PadhAI Pro subscription ends tomorrow. Click the link below to renew and continue access. Button: Renew → {link} |
| subscription_expiry_ended | T (at expiry) | Your PadhAI Pro subscription has ended. Click the link below to renew and restore access. Button: Renew → {link} |
| subscription_expiry_24h_post | T+24 hours | Your PadhAI Pro subscription has ended. Click the link below to renew and restore your access. Button: Renew → {link} |
| subscription_expiry_48h_post | T+48 hours | Renew your PadhAI Pro subscription to restore access. Click the link below to renew. Button: Renew → {link} |

**TRAI DLT (SMS, PDAIUP-S Service Implicit) — 5 templates.** Variable = `{#var#}`.

| Template name | Timing | Messaging (body) |
|---------------|--------|------------------|
| subscription_expiry_2d_reminder | T-2 days | Your PadhAI Pro subscription ends in 2 days. Click the link to renew and continue access. {#var#} |
| subscription_expiry_1d_reminder | T-1 day | Your PadhAI Pro subscription ends tomorrow. Click the link to renew and continue access. {#var#} |
| subscription_expiry_ended | T (at expiry) | Your PadhAI Pro subscription has ended. Click the link to renew and restore access. {#var#} |
| subscription_expiry_24h_post | T+24 hours | Your PadhAI Pro subscription has ended. Click the link to renew and restore your access. {#var#} |
| subscription_expiry_48h_post | T+48 hours | Renew your PadhAI Pro subscription to restore access. Click the link to renew. {#var#} |

*Resolve `{link}` (WA) and `{#var#}` (SMS) at send time to the renewal/checkout URL. **Total: 10 templates.***

---

**Subscription Expiry — Names, messaging & full details (SMS + WhatsApp)**

Register **10 templates** (5 WhatsApp + 5 SMS). **Purpose:** Notify before/after subscription ends.  
**WhatsApp:** Meta, **Utility**. Variable: `{link}` (button URL only).  
**SMS:** TRAI DLT, **Service Implicit** (PDAIUP-S). Variable: `{#var#}` only.

---

**1. subscription_expiry_2d_reminder**  
**When sent:** 2 days before subscription end date  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | subscription_expiry_2d_reminder | **Body:** Your PadhAI Pro subscription ends in 2 days. Click the link below to renew and continue access. **Button:** Renew → `{link}` |
| **SMS** | subscription_expiry_2d_reminder | Your PadhAI Pro subscription ends in 2 days. Click the link to renew and continue access. {#var#} |

---

**2. subscription_expiry_1d_reminder**  
**When sent:** 1 day before subscription end date  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | subscription_expiry_1d_reminder | **Body:** Your PadhAI Pro subscription ends tomorrow. Click the link below to renew and continue access. **Button:** Renew → `{link}` |
| **SMS** | subscription_expiry_1d_reminder | Your PadhAI Pro subscription ends tomorrow. Click the link to renew and continue access. {#var#} |

---

**3. subscription_expiry_ended**  
**When sent:** On subscription end date (day it expires)  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | subscription_expiry_ended | **Body:** Your PadhAI Pro subscription has ended. Click the link below to renew and restore access. **Button:** Renew → `{link}` |
| **SMS** | subscription_expiry_ended | Your PadhAI Pro subscription has ended. Click the link to renew and restore access. {#var#} |

---

**4. subscription_expiry_24h_post**  
**When sent:** 24 hours after subscription ended  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | subscription_expiry_24h_post | **Body:** Your PadhAI Pro subscription has ended. Click the link below to renew and restore your access. **Button:** Renew → `{link}` |
| **SMS** | subscription_expiry_24h_post | Your PadhAI Pro subscription has ended. Click the link to renew and restore your access. {#var#} |

---

**5. subscription_expiry_48h_post**  
**When sent:** 48 hours after subscription ended  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | subscription_expiry_48h_post | **Body:** Renew your PadhAI Pro subscription to restore access. Click the link below to renew. **Button:** Renew → `{link}` |
| **SMS** | subscription_expiry_48h_post | Renew your PadhAI Pro subscription to restore access. Click the link to renew. {#var#} |

---

**All names (for registration):**  
subscription_expiry_2d_reminder · subscription_expiry_1d_reminder · subscription_expiry_ended · subscription_expiry_24h_post · subscription_expiry_48h_post  
(Use same name for both WhatsApp and SMS for each timing.)

**Checklist:** [ ] Meta: 5 WA (Utility). [ ] DLT: 5 SMS (Service Implicit). [ ] At send: replace `{link}` and `{#var#}` with renewal URL.

---

### Phase 1 · 3 — Cancellation Recovery

**Title**  
`Cancellation Recovery — Register templates, CleverTap journey, trigger on cancel or checkout drop-off`

**Description**

1. **Template registration (priority 4)**  
   Register Cancellation Recovery templates. **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** +4h, +24h, +48h (3 templates). Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** +4h, +24h, +48h (3 templates). Submit DLT then Meta.

2. **Backend and events**  
   - Emit `subscription_cancelled` when user cancels from profile or when cancellation is confirmed via PhonePe/Play backend.  
   - For checkout drop-off: if user reached payment (e.g. `subscribe_now_clicked` or `payment_inprogress`) but never `payment_success_screen_shown` within threshold (e.g. 4 hours), treat as cancellation and emit same or equivalent event to CleverTap.  
   - Send user and event to CleverTap.

3. **CleverTap journey**  
   Build one journey for Cancellation.  
   - **Trigger:** `subscription_cancelled` (or checkout drop-off event).  
   - **Steps:** wait 4h; send WA + SMS (+4h); wait 20h; send WA + SMS (+24h); wait 24h; send WA + SMS (+48h).  
   - **Exit:** if user completes payment (`subscription_activated`).  
   Use approved templates; links `{link}`; respect opt-out/DND.

**Acceptance**

- Templates for Cancellation submitted and approved.  
- Explicit cancel and checkout drop-off (after threshold) both trigger journey.  
- CleverTap sends WA + SMS at +4h, +24h, +48h; journey exits on payment success.

**Cancellation Recovery — Templates to register (complete list)**

Register these **6 templates**. **WhatsApp:** Meta utility. **SMS:** TRAI DLT Service Implicit (PDAIUP-S). Variable: WA = `{link}`, SMS = `{#var#}`.

| # | Template name | Channel | Timing | Body / copy |
|---|---------------|--------|--------|-------------|
| 1 | **cancellation_recovery_4h** | WhatsApp | +4h | Your PadhAI Pro subscription was cancelled. Click the link below to resubscribe and restore access. **Button:** Resubscribe → `{link}` |
| 2 | **cancellation_recovery_4h** | SMS | +4h | Your PadhAI Pro subscription was cancelled. Click the link to resubscribe and restore access. {#var#} |
| 3 | **cancellation_recovery_24h** | WhatsApp | +24h | Your PadhAI Pro subscription was cancelled. Click the link below to resubscribe and continue learning. **Button:** Resubscribe → `{link}` |
| 4 | **cancellation_recovery_24h** | SMS | +24h | Your PadhAI Pro subscription was cancelled. Click the link to resubscribe and continue learning. {#var#} |
| 5 | **cancellation_recovery_48h** | WhatsApp | +48h | Resubscribe to PadhAI Pro to restore access. Click the link below to resubscribe. **Button:** Resubscribe → `{link}` |
| 6 | **cancellation_recovery_48h** | SMS | +48h | Resubscribe to PadhAI Pro to restore access. Click the link to resubscribe. {#var#} |

**Cancellation Recovery — Names, messaging & full details (SMS + WhatsApp)**

Register **6 templates** (3 WhatsApp + 3 SMS). **Purpose:** Re-engage after cancel or checkout drop-off.  
**WhatsApp:** Meta, **Utility**. Variable: `{link}` (button URL only).  
**SMS:** TRAI DLT, **Service Implicit** (PDAIUP-S). Variable: `{#var#}` only.

---

**1. cancellation_recovery_4h**  
**When sent:** 4 hours after cancel or checkout drop-off  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | cancellation_recovery_4h | **Body:** Your PadhAI Pro subscription was cancelled. Click the link below to resubscribe and restore access. **Button:** Resubscribe → `{link}` |
| **SMS** | cancellation_recovery_4h | Your PadhAI Pro subscription was cancelled. Click the link to resubscribe and restore access. {#var#} |

---

**2. cancellation_recovery_24h**  
**When sent:** 24 hours after cancel or checkout drop-off  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | cancellation_recovery_24h | **Body:** Your PadhAI Pro subscription was cancelled. Click the link below to resubscribe and continue learning. **Button:** Resubscribe → `{link}` |
| **SMS** | cancellation_recovery_24h | Your PadhAI Pro subscription was cancelled. Click the link to resubscribe and continue learning. {#var#} |

---

**3. cancellation_recovery_48h**  
**When sent:** 48 hours after cancel or checkout drop-off  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | cancellation_recovery_48h | **Body:** Resubscribe to PadhAI Pro to restore access. Click the link below to resubscribe. **Button:** Resubscribe → `{link}` |
| **SMS** | cancellation_recovery_48h | Resubscribe to PadhAI Pro to restore access. Click the link to resubscribe. {#var#} |

---

**All names:** cancellation_recovery_4h · cancellation_recovery_24h · cancellation_recovery_48h  
**Checklist:** [ ] Meta: 3 WA (Utility). [ ] DLT: 3 SMS (Service Implicit). [ ] At send: replace `{link}` and `{#var#}` with resubscribe/checkout URL.

---

### Phase 1 · 4 — Trial Activated for 7 days

**Title**  
`Trial Activated for 7 days — Register templates, CleverTap journey, trigger on trial activation (first app download)`

**Description**

1. **Template registration (priority 5)**  
   Register **one** Trial Activated template per channel (1 WA + 1 SMS = 2 templates total). Post-trial engagement is handled by other flows; this flow sends a single welcome message when the 7-day trial is activated.  
   - **WhatsApp (Meta utility):** 1 template. Copy below; link = `{link}`.  
   - **SMS (TRAI DLT Service Implicit):** 1 template. Copy below; variable = `{#var#}`.

2. **Backend and trigger**  
   - When a user downloads the app for the first time and the 7-day trial is activated, emit `trial_activated`.  
   - Send user identity and event to CleverTap so this journey triggers once per user.

3. **CleverTap journey**  
   Build one journey for Trial Activated.  
   - **Trigger:** `trial_activated` (first app download, trial activated).  
   - **Steps:** send WA + SMS instant (one message only).  
   Use approved templates; respect opt-out/DND.

**Acceptance**

- Trial Activated WA and SMS templates submitted and approved (Meta Utility, DLT Service Implicit).  
- On `trial_activated`, user receives one WA + one SMS with the "we will teach you how to use the app for UPSC" messaging.  
- No further templates in this flow; post-trial engagement uses other flows.

**Trial Activated for 7 days — Templates to register (complete list)**

Register **2 templates** (1 WhatsApp + 1 SMS). **WhatsApp:** Meta utility. **SMS:** TRAI DLT Service Implicit (PDAIUP-S). Variable: WA = `{link}`, SMS = `{#var#}`.

| # | Template name | Channel | Timing | Body / copy |
|---|---------------|--------|--------|-------------|
| 1 | **trial_activated_7days** | WhatsApp | Instant | Your 7-day trial of PadhAI Pro is active. We will teach you how to use the app for UPSC in the next few days. Click the link below to open the app and get started. **Button:** Open app → `{link}` |
| 2 | **trial_activated_7days** | SMS | Instant | Your 7-day trial of PadhAI Pro is active. We will teach you how to use the app for UPSC in the next few days. Click the link to open the app and get started. {#var#} |

**Trial Activated — Names, messaging & full details (SMS + WhatsApp)**

Register **2 templates** (1 WhatsApp + 1 SMS). **Purpose:** Welcome users when their 7-day trial is activated on first app download; set expectation that we will teach them how to use the app for UPSC. Post-trial engagement is handled by other flows.  
**WhatsApp:** Meta, **Utility**. Variable: `{link}` (button URL only).  
**SMS:** TRAI DLT, **Service Implicit** (PDAIUP-S). Variable: `{#var#}` only.

---

**trial_activated_7days**  
**When sent:** Instant on `trial_activated` (first app download, 7-day trial activated)  

| Channel | Name | Messaging (full content to submit) |
|---------|------|------------------------------------|
| **WhatsApp** | trial_activated_7days | **Body:** Your 7-day trial of PadhAI Pro is active. We will teach you how to use the app for UPSC in the next few days. Click the link below to open the app and get started. **Button:** Open app → `{link}` |
| **SMS** | trial_activated_7days | Your 7-day trial of PadhAI Pro is active. We will teach you how to use the app for UPSC in the next few days. Click the link to open the app and get started. {#var#} |

**All names:** trial_activated_7days  
**Checklist:** [ ] Meta: 1 WA (Utility). [ ] DLT: 1 SMS (Service Implicit). [ ] At send: replace `{link}` and `{#var#}` with app open URL.

---

### Phase 1 · 5 — Short-Term Win-Back

**Title**  
`Short-Term Win-Back — Register templates, CleverTap journey, trigger on 7-day inactivity`

**Description**

1. **Template registration (priority 6)**  
   Register Short-Term Win-Back templates. **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** Day 7, Day 10, Day 14 (3 templates). Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** Day 7, Day 10, Day 14 (3 templates). Submit DLT then Meta.

2. **Backend and data**  
   - Ensure `last_activity` or last `app_open` timestamp is available per user (e.g. from app events or CleverTap).  
   - CleverTap must be able to segment users whose `last_activity` is exactly 7 days ago (or 10 and 14 for subsequent steps).  
   - Dedupe so a user does not re-enter this flow multiple times for the same inactivity spell (e.g. once per 7-day-inactive entry).

3. **CleverTap journey**  
   Build one journey for Short-Term Win-Back.  
   - **Trigger:** user enters segment “last_activity 7 days ago” (and not already in this journey).  
   - **Steps:** send WA + SMS (Day 7); wait 3 days send WA + SMS (Day 10); wait 4 days send WA + SMS (Day 14).  
   Use approved templates; links `{link}`. Respect opt-out/DND. Ensure no duplicate sequences per user per qualifying period.

**Acceptance**

- Templates for Short-Term Win-Back submitted and approved.  
- Users inactive 7 days get WA + SMS at Day 7, Day 10, Day 14.  
- No duplicate sequences per user per spell.

---

### Phase 1 · 6 — Weekly to Annual Upsell

**Title**  
`Weekly to Annual — Register templates, CleverTap journey, 3 messages in first 7 days for weekly subscribers`

**Description**

1. **Template registration (priority 7)**  
   Register Weekly to Annual templates. **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** Day 1, Day 3, Day 6 (3 templates). Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** Day 1, Day 3, Day 6 (3 templates). Submit DLT then Meta.

2. **Backend and events**  
   - When `subscription_activated` is emitted, include duration (weekly or target_plan).  
   - CleverTap must receive this so it can start the upsell journey only for users with duration = weekly.  
   - When user upgrades to annual (`subscription_activated` with duration = target_plan or equivalent), journey must exit and cancel remaining messages.

3. **CleverTap journey**  
   Build one journey for Weekly to Annual.  
   - **Trigger:** `subscription_activated` with duration = weekly.  
   - **Steps:** send WA + SMS (Day 1); wait 2 days send WA + SMS (Day 3); wait 3 days send WA + SMS (Day 6). All within first 7 days of subscription.  
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
   Register Long-Term Win-Back templates (three sub-flows). **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** Monthly: Day 30, +24h, +48h (3). App Update: update day, +24h, +48h (3). Final: single touchpoint (1). Total 7 templates. Copy from Template Library; links `{link}`.  
   - **SMS (TRAI DLT):** Same 7 timings (Monthly 3, App Update 3, Final 1). Submit DLT then Meta.

2. **Backend and data**  
   - `last_activity` (or last `app_open`) must be available.  
   - For App Update sub-flow: app version or release event must be available so CleverTap can target “inactive users” at update release.  
   - For Final (180+ days): segment users inactive 180+ days; single WA only, no SMS.

3. **CleverTap journeys**  
   Build three sub-flows or one journey with branches.  
   - **(a) Monthly:** trigger when last_activity = 30 days ago; send WA + SMS Day 30, wait 24h send WA + SMS, wait 24h send WA + SMS.  
   - **(b) App update:** trigger on app release; target inactive users; send WA + SMS on update day, then +24h, then +48h.  
   - **(c) Final:** trigger when last_activity 180+ days; send single step WA + SMS (KEEP/STOP).  
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
   Register Feature Paywall Abandonment templates (same pattern as Phase 1: register templates and build journey in one ticket). **At every step we send both WA + SMS.**  
   - **WhatsApp (Meta utility):** +3h, +24h, +48h (3 templates).  
   - **SMS (TRAI DLT PDAIUP-S):** +3h, +24h, +48h (3 templates).  
   - Message copy must be feature-specific: four variants (News, PYQ, AI Explain, Mock) so the right copy is sent based on `feature_paywall_abandoned.feature`. Use Template Library for copy; all links as `{link}`. Submit DLT then Meta. If Meta/DLT require one template per variant, register all required variants.

2. **CleverTap journey**  
   Build one journey for Feature Paywall Abandonment.  
   - **Trigger:** event `feature_paywall_abandoned`.  
   - **Steps:** wait 3h; send WA + SMS (feature-specific); wait 21h; send WA + SMS (+24h, feature-specific); wait 24h; send WA + SMS (+48h, feature-specific).  
   - Resolve `{link}` per user/context. Respect opt-out/DND. Exit if user subscribes (`subscription_activated`).

**Acceptance**

- Feature-specific WA and SMS templates for Feature Paywall submitted and approved.  
- CleverTap journey sends WA + SMS at +3h, +24h, +48h with copy that matches the feature user abandoned; links use `{link}`; journey exits on subscribe.

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
| 4 | 1 | Trial Activated for 7 days — Register templates, CleverTap journey, trigger on trial activation (first app download) |
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

## Event catalog — How to catch all events and trigger -2d / +N journeys

Use this to ensure every journey can be triggered at the right time (-2d, +6h, +24h, etc.). **Source** = where to emit (App, Backend, or both). **Properties** = what to send so CleverTap (or your scheduler) can run the journey at the correct offset.

### 1. Events you already have / are straightforward

| Event | Source | Key properties | Journey(s) | Notes |
|-------|--------|----------------|-----------|--------|
| `payment_failed_screen_shown` | App | — | Payment Failure | Emit when user sees failure screen; backend also on gateway failure callback. |
| `payment_success_screen_shown` | App (and web) | — | — | Use to trigger backend → `subscription_activated`. |
| `subscription_activated` | Backend | `duration` (weekly \| target_plan), `platform` (app \| web), **`subscription_start_date`**, **`subscription_end_date`** | Payment Success, Weekly to Annual | **Add** `subscription_start_date` and `subscription_end_date` (ISO) so Day 3/7 and expiry -2d/-1d can be scheduled. |
| `subscription_cancelled` | Backend (or app if user cancels in-app) | `cancelled_at` | Cancellation Recovery | Emit when user cancels from profile or when PhonePe/Play confirms cancellation. |
| `feature_paywall_abandoned` | App | `feature` (news \| pyq \| ai_explain \| mock) | Feature Paywall Abandonment | Emit when user dismisses paywall without subscribing. |

### 2. Events / data you must add to support all schedules

#### Subscription Expiry (-2d, -1d, instant, +24h, +48h)

- **Option A (recommended):** Backend emits **`subscription_ended`** when the subscription period ends, with:
  - `subscription_end_date` (or `ended_at`) — so CleverTap can also schedule **in advance** for -2d and -1d.
- **Option B:** Backend (or a cron) **pre-schedules** expiry messages: for every active subscription, compute `end_date` and at `end_date - 2 days` and `end_date - 1 day` emit an event or push the user into a “subscription_expiring_2d” / “subscription_expiring_1d” segment so the journey sends the right template.
- **Required:** A **user property** or event payload **`subscription_end_date`** (or equivalent) so the journey engine knows when to send -2d and -1d. If you only emit `subscription_ended` on the day of expiry, you cannot send -2d and -1d unless you pre-compute and schedule (e.g. daily job that finds subscriptions ending in 2 days / 1 day and triggers the journey).

**Recommended:** Emit **`subscription_expiring_soon`** from backend (scheduled job) with:
- `days_until_end` (2 or 1)
- `subscription_end_date`
- User identity  
So one journey can branch: if `days_until_end == 2` → send -2d WA; if `days_until_end == 1` → send -1d SMS. On the actual end date, emit `subscription_ended` to trigger instant + +24h + +48h.

#### Cancellation / checkout drop-off (+4h, +24h, +48h)

- **Explicit cancel:** Emit `subscription_cancelled` (you have this).
- **Checkout drop-off:** You already have **`subscribe_now_clicked`** and **`payment_inprogress`**. Backend: if `payment_inprogress` (or `subscribe_now_clicked`) and **no** `subscription_activated` within **4 hours**, **emit `checkout_drop_off`**. That event triggers the same Cancellation journey. No new app events needed.

#### Trial Activated for 7 days (instant)

- When a user downloads the app for the first time and the 7-day trial is activated, **emit `trial_activated`** (from app or backend). CleverTap journey triggers once and sends one WA + one SMS. Post-trial engagement is handled by other flows (e.g. Short-Term Win-Back, Subscription Expiry).
- **Event to add:** **`trial_activated`** — fired when trial is activated on first app download.

#### Short-Term Win-Back (Day 7, Day 10, Day 14)

- You already have **`app_open`**. CleverTap (or backend) updates **user property `last_activity`** (date) from `app_open`.
- **Journey trigger:** CleverTap segment “last_activity = 7 days ago” (and 10, 14 for later steps). Journey triggers on segment entry. No separate backend events (e.g. inactive_7_days) required.
- **Only to add:** Ensure **user property `last_activity`** is set from `app_open`.

#### Long-Term Win-Back (30d, app update, 180d)

- **30d and 180d:** Use **`last_activity`** (from existing `app_open`). Trigger: CleverTap segments “last_activity 30 days ago” or “last_activity ≥ 180 days ago”. Journey on segment entry. No inactive_30_days / inactive_180_days / app_version_released events required.
- **App update:** Target “inactive users” via segment (e.g. last_activity > 7 days) and run the update-day journey when you release; no separate `app_version_released` event needed.

#### Weekly to Annual (Day 1, Day 3, Day 6)

- Trigger: **`subscription_activated`** with **`duration = weekly`**. Journey starts then; send WA Day 1, SMS Day 3, WA Day 6. Ensure `subscription_activated` includes **`subscription_start_date`** so “Day 1 / 3 / 6” are computed correctly.

### 3. Events to add — not on the sheet yet

Use this as the implementation checklist. **Already have (not listed here):** `subscribe_now_clicked`, `payment_inprogress`, `account_created`, `app_open`. Payment Failure / Success events not listed.

**Flow → trigger events (for reference):**

| Flow | Trigger event(s) |
|------|-------------------|
| Payment Failure | (done) `payment_failed_screen_shown`, gateway failure callback |
| Payment Success | (done) `subscription_activated` |
| Subscription Expiry | **`subscription_expiring_soon`** (-2d, -1d), **`subscription_ended`** (T, +24h, +48h) |
| Cancellation Recovery | `subscription_cancelled`, **`checkout_drop_off`** (use existing `subscribe_now_clicked` / `payment_inprogress` + no success in 4h) |
| Trial Activated for 7 days | **`trial_activated`** (when trial is activated on first app download) |
| Short-Term Win-Back | **`last_activity`** (user property from existing `app_open`) — segment / journey on “7 days inactive” |
| Weekly to Annual | `subscription_activated` with duration=weekly (add **`subscription_start_date`** if missing) |
| Long-Term Win-Back | **`last_activity`** (from existing `app_open`) — segment / journey on “30d / 180d inactive”, “app update” |
| Feature Paywall (Phase 2) | **`feature_paywall_abandoned`** |

| # | Event | Source | Properties | Used for (flow) |
|---|-------|--------|------------|------------------|
| 1 | **`subscription_ended`** | Backend | `subscription_end_date` (ISO), `subscription_type` (optional) | Subscription Expiry (at T, +24h, +48h) |
| 2 | **`subscription_expiring_soon`** | Backend (scheduled job) | `days_until_end` (2 or 1), `subscription_end_date` | Subscription Expiry (-2d, -1d) |
| 3 | **`checkout_drop_off`** | Backend | — | Cancellation Recovery (when `payment_inprogress` + no success in 4h) |
| 4 | **`trial_activated`** | App / Backend | — | Trial Activated for 7 days (instant on first download, trial activated) |
| 5 | **User property: `last_activity`** | From existing `app_open` (CleverTap or backend) | date (last active) | Short-Term & Long-Term Win-Back (segments) |
| 6 | **`feature_paywall_abandoned`** | App | `feature` (news \| pyq \| ai_explain \| mock) | Feature Paywall Abandonment (Phase 2) |

**Existing events — add these properties if missing:**

| Event | Add these properties |
|-------|----------------------|
| **`subscription_activated`** | `subscription_start_date` (ISO), `subscription_end_date` (ISO) — needed for Payment Success Day 3/7, Weekly to Annual, and Subscription Expiry -2d/-1d scheduling. |

---

### 4. Summary — Full event list (add only the ones not on sheet)

**Already have:** `subscribe_now_clicked`, `payment_inprogress`, `account_created`, `app_open`, payment failure/success events.

| Event | Source | Properties | Used for |
|-------|--------|------------|----------|
| `subscription_activated` | Backend | `duration`, `platform`, **`subscription_start_date`**, **`subscription_end_date`** | Payment Success, Weekly to Annual, expiry pre-schedule |
| **`subscription_ended`** | Backend | `subscription_end_date`, `type` | Subscription Expiry (instant, +24h, +48h) |
| **`subscription_expiring_soon`** | Backend (scheduled) | `days_until_end` (2 or 1), `subscription_end_date` | Subscription Expiry (-2d, -1d) |
| `subscription_cancelled` | Backend / App | `cancelled_at` | Cancellation Recovery |
| **`checkout_drop_off`** | Backend | — | Cancellation Recovery (payment_inprogress + no success in 4h) |
| **`trial_activated`** | App / Backend | — | Trial Activated for 7 days (instant) |
| **User property: `last_activity`** | From `app_open` (CleverTap or backend) | date | Short-Term and Long-Term Win-Back |
| **`feature_paywall_abandoned`** | App | `feature` | Feature Paywall Abandonment (Phase 2) |

### 5. How -2d and +N work in practice

- **-2d / -1d (Subscription Expiry):** Backend cron (e.g. daily) finds subscriptions with `end_date` = today + 2 days (or + 1 day), and emits `subscription_expiring_soon` with `days_until_end: 2` or `1`. Journey receives the event and sends the right template. No need for CleverTap to “look back” — you push the trigger at the right calendar time.
- **Trial Activated (instant):** When user downloads the app for the first time and 7-day trial is activated, emit `trial_activated`. Journey sends one WA + one SMS. No cron needed.
- **+4h (Checkout drop-off):** Backend checks: “payment_inprogress” at T and no “subscription_activated” by T+4h → emit `checkout_drop_off`. Journey runs.
- **Day 7 / 30 / 180 (Win-back):** Use user property `last_activity` (updated from existing `app_open`). CleverTap segments: “last_activity = 7 (or 30, 180) days ago”. Journey triggers on segment entry. No separate inactive_* or app_version_released events required.

---

*For Linear: use the **Title** as the issue title and **Description** plus **Acceptance** as the issue description.*
