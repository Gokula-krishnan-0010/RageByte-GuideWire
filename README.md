# SignalArmor — AI-Powered Parametric Income Insurance for India's Delivery Workers

**Guidewire DEVTrails 2026 | Team RageByte**

> Protecting the earnings of India's 12 million gig delivery workers against
> uncontrollable disruptions — weather, pollution, and civic shutdowns —
> through automated parametric payouts and physics-backed fraud prevention.

---

## The Problem

India's platform delivery sector is the backbone of urban commerce.
In FY 2024–25, the gig workforce surged to **12 million workers**,
growing at a CAGR of 17% from 7.7 million in 2020–21. Delivery partners
on Zomato and Swiggy alone number over 5 million active riders.

Yet these workers carry enormous income risk with zero protection:

- **₹21,000–₹26,500/month** is the net earning potential for a full-time
  Zomato delivery partner in 2025 (₹102/hr EPH × 10hrs × 26 days,
  minus ~20% fuel/maintenance). Source: Zomato CEO Deepinder Goyal, Jan 2026.
- **78% earn under ₹2.5 lakh/year** (under ₹21,000/month), leaving zero
  financial buffer. Source: Borzo India Survey, 2024.
- **20–30% monthly income loss** during severe weather events, civic
  disruptions, or pollution alerts — with no compensation mechanism.
- **0% of these workers** have access to parametric income insurance.
  They are classified as independent contractors, excluded from ESIC,
  EPF, or any statutory income protection.
- India's gig economy is projected to reach **23.5 million workers by 2030**
  and **61.9 million by 2047** (NITI Aayog), making this the largest
  uninsured workforce segment in the country.

A single severe rain event in Mumbai can wipe out 2–3 days of earnings
for 50,000+ delivery workers simultaneously. There is currently no
product that addresses this.

---

## Our Solution — SignalArmor

SignalArmor is a **native Android parametric insurance platform** for
food delivery workers (Zomato / Swiggy persona). Workers pay a small
**weekly premium** and receive automatic income replacement when a
verified external disruption makes delivery work impossible in their zone.

**No claim filing. No paperwork. Payout arrives before the storm ends.**

### Coverage Scope (Strictly Defined)

SignalArmor covers **income lost due to verified external disruptions only**:

| Covered | Explicitly Excluded |
|---|---|
| Severe rainfall (>10mm/hr) halting deliveries | Vehicle damage or repair costs |
| Red-alert weather zones (cyclone, flood) | Health, injury, or accident bills |
| Severe pollution (AQI >300) halting outdoor work | Equipment loss or theft |
| Civic disruptions (verified curfew, zone closure) | Personal or family emergencies |

The payout is calibrated to **lost hours × average hourly earnings** for
the worker's historical earnings tier — not a flat amount.

---

## 1. Persona, Scenarios, and Workflow

### Chosen Persona: Food Delivery Partner (Zomato / Swiggy)

We chose food delivery because:
- Peak earning hours are evenings and weekends — exactly when storm
  disruptions are most severe.
- Workers operate in defined urban delivery zones, making geofenced
  parametric triggers precise and auditable.
- Zomato and Swiggy together have 5M+ active partners, representing
  the largest addressable segment.
- Average active hours: 7 hrs/working day, 38 working days/year
  (part-time profile) to 260 days/year (full-time profile).

### Scenario A — Priya, the Genuine Stranded Worker

Priya is a Swiggy delivery partner in Andheri, Mumbai. At 7:15pm on
a Tuesday, a sudden pre-monsoon squall hits. Rainfall crosses 18mm/hr.
SignalArmor's weather polling detects the breach. An FCM push reaches
Priya's phone. She taps "I am affected." Her phone's sensor bundle
(GPS, accelerometer, barometric pressure, cell tower IDs) is assembled,
cryptographically signed, and uploaded. The server verifies:
- Her barometer reads 989 hPa — consistent with the storm
- Her accelerometer shows erratic movement consistent with outdoor wind
- Her cell towers confirm she is in the Andheri zone, not at home
- Open-Meteo confirms 18.3mm/hr rainfall at her coordinates

Fraud score: 4/100. Auto-pay: ₹180 (2 hours × ₹90 avg) reaches
her UPI account in 47 seconds.

### Scenario B — The "Ghost Claimer" Fraud Ring

A Telegram group of 500 delivery workers in Chennai coordinates to
spoof their GPS to a red-alert flood zone while sitting safely at home.
They file simultaneous claims. SignalArmor's defense:
- **Device attestation**: 340 of 500 devices have mock location enabled
  in developer settings → Play Integrity returns `MEETS_BASIC_INTEGRITY`
  only → +40 fraud points each, all auto-rejected
- **Cell tower resolution**: The remaining 160 devices pass attestation
  but their cell towers (resolved via OpenCellID) place them in
  residential neighborhoods 8km from the claimed zone → +30 pts each
- **Claim surge detector**: 500 claims in 2 minutes from a 3km zone
  triggers the population-level ring flag → +20 pts added to all
- **BSSID fingerprint**: 214 devices are connected to known home Wi-Fi
  routers from their onboarding baseline scan → +25 pts each

Net result: All 500 claims held. Zero fraudulent payout. Entire ring
flagged for investigation.

### Application Workflow

```
Worker registers (once)
    ↓
Chooses weekly coverage tier (₹29/₹49/₹79 per week)
    ↓
App runs foreground sensor service during active shifts
    ↓
Backend polls Open-Meteo every 10 minutes per worker zone
    ↓
Storm threshold breached → FCM trigger to affected workers
    ↓
Worker taps confirm → bundle assembled + KeyStore signed
    ↓
Server: signature verify → cell tower check → weather
        corroboration → fraud score (0–100)
    ↓
Score 0–39: Auto-pay via Razorpay UPI (<60 seconds)
Score 40–69: Partial pay (60%) + 2hr async re-check
Score 70+:   Hold + human review queue (2hr SLA)
    ↓
Immutable audit log written to Supabase
```

---

## 2. Weekly Premium Model

### Why Weekly?

Delivery workers earn and spend in weekly cycles. A monthly premium
creates a cash flow mismatch — workers in Tier-2 cities earning
₹18,000/month simply will not prepay ₹200+ upfront. Weekly pricing
aligns with when workers *receive* their platform payouts.

### Premium Tiers

| Tier | Weekly Premium | Max Weekly Coverage | Best For |
|---|---|---|---|
| Essential | ₹29/week | ₹360 (4 hrs at ₹90) | Part-time (<20 hrs/wk) |
| Standard | ₹49/week | ₹720 (8 hrs at ₹90) | Regular (20–40 hrs/wk) |
| Pro | ₹79/week | ₹1,260 (14 hrs at ₹90) | Full-time (40+ hrs/wk) |

Coverage activates every Monday 12:00am and expires Sunday 11:59pm.
Unused coverage does not roll over. Workers can upgrade/downgrade weekly.

### Dynamic Pricing with AI

The weekly premium is *not static*. A **Random Forest Regressor** adjusts
it based on the worker's hyper-local risk profile:

- **Zone risk factor**: Historical disruption frequency in the worker's
  registered delivery zone (sourced from 24 months of Open-Meteo
  archive data per pin code)
- **Device trust score**: Workers with consistently high Play Integrity
  scores pay 8–12% less (lower fraud risk = lower operational cost)
- **Claim history**: Workers with 6+ months of zero fraudulent signals
  receive a 15% loyalty discount on the Standard/Pro tiers
- **Seasonal adjustment**: Monsoon weeks (June–September) carry a
  +10–20% uplift for zones with historical waterlogging patterns

Example: A full-time Swiggy partner in Dharavi (high flood-risk zone,
monsoon season, new account) pays ₹89/week on the Pro tier.
The same worker in Bandra (lower flood risk, 12 months clean history)
pays ₹67/week.

### Parametric Triggers

A trigger fires when **any** of the following conditions are met
for the worker's registered delivery zone:

| Trigger | Threshold | Source API | Income Loss Basis |
|---|---|---|---|
| Rainfall intensity | >10mm/hr sustained 30 min | Open-Meteo | Deliveries halted |
| Severe weather alert | Red/Orange IMD alert in zone | Open-Meteo | Platform suspends ops |
| Air quality index | AQI >300 (Hazardous) | OpenAQ | Outdoor work prohibited |
| Civic disruption | Verified curfew/zone closure | Manual + Traffic API | Access blocked |
| Extreme heat | >44°C for 2+ hours | Open-Meteo | Health-risk advisory |

---

## 3. AI/ML Integration

### A. Dynamic Premium Engine — Random Forest Regressor

**Input features**: zone_flood_score, zone_heat_score,
worker_tenure_days, claim_frequency_90d, device_trust_score,
season_index, city_tier, delivery_platform.

**Output**: weekly_premium_inr (continuous value, clipped to tier bands)

**Training data**: Simulated from Open-Meteo 24-month historical
archive × Indian city disruption event logs. Retrained monthly
as real claim data accumulates.

### B. GPS Anti-Spoofing — The SignalArmor Verification Cage

This is SignalArmor's core innovation and namesake. Rather than trusting
software-reported GPS, we build a "Verification Cage" from three
independent physical signals that no user-space application can forge:

**Signal 1 — Cellular Timing Advance (TA)**

![Cell_tower_triangulation](images/Cell_tower_triangulation.jpeg)

Timing Advance is the number of symbols (in units of ~550 meters) a
mobile device is instructed to advance its transmission to compensate
for propagation delay to the cell tower. It is set by the base station's
hardware, not the device — it physically cannot be spoofed by any
user-space application, including GPS mock location apps.

A device claiming to be at Location A but whose Timing Advance value
places it ≥2km from that location's nearest cell tower is presenting
a physically impossible scenario. We read TA via
`CellInfoLte.getTimingAdvance()` (Android API) and cross-reference
it against the tower's known position from OpenCellID.

This is the strongest single anti-spoofing signal available without
a carrier partnership — and it works even when mock location is
explicitly enabled in developer settings.

**Signal 2 — Barometric Pressure Correlation**

A storm produces a measurable pressure drop (typically 10–25 hPa below
normal). We compare the device barometer reading against Open-Meteo's
real-time surface pressure for the claimed coordinates. A delta >8 hPa
indicates the device is not physically present in the claimed weather zone.

**Signal 3 — Accelerometer / IMU Storm Signature**

A genuine outdoor delivery worker in a storm shows: high-variance
tri-axis acceleration (wind gusts, road vibration, unsteady handling),
device orientation changes, and periodic spikes consistent with
two-wheeler operation. An indoor fraudster shows near-flat variance
(<0.3 m/s² across all axes). A **1D CNN** trained on labeled motion
segments classifies the 60-minute sensor buffer as one of:
`outdoor_moving`, `outdoor_sheltering`, `indoor_resting`, `synthetic`.

### C. Fraud Detection — Multi-Layer Scoring Engine

Every claim is scored on a 0–100 scale. Points are additive:

| Signal | Condition | Points |
|---|---|---|
| Device attestation | Mock location ON or device rooted | +40 |
| Cell tower (Timing Advance) | TA places device >2km from GPS claim | +30 |
| GPS trajectory | Implied speed >120 km/h in 60min trail | +30 |
| Home BSSID | Known home router detected at claim time | +25 |
| Weather API | Open-Meteo shows no storm at claimed zone | +25 |
| Barometer | Device hPa vs. API hPa delta >8 hPa | +20 |
| Accelerometer | Flatline variance over 10 minutes | +15 |
| Ring detector | >15 simultaneous claims in 2km / 3 min | +20 |
| New account | Account age <14 days at first claim | +10 |

Score 0–39: Auto-pay (Tier 1)
Score 40–69: Partial pay + async re-check (Tier 2)
Score 70–100: Hold + human review queue (Tier 3)

### D. Ring Detection — Graph Neural Network

A GNN is constructed where workers are nodes and edges are shared
static attributes: common Wi-Fi BSSID history (from onboarding scans),
device fingerprint similarity (model + OS + install timestamp cluster),
and platform referral chain (who onboarded whom).

When a claim burst fires, the GNN aggregates fraud probability across
connected subgraphs. A cluster of 20 workers sharing 3 common historical
BSSIDs is flagged as a coordinated ring even if each individual passes
their standalone checks. The lead node (highest connectivity, newest
account) is identified and suspended pending investigation.

---

## 4. Low Network / Offline Resilience

A genuine storm is exactly when network connectivity degrades most.
SignalArmor handles this via **Offline-First Signed Bundles**:

1. Sensor data is continuously buffered on-device in an encrypted
   Room database (SQLite)
2. At claim time, the bundle is signed in the Android KeyStore secure
   enclave — the private key physically cannot leave the hardware chip
3. The signed bundle is queued in WorkManager with exponential backoff
4. When connectivity returns (even minutes later), the bundle uploads
   and is processed — the cryptographic signature proves it was created
   before upload, preventing retroactive data fabrication
5. Workers receive an on-device provisional confirmation immediately,
   with final settlement confirmed upon upload

---

## 5. Tech Stack

| Layer | Technology | Cost |
|---|---|---|
| Mobile | Kotlin (Android) — native Telephony/Sensor access | Free |
| Backend | Node.js + Express on Railway.app | Free (500 hrs/mo) |
| Database | Supabase PostgreSQL + PostGIS | Free (500MB) |
| Auth | Supabase Auth — OTP SMS login | Free tier |
| Weather triggers | Open-Meteo API | Free, unlimited |
| Pollution triggers | OpenAQ API | Free, unlimited |
| Cell tower lookup | OpenCellID API | Free (100k/day) |
| Device attestation | Google Play Integrity API | Free |
| Push notifications | Firebase Cloud Messaging | Free |
| Payment | Razorpay Payout API (test/sandbox mode) | Free in test |
| AI/ML runtime | PyTorch (GNN) + TFLite (on-device CNN) | Free |
| Fraud scoring | Custom Node.js weighted engine | Own code |
| Visualisation | Mapbox GL — signal sector vs. GPS pin map | Free tier |

**Total monthly infrastructure cost at MVP scale: ₹0**

---

## 6. Adversarial Defense & Anti-Spoofing Strategy

*[Added per DEVTrails Phase 1 urgent requirement, March 20, 2026]*

### Differentiation: Genuine Worker vs. GPS Spoofer

GPS spoofing apps (Fake GPS, Mock Locations) operate at the Android
OS location API layer — they inject false coordinates into `LocationManager`.
They cannot touch hardware-level signals. SignalArmor's differentiation:

A genuine worker in a storm produces a correlated, consistent signature
across six independent physical layers simultaneously. A fraudster at home
with a spoofed GPS produces a signature where the software layer (GPS) is
inconsistent with every hardware layer (barometer, accelerometer, cell
towers, Wi-Fi). No single app can forge all six simultaneously.

The decisive check is **Timing Advance** — set by the cell tower's
hardware, physically unreachable by any user-space spoofing application.
This is the signal that gives SignalArmor its name: the armor is built
from physics, not software rules.

### Data Points Beyond GPS

1. Timing Advance (TA) — cell tower hardware-imposed transmission delay
2. Neighboring cell list — fingerprint of visible towers by signal strength
3. BSSID scan — nearby Wi-Fi router IDs (home network detection)
4. Barometric pressure — correlated against Open-Meteo archive
5. Accelerometer variance — 1D CNN storm signature classification
6. GPS accuracy / signal-to-noise ratio — storm degrades GPS precision
7. Claim surge velocity — population-level ring detection
8. Device attestation verdict — Play Integrity hardware root of trust

### UX Balance for Flagged Claims

We never say "fraud" or "suspicious" to any worker. The UX language:

- Tier 1: "₹180 transferred to your account · UTR: XXXXXXXX"
- Tier 2: "₹108 transferred now · We're doing a quick check
   for the remaining ₹72 · You'll hear from us within 2 hours"
- Tier 3: "Your claim is under a quick verification review.
   We'll update you by [time + 2hrs]. Need help? Call 1800-XXX-XXXX"

Workers are never penalized without recourse. A genuine worker who
triggers soft flags (e.g., dropped network in a storm) will have
their claim upgraded once the 2-hour async re-check completes with
additional sensor data gathered post-incident.

---

## 7. Development Plan — 6-Week Roadmap

### Phase 1 (Mar 4–20) — Ideation & Foundation ✓
- [x] Persona definition (Food delivery, Zomato/Swiggy)
- [x] Weekly premium model designed
- [x] Parametric trigger thresholds defined
- [x] Anti-spoofing strategy (Timing Advance + sensor fusion)
- [x] Tech stack finalized
- [x] README + 2-min video submitted

### Phase 2 (Mar 21–Apr 4) — Automation & Protection
- [ ] Android app: OTP onboarding + zone registration
- [ ] Sensor foreground service (GPS + accel + baro + cell towers)
- [ ] Backend: Open-Meteo poller + zone breach detector
- [ ] FCM trigger dispatch to affected workers
- [ ] Play Integrity attestation integration
- [ ] Bundle signing via Android KeyStore
- [ ] Dynamic premium calculator (Random Forest)
- [ ] Razorpay sandbox payout integration
- [ ] Tier 1 auto-pay flow end-to-end demo

### Phase 3 (Apr 5–17) — Scale & Optimise
- [ ] OpenCellID cell tower resolution + Timing Advance check
- [ ] 1D CNN on-device accelerometer storm classifier (TFLite)
- [ ] GNN ring detector (PyTorch — batch job every 5 min)
- [ ] Full 9-signal fraud scoring engine
- [ ] Tier 2/3 partial pay + human review queue
- [ ] Worker dashboard: earnings protected, coverage status, claim history
- [ ] Insurer admin dashboard: loss ratio, zone heatmap, fraud queue
- [ ] 5-min demo video: simulated rainstorm → auto-claim → UPI payout
- [ ] Final pitch deck

---

## 8. Business Viability

**Total Addressable Market**: 5 million food delivery workers in India
with average weekly earnings of ₹4,500–₹6,500 (₹18,000–₹26,000/month).

**Revenue model**: ₹49/week Standard tier × 1 million active subscribers
= ₹49 million/week = ₹2.55 billion/year gross premium.

**Loss ratio target**: 35–45% (₹17–22M/week in payouts). Historical
disruption frequency in Tier-1 cities: approximately 8–12 red-alert days
per year per zone (monsoon + heatwave + civic events combined).

**Competitive moat**: Zero competing product exists in this exact segment.
Government welfare boards (Rajasthan 2023, Karnataka 2024–25 bills) are
moving toward mandatory gig worker welfare — a parametric income insurance
product is the natural private-sector complement to those frameworks.

---

## Market Context

- India gig workforce: **12 million in FY2024–25**, growing at 17% CAGR
- Projected: **23.5 million by 2030**, **61.9 million by 2047** (NITI Aayog)
- 92% year-on-year increase in blue-collar gig hiring in 2024
- 78% of delivery workers earn under ₹2.5 lakh/year — zero financial buffer
- Real wages fell 11% since 2019 (NCAER 2022) — protection need is growing
- 0 statutory income protection mechanisms exist for gig workers currently
- India contributes 27% of global gig/freelancers — largest uninsured
  workforce segment in the formal-to-informal continuum

---

*Team RageByte | Guidewire DEVTrails 2026*
*Persona: Food Delivery (Zomato/Swiggy) | Platform: Android Native*
*Coverage: Income loss from external disruptions only*
*Premium cycle: Weekly | Payout: UPI instant transfer*
