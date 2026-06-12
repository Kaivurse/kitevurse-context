# CLAUDE-REFERENCE-FULL.md — KiteVurse Complete Technical Reference

**Last updated:** 2026-06-11 (manually updated — nightly auto-update job discontinued)
**Maintenance:** Manual. Update this file when architecture, phase status, or product decisions change.
**Source of truth for:** All KiteVurse context — DB schema, RPCs, Edge Functions, frontend, phase status.
**Do NOT use:** Notes/DBSCHEMA.md (pre-normalization snapshot, stale)

---

## SESSION RULES

### How sessions start
- Fetch and read this file at the start of every session
- Also fetch: `https://raw.githubusercontent.com/Kaivurse/kitevurse-context/refs/heads/main/CLAUDE-SECTIONS-MANUAL.md`
- Never ask Dan to re-explain context — derive it from these files

### How sessions end (Claude Code)
Every session ends with a standardised handoff block — no exceptions:

```
--- HANDOFF ---
Commit: <hash>
Branch: <branch>
Files changed: <list>
Test results: <pass/fail summary>
Next: <what to do next session>
--- END HANDOFF ---
```

### Non-negotiable session rules
- **One task per session.** No sprawl.
- **Every session needs tangible output** — something Dan can click, test, or see.
- **Always end with a commit** unless Dan explicitly says not to.
- **Never ask questions already answered** in this file or earlier in the conversation.
- **Never give Dan a list of things to remember.** Bake everything into files, scripts, automated processes.
- **Every Claude Code prompt must include:** complete ready-to-paste prompt in ONE block, thorough QA section (375px + 768px + 1280px, happy path + edge cases, console error check, `npm run build` pass), Step 0 verification gate.
- **Search conversation history before asking any question.** If the answer is there, use it.

---

## DAN'S WORKING PROFILE

- Best hours: 7am–12pm. Schedule hard decisions here.
- Needs tangible output every session. Something visible to touch, click, or test.
- Runs on momentum — never question completed work without a specific technical reason.
- **ADHD:** Never give lists of things to remember. One task at a time.
- Exceptional data analysis skills. Systems integration mindset.
- Gets derailed when Claude asks questions it already knows the answers to.
- All technical decisions delegated to Claude. Claude leads with a clear recommendation.

---

## PHASE PLAN & ROADMAP

### What KiteVurse Is

Trip advice from the most well-traveled kiter you know — who tells you the truth about a spot.

Not a booking site. Not a directory. The honest, AI-powered planning tool the kitesurfing community doesn't have yet.

### Business Model (Locked)

| Revenue Stream | Mechanic | Timeline |
|---|---|---|
| Booking.com affiliate | ~3–6% effective on accommodation referrals | 2–3 months post-launch (need traffic first) |
| Skyscanner CPC | Cost-per-click on flight searches | Live at launch |
| SafetyWing referral | $15–25 per signup | Live at launch |
| **KiteVurse Pro** | **$49/year** | **Month 9+** |

### The Five Phases

| Phase | Focus | Status |
|---|---|---|
| **1 — Data Complete** | Finish collection, build review queue, ingest to production | ✅ Complete |
| **2 — Frontend Integration** | Wire data into UI, mobile-first rebuild, all pages built | ✅ Complete (June 9, 2026) |
| **3 — Trip Engine Upgrade** | Smarter planner, better AI calls, voice, Off The Water, Day Plan generator | ✅ Complete (June 12, 2026)|
| **4 — Soft Launch + Community** | Vercel deploy, Facebook engagement, real users | 🔴 Not started |
| **5 — Agents + Pro + Scale** | Agentic system, KiteVurse Pro, expand to 76 destinations | 🔴 Not started |

---

### PHASE 2 — Complete ✅

**Completed June 9, 2026.** All pages built and shipped.

| Page / Feature | Status |
|---|---|
| Explorer page | ✅ Complete |
| Destination page | ✅ Complete |
| Trip plan intake (3-step modal) | ✅ Complete |
| Trip plan output | ✅ Complete |
| Authentication (magic link) | ✅ Complete |
| Save flow | ✅ Complete |
| Share flow | ✅ Complete |
| Profile page | ✅ Complete |
| Shared view | ✅ Complete |
| Tide module | ✅ Complete |

**Trip planner modal structure (locked):**
- Step 1 (You): group type, skill level, trip goal [Sessions / Progress / Explore / Chill]
- Step 2 (Your Setup): weight, gear situation, budget
- Step 3 (When & Where): nationality, arrival, departure, base (area picker from destination_areas)

Trip goal drives: verdict framing, lifestyle narrative, Off The Water content priorities, costs display.

**Section nav (locked):** WATER · THERE & IN · SLEEP · OFF THE WATER · COSTS

**Design tokens (locked):**
- Page bg: `#f6f4ef`, dark: `#0a0e14`, accent: `#2DABE5`, accent-deep: `#006194`
- Green: `#7ed3a2`, amber: `#d99a3a`
- Fonts: Inter Tight (display/headings), JetBrains Mono (labels/numbers/eyebrows)

---

### PHASE 3 — Active 🟡

**Phase 3 sequence (locked):**
1. Trip engine upgrade (discovery → implementation)
2. Voice update (back-to-back with engine upgrade)
3. Off The Water — Day Plan generator build
4. Phase 3.5: Explorer hero banner rebuild

**Current Phase 3 state:**

| Item | Status |
|---|---|
| P21 cost data collection (Perplexity) | ✅ Complete |
| Cost data ingested to production tables | ✅ Complete |
| Trip engine upgrade (4 calls) | ✅ Complete |
| KITEVURSE_VOICE system prompt | ✅ Added |
| Enriched DB data injected into prompts | ✅ Complete |
| Off The Water section built | ✅ Complete |
| Tools sheet cleanup (rename nowind → dayplan) | ✅ Complete |
| Day Plan generator | ✅ Complete |
| Explorer hero banner rebuild | 🔴 Not started |
| Migration 056 (tides cache) applied to live DB | 🔴 Pending — review all pending migrations before push |

**⚠️ Migration risk:** There are approximately 35+ pending migrations between 036 and current state. Review all before pushing to live DB. Do not push blindly.

---

### PHASE 4 — Groups Tool (Not started)

**Blocked until:** lat/lng backfill complete AND map build complete.

Groups Tool features:
- Map (KVMap component, Mapbox GL JS via react-map-gl)
- Tide Window Times (WorldTides integration already built)
- No Wind Day Generator (4 modes: mood-based, group-aware, location-aware, spontaneous tap)

**Day Generator architecture (Phase 4):**
- Extends `generateNoWindItinerary()` reusable function from Edge Function
- MUST be built as reusable — not rebuilt
- Input: venue data + lat/lng + group type + mood/energy level + base area
- Output: sequenced day with timeline, real venues, honest notes
- Hard dependency: lat/lng backfill complete (✅ done — 1,830 coordinates captured)
- Hard dependency: map build complete

---

### PHASE 5 — Agents + Pro + Scale

- KiteVurse Pro ($49/year) — don't build until 200+ free users actively using trip planning
- Scale to 76 destinations (already seeded as `is_active = false`)
- ⚠️ Fix DELETE + INSERT overwrite problem before building periodic refresh — it will overwrite manually curated data
- DENNIS dashboard (`/admin/jarvis`) — populate with real agent data in Phase 5

---

## ARCHITECTURE DECISIONS (LOCKED)

| Decision | What | Why |
|---|---|---|
| All DB access through RPCs | Never direct table queries from frontend or Edge Function | RLS on normalized tables requires SECURITY DEFINER |
| SECURITY DEFINER | Mandatory on all RPCs joining RLS-normalized tables | Missing = silent all-nulls bug, no error |
| pgcrypto RPCs | Need `SET search_path = public, extensions` | digest() requires extensions in path |
| Claude API never generates facts | Airport codes, wind data, hazards, school names from DB only | Mauritius hallucination (SIR vs MRU) made this non-negotiable |
| Slug routing | `/destinations/:slug` always | No ID routes |
| Tap-only trip planner | No text fields in modal steps 2–3 | Reduces friction, enforces structured data |
| `d.town` not `d.name` | SQL queries against destinations table use `d.town` | `d.name` does not exist — this mistake has been made repeatedly |
| Supabase client import | `@/integrations/supabase/client` | NOT `@/lib/supabase` |
| E2E tests location | `e2e/` directory | NOT `tests/` |
| Ingestion scripts | Must use `npx supabase db query --linked` | REST API with publishable key has no UPDATE permissions — silently fails |
| `research_staging.raw_response` | Cannot cast with `::jsonb` directly | Use SUBSTRING + POSITION to extract fields |
| `kite_schools` table | DROPPED | `schools` table is source of truth (183 rows) |
| `dest.id` (UUID) | FK for all supplementary table queries | Never use `dest.destination_id` (text code) as FK |
| DELETE + INSERT ingestion | Current pattern | Must switch to merge logic before Q3 refresh — will overwrite manually curated data |

---

## TRIP PLAN OUTPUT — CURRENT ARCHITECTURE

### Section Nav (locked)
`WATER · THERE & IN · SLEEP · OFF THE WATER · COSTS`

Five sections. The old six-section layout (which included separate LIFESTYLE and NO WIND sections) is gone.

### Edge Function: generate-trip-plan

**Current call count: 4 parallel calls** (down from original 10–11)

| Call | ID | Max Tokens | What Claude writes |
|---|---|---|---|
| 1 | `verdict` | 400 | GO/EYES OPEN/WAIT headline + 3 reasons in honest voice |
| 2 | `on_the_water_narrative` | 500 | 1–2 sentence narrative framing water conditions |
| 3 | `lifestyle_narrative` | 500 | Group-conditional lifestyle narrative (feeds Off The Water feel card) |
| 4 | `no_wind_day` | variable | Reusable no-wind itinerary via `generateNoWindItinerary()` |

**`generateNoWindItinerary()` function:**
- Declared as standalone reusable function BEFORE Deno.serve block
- Purpose: Groups Tool Day Generator (Phase 4) will extend this, not rebuild it
- Input: venue data + lat/lng + group type + mood/energy + base area
- Output: sequenced day with timeline, real venues, honest notes

**KITEVURSE_VOICE system prompt (applied to all calls):**
```
Write like a real person who kites. Use contractions. Prefer simple words.
Understate rather than oversell.
Avoid: 'presents,' 'establishes,' 'ecosystem,' 'optimal,' 'showcases,' 'leverages.'
Be honest about limitations. Specific beats vague. Short beats long.
If you'd never say it to a kiter at the bar, remove it.
```

**Known issue:** Rate limit risk on 4 simultaneous calls. Retry logic or sequential `no_wind_day` call needed.

### Verdict Zone (dark, always visible)

- `‹ START OVER` → resets to intake
- Meta chips: destination + dates + skill + gear + group
- `VerdictPill` (calculated from DB — not AI)
- Verdict headline + 3 reasons (from `verdict` AI call)
- Amber watch-out box (tide_notes or reality_check)
- Kite quiver card: weight bracket + season name, sizes from `scaleKiteSizes(kite_sizes_70kg, weightBracket)`

### Section 04 — Off The Water

**Structure (top to bottom):**

1. **Feel card** (dark): 3-sentence AI narrative, group-conditional. Fed by `lifestyle_narrative` Edge Function call. Shows skeleton while streaming.
2. **2×2 intent picker:** Recover / Stay active / Eat & drink / Sort things out. One pre-selected on load so section is never empty. Tapping an intent swaps venue cards inline — no navigation.
3. **Venue cards:** Icon tile design — accent blue background `rgba(45,171,229,0.12)`, accent blue icon `#1a7baa`, open layout with hairline separators. Name wraps freely. Rating right-aligned on meta row. Max 2 chips per card. `google_maps_url` link on its own line. No colour coding by category.
4. **"Generate day plan →" button** (secondary, bottom of section): Calls `setToolsOpen(true)` + `setToolsTab('dayplan')`. Opens Tools sheet to DAY PLAN tab. No pre-population of mood — kiter picks fresh.

**Intent → data source mapping:**
- Recover → `recovery_services` (service_type: massage, physio, spa)
- Stay active → `no_wind_activities` (category: yoga, gym)
- Eat & drink → `restaurants`, `nightlife_venues`
- Sort things out → `transportation`, pharmacies from `google_places_raw`

**Venue card field sources:**
- `recovery_services.service_type` → icon and intent mapping
- `no_wind_activities.category` → icon and intent mapping
- `google_places_raw.category` → icon and intent mapping
- `formatVenueName()` utility: strips all-caps Google strings, removes location suffixes (" - Diani", " Beach Kenya")

### Tools Sheet

Three tabs: **MAP · TIDES · DAY PLAN**

```typescript
type ToolsTab = 'map' | 'tide' | 'dayplan'
```

- **MAP** — KVMap component (Mapbox GL JS via react-map-gl). Built.
- **TIDES** — TideWindowChart. Built. Hidden when `tide_dependent = false`.
- **DAY PLAN** — 🟡 Placeholder only. Generator not yet built.

**"At the spot" strip label:** "Map · Tides · Day plan"
Tides chip hidden when `tide_dependent = false`.

### Day Plan Generator — Target Architecture (not yet built)

Lives in Tools sheet, DAY PLAN tab.

**UI:**
- Multi-select mood picker (reuses intent categories from Off The Water)
- Duration: Full day / Half day
- Time of day: Morning / Afternoon / Evening
- "Build my day →" button
- Output: sequenced itinerary with timeline, real venue cards, honest notes

**Backend:**
- Edge Function: `action: 'day_plan'` routing
- Calls `generateNoWindItinerary()` (already exists, reusable)
- Real venues injected from DB — no AI-invented venue names
- Discovery report: `docs/no-wind-generator-plan.md` (exists in repo — contains mood → data source mapping and DB query findings)

**Build sequence:**
1. Read `docs/no-wind-generator-plan.md` first
2. Session 1: Edge Function `action: 'day_plan'` routing + multi-mood support
3. Session 2: Frontend — mood picker, duration, time, generate button, timeline result cards

---

## 1. DATABASE TABLES

### 1A. Core Normalized Tables (10 tables)

All 10 tables have RLS enabled. All read by main RPCs via SECURITY DEFINER. ~101 rows per table (25 active + 76 inactive destinations).

**destination_conditions** — Wind, water, climate, season data

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| average_wind_knots | smallint |
| wind_reliability_score | smallint |
| gustiness_score | smallint |
| storm_risk_score | smallint |
| forecast_confidence_score | smallint |
| water_type | text[] |
| chop_level_score | smallint |
| wave_size_average | numeric(4,1) |
| launch_type | text[] |
| average_air_temp_c | smallint |
| average_water_temp_c | smallint |
| sun_exposure_score | smallint |
| peak_months | smallint[] |
| shoulder_months | smallint[] |
| off_season_months | smallint[] |
| rainy_season_months | smallint[] |
| best_month_start | smallint |
| best_month_end | smallint |
| parsed_data | jsonb |
| data_source | text |
| created_at / updated_at | timestamptz |

---

**destination_sports** — Discipline scores and skill level data

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| disciplines_supported | text[] |
| skill_levels_supported | text[] |
| beginner_friendliness_score | smallint |
| kitesurf_score | smallint |
| wingfoil_score | smallint |
| kitefoil_score | smallint |
| freestyle_score | smallint |
| freeride_score | smallint |
| wave_riding_score | smallint |
| downwind_score | smallint |
| created_at / updated_at | timestamptz |

---

**destination_safety** — Hazards and risk scores

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| jellyfish_risk_score | smallint |
| algae_risk_score | smallint |
| water_hazard_score | smallint |
| theft_risk_score | smallint |
| scam_risk_score | smallint |
| medical_access_score | smallint |
| crowding_score | smallint |
| created_at / updated_at | timestamptz |

---

**destination_budget** — Cost and accommodation data

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| budget_level | text |
| average_daily_budget_usd | numeric(8,2) |
| average_monthly_budget_usd | numeric(10,2) |
| average_meal_cost_usd | numeric(6,2) |
| accommodation_cost_score | smallint |
| scooter_rental_monthly_usd | numeric(8,2) |
| long_term_rental_availability | boolean |
| created_at / updated_at | timestamptz |

---

**destination_transport** — Getting there and getting around

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| nearest_airport | text |
| airport_distance_minutes | smallint |
| transportation_required | boolean |
| scooter_friendly | boolean |
| car_recommended | boolean |
| walkable | boolean |
| created_at / updated_at | timestamptz |

---

**destination_tides** — Tide-specific data

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| tide_dependent | boolean (default false) |
| best_tide | text |
| tide_notes | text |
| tidal_range | text |
| tidal_currents | boolean |
| booties_recommended | boolean |
| created_at / updated_at | timestamptz |

---

**destination_gear** — Wetsuit and equipment requirements

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| wetsuit_needed | boolean |
| recommended_wetsuit_thickness | text |
| created_at / updated_at | timestamptz |

---

**destination_lifestyle** — Digital nomad and community data

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| wifi_quality_score | smallint |
| coworking_available | boolean |
| beachfront_availability | boolean |
| digital_nomad_friendliness | smallint |
| facebook_groups_available | boolean |
| expat_community_strength | smallint |
| local_scene_size | smallint |
| created_at / updated_at | timestamptz |

---

**destination_infrastructure** — Schools (JSONB), facilities, gear, rescue

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| kite_schools | jsonb |
| launch_zones | jsonb |
| gear_rental | jsonb |
| gear_storage | jsonb |
| repair_services | jsonb |
| rescue_services | jsonb |
| storage_available | boolean |
| beach_assistance_available | boolean |
| wingfoil_schools_available | boolean |
| created_at / updated_at | timestamptz |

---

**destination_editorial** — Narrative and prose content

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| vibe_summary | text |
| best_for_summary | text |
| not_for_summary | text |
| reality_check | text |
| practical_notes | text |
| daily_life_summary | text |
| no_wind_activities | text |
| fitness_options | text |
| no_wind_activities_structured | jsonb |
| no_wind_verdict | text |
| cultural_notes_universal | text |
| cultural_notes_family | text |
| night_scene | text |
| areas_recommendation | text |
| scene_summary | text |
| proximity_reality | text |
| created_at / updated_at | timestamptz |

---

**destination_meta** — Data quality and research status

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| confidence_score | smallint |
| data_status | text |
| manually_verified | boolean (default false) |
| source_count | smallint |
| research_priority | smallint |
| needs_research | boolean (default false) |
| personal_experience | boolean (default false) |
| created_at / updated_at | timestamptz |

---

### 1B. Supplementary Content Tables (public read RLS — query directly, no RPC needed)

---

**schools** — Kite schools and instructors (183 rows)

⚠️ `kite_schools` table is DROPPED. This is the single source of truth.

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| school_name | text |
| slug | text |
| school_type | text[] |
| disciplines_supported | text[] |
| website / whatsapp / email / phone / instagram_url / google_maps_url | text |
| storage_available / rescue_available / gear_rental_available / foil_rental_available | boolean |
| wingfoil_lessons_available / kitefoil_lessons_available / repair_services_available | boolean |
| rating | numeric |
| review_count / price_level / trust_score | integer |
| manually_verified | boolean |
| certifications | text[] |
| lesson_price_usd_from | integer |
| accommodation_available | boolean |
| children_lessons_available / radio_helmets | boolean |
| gear_brand | text |
| kv_pick | boolean (default false — max 1 per destination, set manually) |
| created_at / updated_at | timestamptz |

---

**destination_areas** — Neighbourhood/area intel per destination (P14 output, ~175 rows)

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| name | text |
| distance_to_kite_spot_km | numeric |
| vibe | text |
| best_for | text |
| accommodation_options | text |
| restaurants_nightlife | text |
| price_relative | text |
| walkable_to_kite | boolean |
| kiter_verdict | text (BEST BASE / GOOD OPTION / TRADEOFF / SKIP) |
| sleep_summary | text (added migration 046) |
| eat_summary | text (added migration 046) |
| data_source | text |
| manually_verified | boolean |
| is_active | boolean |
| created_at / updated_at | timestamptz |

---

**kite_hubs** — Kite scene hubs and bases per destination (P15 output, ~147 rows)

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| name | text |
| type | text (school_base / beach_club / informal_spot) |
| location_description | text |
| school_names | text[] |
| facilities | text[] |
| vibe | text |
| crowd | text |
| accommodation_on_site | boolean |
| accommodation_notes | text |
| hours | text |
| kiter_verdict | text (SOCIAL HUB / GOOD BASE / QUIET OPTION / LESSONS ONLY) |
| data_source | text |
| manually_verified | boolean |
| is_active | boolean |
| created_at / updated_at | timestamptz |

---

**destination_wind_seasons** — Per-season wind data (supplementary, query directly)

Key columns: destination_id FK, season_name, months (smallint[]), avg_wind_knots_low/high, wind_direction, shore_angle, wind_angle_degrees, rideable_days_per_week_low/high, kite_sizes_70kg (text[]), chop, honest_take, is_active.

---

**kite_accommodations** — Accommodation options per destination

Key columns: destination_id FK, name, area_name, distance_to_launch_m, price_per_night_usd_low/high, community_favourite (boolean), gear_storage (boolean), kite_packages (boolean), website, google_maps_url, is_active.

---

**kite_gear_services** — Gear rental and repair services

Key columns: destination_id FK, service_type (rental/repair/storage), name, stranded_risk (LOW/MEDIUM/HIGH), price_per_day_usd, google_maps_url, is_active.

---

**kite_spots** — Per-spot geographic and safety data

Key columns: destination_id FK, spot_name, gps_lat, gps_lng, gps_source, spot_type, wind_angle, ideal_wind_direction, bottom_type, launch_depth, skill_level, crowd_level_peak, tide_dependent, data_source, is_active.

⚠️ Only `kite_spots` currently has coords natively. Venue tables had coords added via migration 057 (lat/lng backfill — 1,830 coordinates captured).

---

**no_wind_activities** — Activities for wind-free days

Key columns: destination_id FK, category (yoga/gym/coworking/cultural/tour/other), name, google_maps_url, description, services_offered (text[]), drop_in_price_usd, hourly_price_usd, opening_hours, beginner_friendly, drop_in_welcome, rating, worth_it_verdict, **latitude, longitude** (added migration 057), is_active.

---

**recovery_services** — Massage, physio, sports recovery

Key columns: destination_id FK, name, service_type (massage/physio/spa), google_maps_url, services_offered (text[]), price_per_hour_usd, walk_in_friendly, appointment_required, athlete_experience, why_good_for_kiters, rating, worth_it_verdict, **latitude, longitude** (added migration 057), is_active.

---

**restaurants** — Dining options

Key columns: destination_id FK, name, meal_type, cuisine_type, google_maps_url, average_meal_usd, opening_hours, good_for_solo/groups/families, rating, worth_it_verdict, **latitude, longitude** (added migration 057), is_active.

---

**nightlife_venues** — Bars, clubs, live music

Key columns: destination_id FK, name, venue_type, google_maps_url, vibe, beer_price_usd, good_for_solo/groups/kiter_hangout, rating, worth_it_verdict, **latitude, longitude** (added migration 057), is_active.

---

**transportation** — Scooter/car/transport vendors

Key columns: destination_id FK, vendor_name, transport_type, google_maps_url, daily_price_usd/weekly/monthly, board_bag_friendly, airport_pickup_available, trust_score, scam_risk_score, is_active.

---

**visa_requirements** — Entry requirements per destination/passport pair

Key columns: destination_id UUID FK (nullable), passport_country, passport_country_code, visa_required (nullable), visa_free_days, visa_type, cost_usd, apply_where, application_url, e_visa_available, voa_available, valid_as_of, manually_verified.

---

**destination_daily_living** — Practical on-the-ground info

Key columns: destination_id FK, transport_primary, transport_price_range, transport_negotiate (boolean), ride_app_available, payment_app_name, payment_app_tourist_ok, usd_accepted (boolean), tap_water_safe (boolean), sim_worth_it (boolean), sim_carrier, sim_where_to_buy, english_sufficient (boolean), english_notes, beach_dress, offbeach_dress, tip_restaurant_pct, bargaining.

---

**culture_safety** — Language, safety, customs per destination

Key columns: destination_id UUID UNIQUE FK, primary_language, english_tourist_areas, overall_safety_rating, kite_beach_safety_rating, petty_crime_risk, common_scams (text[]), dress_code_beach/town, tipping_restaurants_pct, alcohol_available, manually_verified.

---

**saved_trips** — User saved trip plans (migration 045)

| Column | Type |
|---|---|
| id | uuid PK |
| user_id | uuid FK → auth.users |
| destination_id | uuid FK → destinations |
| destination_slug | text |
| destination_name | text |
| nationality | text |
| arrival_date | date |
| departure_date | date |
| group_type | text |
| skill_level | text |
| weight_bracket | text |
| gear_situation | text |
| budget_level | text |
| verdict | text CHECK (GO/EYES_OPEN/WAIT) |
| share_token | text UNIQUE DEFAULT encode(gen_random_bytes(12), 'base64url') |
| created_at | timestamptz |

RLS: Users see/insert/delete own trips. Public read via share_token.

---

**destination_cost_staging** — P21 cost data staging (created Phase 3)

25 rows (one per active destination), status = ingested. Source data written to production tables: `kite_accommodations`, `kite_gear_services`, `schools`, `destination_daily_living`.

---

**airports** — 1,176 large airports + 5 destination-only airports

`airport_iata CHAR(3)` FK added to destinations table.

---

**destination_media** — Hero images

Key columns: id, destination_id (text — FK joins via `d.destination_id` not `d.id`), image_url, is_hero, created_at.

---

**research_staging** — Research pipeline staging

Key columns: destination_slug (text), prompt_id, prompt_name, status (pending_review/approved/parsed/rejected), raw_response, collected_at.

⚠️ `raw_response` cannot be cast with `::jsonb` directly — Perplexity sometimes prepends `#` or invalid chars. Always use `SUBSTRING + POSITION` to extract fields, or check with `LEFT(raw_response, 500)` first.

---

### 1C. Additional Tables (Phase 1 schema)

**coworking_spaces** — Co-working spaces and wifi cafés. Public read RLS.

**medical_emergency** — Emergency contacts, hospitals, rescue protocol. Public read RLS.

**open_meteo_raw** — Monthly historical weather (300 rows — 12/destination × 25 destinations). UNIQUE (destination_id, year, month).

**google_places_raw** — Venue records from Google Places API. UNIQUE (destination_id, category, place_name).

**seabreeze_raw** — Forum threads from seabreeze.com.au.

**social_intel_staging** — Raw ingested posts. ~300 pending rows post-dedup.

**approved_social_intel** — Audit trail for every approved staging batch.

**noaa_copernicus_raw** — Monthly wave + SST climatology (300 rows). Auto-approved, confidence 9.0.

**approval_queue** — Universal approval queue for all manual review workflows.

---

## 2. DATABASE FUNCTIONS / RPCs

### Active RPCs (used by frontend)

---

**explore_destinations_multi_sport_v1**
```
Parameters: input_month int, input_sports text[], input_region text, input_skill_level text
Returns: destination cards (23 columns), sorted by destination_strength_score DESC
SECURITY DEFINER: ✅
```
Reads from normalized tables. Only returns `is_active = true` destinations.

---

**get_destination_detail_by_slug_v1**
```
Parameters: input_slug text
Returns: single row with ~57 aliased fields
SECURITY DEFINER: ✅
```
All field names are **aliased** — never use raw DB column names.
Reads from: destinations, destination_conditions, destination_sports, destination_safety, destination_transport, destination_budget, destination_lifestyle, destination_infrastructure, destination_tides, destination_gear, destination_editorial, destination_media.

---

**get_schools_by_destination_v1**
```
Parameters: input_slug text
Returns: table of school records
SECURITY DEFINER: ✅
Sort: kv_pick DESC, manually_verified DESC, capability_count DESC, school_name ASC
```
Joins schools to destinations via destination_id. Never query `schools` table directly.

---

**get_fitness_by_destination_v1**
```
Parameters: input_slug text
Returns: table (one row per facility)
SECURITY DEFINER: ✅
```

---

### Admin RPCs

**get_pending_social_intel_v1** — lists pending staging rows with destination info
**approve_social_intel_row_v2(p_id, p_excluded_indices, p_custom_venues)** — 3 params (current version). The 2-param version was dropped in migration 032.
**reject_social_intel_row** / **flag_social_intel_row**
**get_all_destinations_for_admin** — all 101 destinations for admin dropdown
**add_manual_venue_v1** — writes directly to google_places_raw bypassing staging queue

---

### Data Ingestion RPCs

**ingest_open_meteo** — upserts open_meteo_raw. Requires pgcrypto → `SET search_path = public, extensions`.
**ingest_google_places** — upserts google_places_raw. Note: param is `input_category` not `category`.
**ingest_reddit_posts(input_slug, posts)** — write-only, no HTTP. Final signature: 2 params.
**ingest_seabreeze_data** / **ingest_rome2rio_data** / **ingest_noaa_copernicus_data**

---

## 3. SUPABASE EDGE FUNCTIONS

All in `supabase/functions/`. Deployed with `--no-verify-jwt`. Env var: `SERVICE_ROLE_KEY` (NOT `SUPABASE_SERVICE_ROLE_KEY` — Supabase blocks that prefix).

---

**generate-trip-plan** — Main trip plan generator

**Current architecture: 4 parallel Claude API calls**

| Call | Section ID | Max Tokens |
|---|---|---|
| 1 | `verdict` | 400 |
| 2 | `on_the_water_narrative` | 500 |
| 3 | `lifestyle_narrative` | 500 |
| 4 | `no_wind_day` | variable |

Output: NDJSON stream — one line per section as calls complete. Frontend renders DB data immediately; AI narrative hydrates when it lands.

`generateNoWindItinerary()` — standalone reusable function declared BEFORE Deno.serve block. Groups Tool Day Generator extends this function.

KITEVURSE_VOICE system prompt applied to all calls (see Trip Plan section above).

Enriched DB data injected into all prompts: wind seasons, kite hubs, daily living data, no-wind activities.

⚠️ Known issue: rate limit risk on 4 simultaneous calls. Needs retry logic or sequential `no_wind_day`.

---

**extract-social-intel** — Parses raw social posts via Claude API
**ingest-reddit** — Reddit RSS/Atom (note: may 403 from cloud IPs)
**fetch-open-meteo** — Monthly historical weather from Open-Meteo archive API
**search-google-places** — Google Places Text Search API (v1). Cost: ~$0.032/search. Env: `GOOGLE_PLACES_API_KEY`. Updated migration 058 to capture lat/lng from API response.
**parse-open-meteo** / **parse-google-places** — Read-only verification functions
**scrape-seabreeze-forums** / **scrape-rome2rio-transport** / **ingest-noaa-copernicus-data** — Data ingestion
**classify-facebook-posts** / **post-facebook-comments** — Facebook engagement pipeline

---

## 4. MIGRATION HISTORY

### Confirmed Applied (root `migrations/` + `supabase/migrations/`)

| Migration | Description | Status |
|---|---|---|
| 001–006 | Normalized tables, initial RPCs | ✅ Applied |
| 011 | explore_destinations_multi_sport_v1 | ✅ Applied |
| 012 | get_fitness_by_destination_v1 | ✅ Applied |
| 013–020 | Reddit ingest (final: write-only RPC, migration 019) | ✅ Applied |
| 021 | open_meteo_raw + ingest_open_meteo | ✅ Applied |
| 022 | google_places_raw + ingest_google_places | ✅ Applied |
| 023–028 | Seabreeze, Facebook, approval queue, Rome2rio, NOAA | ✅ Applied |
| 029 / 029b | Social intel review RPCs + safe JSON cast fix | ✅ Applied |
| 030 | kite_spots + staging dedup | ✅ Applied |
| 031 | spot_regulations + approve_v2 (2-param, superseded) | ✅ Applied |
| 032 | destination_kiter_logistics + approve_v2 (3-param, current) | ✅ Applied |
| 033 | get_all_destinations_for_admin + add_manual_venue_v1 | ✅ Applied |
| 034 | visa_requirements — add destination_id FK | ✅ Applied |
| 035 | Ingestion column fixes (data_source, is_active, visa_required nullable) | ✅ Applied |
| 036 | destination_areas + kite_hubs + editorial columns | ✅ Applied |

### Known Additions (Phase 2–3, exact numbers may vary)

| Migration | Description | Status |
|---|---|---|
| 045 | saved_trips table + RLS policies | Applied |
| 046 | destination_areas — add sleep_summary + eat_summary columns | Applied |
| 056 | Tides cache table | Built but **NOT YET APPLIED to live DB** |
| 057 | latitude/longitude columns on venue tables (recovery_services, no_wind_activities, restaurants, nightlife_venues, transportation, kite_accommodations) | Applied — 1,830 coords captured |
| 058 | ingest_google_places RPC — updated to capture lat/lng from API response | Applied |

⚠️ **~35+ pending migrations exist between 036 and current state.** Review ALL before pushing to live DB. Do not push blindly. Run `npx supabase migration list --linked` to see full state.

⚠️ **Do NOT apply migration 099** (if it exists) — drops source columns.

---

## 5. FRONTEND PAGES & COMPONENTS

### Pages (`src/pages/`)

**ExplorePage.tsx** — `/`
- RPC: `explore_destinations_multi_sport_v1`
- Hero, destination picker sheet (vaul), filter bar, destination card grid
- Default filters: current month, kitesurfing, all regions, all levels

**DestinationPage.tsx** — `/destinations/:slug`
- RPCs: `get_destination_detail_by_slug_v1`, `get_schools_by_destination_v1`, `get_fitness_by_destination_v1`
- Direct queries: destination_areas, kite_hubs, destination_wind_seasons, kite_accommodations, kite_gear_services, destination_daily_living, culture_safety, kite_spots
- Page structure: back button → overview zone → season switcher → My Base picker → invest footer. Ends at invest footer — nothing after.
- Single column all viewports

**TripPlanPage.tsx** — `/trip/:slug`
- Renders intake until submitted, then switches to output
- State: `'intake' | 'output'`
- Intake: 3-step modal (tap-only steps 2–3)
- Output: TripPlanOutput component

**ProfilePage.tsx** — `/profile`
- No session → show email input for magic link (don't redirect away)
- Data: `saved_trips` table
- Layout: 1-col mobile, 2-col tablet, 3-col desktop

**SharedTripView.tsx** — `/trip/shared/:token`
- Public, no auth required
- Queries `saved_trips` WHERE `share_token = token`
- TripPlanOutput in read-only mode

**Admin.tsx** — `/admin` (untouched)
**JarvisDashboard.tsx** — `/admin/jarvis` (DENNIS dashboard, PIN: 2601, all placeholder data)

---

### Key Components

**TripPlanOutput.tsx** — Trip plan output (main output component)

Section IDs and nav:
```
WATER · THERE & IN · SLEEP · OFF THE WATER · COSTS
id:  water    therein   sleep   offwater     costs
```

NDJSON stream section mapping:
| id | Content |
|---|---|
| `verdict` | Verdict zone headline + 3 reasons |
| `on_the_water_narrative` | Section WATER — wind narrative |
| `lifestyle_narrative` | Section OFF THE WATER — feel card |
| `no_wind_day` | Section OFF THE WATER — day itinerary fallback |

Tools sheet state:
```typescript
type ToolsTab = 'map' | 'tide' | 'dayplan'
const [toolsTab, setToolsTab] = useState<ToolsTab>('map')
const [toolsOpen, setToolsOpen] = useState(false)
```

---

**TripPlannerModal.tsx** — 3-step intake modal

Step structure:
- Step 1 (You): group type, skill level, trip goal [Sessions / Progress / Explore / Chill]
- Step 2 (Your Setup): weight, gear situation, budget
- Step 3 (When & Where): nationality, arrival, departure, base area picker

All steps tap-only except nationality search. Step indicator: `STEP X / 3 — [NAME]` (mono, accent). EXIT → `/destinations/:slug`.

---

**KVMap** — Mapbox GL JS via react-map-gl. Two contexts: `trip_plan` and `no_wind`. Dark minimal basemap. Three-layer system:
- Kite infrastructure (always-on)
- Trip picks (on by default)
- Category pills (off by default, togglable)

Category set: Coffee, Food, Market, Pharmacy, Scooter, Yoga, Gym, Massage, Nightlife.

Vaul bottom sheet on pin tap.

**Hard dependency:** lat/lng backfill complete (✅ done). Map build: Phase 4 Groups Tool.

---

**Shared UI components:**
- `VerdictPill` — GO (green) / EYES_OPEN (amber) / WAIT (red)
- `DeviceFrame` — mobile-only phone frame, invisible at `md:`
- `KVNav` — sticky top nav, dark bg, Logo + profile avatar
- `BottomTabBar` — mobile-only, 62px, EXPLORE / PLANNER / PROFILE
- `DestinationPickerSheet` — vaul Drawer, all active destinations, real-time search

---

### Frontend Rules (locked)

- Destination page: single-column all viewports, ends at invest footer (no sections after)
- Schools: compare CTA line only in trip plan output — NEVER school cards in trip plan
- DeviceFrame: intentional dual-render (mobile only, invisible at `md:+`) — not a bug
- `d.town` not `d.name` in SQL against destinations table — `d.name` does not exist
- Supabase client: `@/integrations/supabase/client` NOT `@/lib/supabase`
- Playwright E2E tests: `e2e/` directory NOT `tests/`

---

## 6. DATA INGESTION STATE

| Dataset | Count | Status |
|---|---|---|
| Active destinations | 25 | is_active = true |
| Inactive destinations (staged) | 76 | is_active = false |
| Normalized table rows | ~101/table | Complete |
| destination_areas | ~175 rows | Complete — 5–10 areas/destination |
| kite_hubs | ~147 rows | Complete — 2–10 hubs/destination |
| open_meteo_raw | 300 rows | 12/destination × 25 |
| noaa_copernicus_raw | 300 rows | 12/destination × 25 |
| seabreeze_raw | ~50 rows | Complete |
| social_intel_staging | ~300 pending | Post-dedup |
| google_places_raw | 2,264 rows | All categories, all destinations |
| Perplexity P7–P21 | All complete | Production tables populated |
| Venue lat/lng | 1,830 coords | Backfill complete (migration 057) |
| P21 cost data | 25 destinations | Ingested to production tables |

**Ingestion rule:** DELETE + INSERT pattern currently in use. Must switch to merge logic before Q3 refresh to avoid overwriting manually curated data.

**Ingestion scripts must use:** `npx supabase db query --linked` — NOT REST API with publishable key. `run_collector.ps1` sets the publishable key (anon role) which has no UPDATE permissions. PostgREST silently returns HTTP 204 for zero-row updates — script appears to succeed but writes nothing.

---

## 7. ENVIRONMENT

### Active Supabase Project: `xthaiitjoccrrqivjlor`

```
VITE_KITEVERSE_SUPABASE_URL    — active project URL  ✅ use this
VITE_KITEVERSE_SUPABASE_ANON_KEY — active anon key   ✅ use this
```

⚠️ Legacy project `ouqqfpqsbaijbinpyiio` — `VITE_SUPABASE_*` vars. DO NOT USE for new work.

### Edge Function Secrets

| Secret | Used By |
|---|---|
| `SERVICE_ROLE_KEY` | All write Edge Functions (NOT `SUPABASE_SERVICE_ROLE_KEY`) |
| `SUPABASE_URL` | Edge Functions needing project URL |
| `GOOGLE_PLACES_API_KEY` | search-google-places |
| `ANTHROPIC_API_KEY` | generate-trip-plan, extract-social-intel, classify-facebook-posts |
| `FACEBOOK_USER_TOKEN` | post-facebook-comments |

### Codebase

```
Repo:       C:\Users\danwi\kite-explorer
Branch:     feat/phase2-redesign
Dev server: localhost:8080 (npm run dev)
Shell:      PowerShell — no && chaining, use ; or separate commands
```

### Stack

| Library | Version |
|---|---|
| React | 18.3.1 |
| TypeScript | 5.8.3 |
| Vite | 5.4.19 |
| react-router-dom | 6.30.1 |
| @tanstack/react-query | 5.83.0 |
| tailwindcss | 3.4.17 (v3 — NOT v4) |
| @supabase/supabase-js | 2.105.3 |
| vaul | 0.9.9 |
| sonner | 1.7.4 |
| lucide-react | 0.462.0 |
| framer-motion | installed, approved |
| Vitest | 3.2.4 |
| Playwright | 1.60.0 (e2e/ directory) |

Radix UI primitives and shadcn/ui installed. Use them.

### Design Tokens (locked)

```
Page bg:         #f6f4ef
Dark bg:         #0a0e14
Accent:          #2DABE5
Accent-deep:     #006194
Green:           #7ed3a2
Amber:           #d99a3a
Muted text:      #6b7a8d
Label text:      #54606c
Inverted text:   #e8eaed
Border light:    rgba(10,14,20,0.09)
Border dark:     rgba(255,255,255,0.08)
```

Fonts: `'Inter Tight'` (display/headings), `'JetBrains Mono'` (labels/numbers/eyebrows).

Breakpoints: mobile < 768px, tablet 768–1279px (md:), desktop ≥ 1280px (lg:).
Test viewports: 375px, 390px, 768px, 1280px.

---

## 8. DATA COLLECTION SCRIPTS

All in `scripts/data-collection/` in repo.

| Script | Status | Purpose |
|---|---|---|
| `kitevurse_places_ingester.py` | ✅ Active | Moves approved Google Places rows → production venue tables. Updated to propagate lat/lng (migration 057). |
| `kitevurse_perplexity_parser.py` | ✅ Active | Reads research_staging → writes P7–P21 to production tables. Resume-safe via progress JSON. |
| `kitevurse_perplexity_collector_v4.py` | ✅ Active | Calls Perplexity API, writes to research_staging. P1–P13 disabled. |
| Retired collectors (v1–v3) | Retired | — |

**Launchers:** `run_ingester.ps1`, `run_parser.ps1` — both load service role key from Windows Credential Manager.

**Parser Prompt Map (P7–P21 all complete):**

| Prompt | Name | Target |
|---|---|---|
| P7 | Culture & Safety | culture_safety |
| P8 | Visa Requirements | visa_requirements |
| P9 | Wind & Water | destination_conditions |
| P10 | Destination Profile | destination_editorial |
| P11 | Spot Geometry | kite_spots |
| P12 | Regulations | spot_regulations |
| P13 | Logistics | destination_kiter_logistics |
| P14 | Neighborhood & Area Intel | destination_areas + editorial.areas_recommendation |
| P15 | Kite Hubs & Scene | kite_hubs + editorial.scene_summary + proximity_reality |
| P21 | Comprehensive Cost Data | kite_accommodations, kite_gear_services, schools, destination_daily_living |

---

## 9. MAINTENANCE

**Nightly auto-update job: DISCONTINUED.** This file is manually maintained. Update it when architecture, phase status, or product decisions change. Commit and push to `github.com/Kaivurse/kitevurse-context` (public) after updating.

Manual trigger (retired — scripts still in repo but not scheduled):
```powershell
.\scripts\maintenance\run_nightly_update.ps1
```

---

## 10. CRITICAL RULES SUMMARY

1. **SECURITY DEFINER** — every RPC reading from RLS-protected normalized tables. Missing = silent all-nulls, no error.
2. **pgcrypto** — any RPC using `digest()` must include `extensions` in search_path.
3. **Never generate factual data via Claude API** — airport codes, wind data, hazards, venue names from DB only.
4. **Anon key only in frontend** — never expose service_role key.
5. **Slug-based routing only** — no ID routes.
6. **RPC-first** — never query tables directly from frontend or Edge Function.
7. **`d.town` not `d.name`** — destinations table uses `d.town`. This mistake has been made repeatedly.
8. **`approve_social_intel_row_v2` signature** — 3 params: `(p_id, p_excluded_indices, p_custom_venues)`. 2-param version dropped in migration 032.
9. **Section ID is `pack`** — NOT `packing` (legacy bug reference — do not revert).
10. **`destination_media` JOIN** — joins on `d.destination_id` (text) not `d.id` (uuid).
11. **Ingestion scripts** — must use `npx supabase db query --linked`. REST API with publishable key silently fails.
12. **`kite_schools` table is DROPPED** — `schools` is source of truth (183 rows).
13. **`dest.id` (UUID)** — FK for all supplementary table queries. Never use `dest.destination_id` (text code) as FK.
14. **Off The Water section** — replaces both old Lifestyle (04) and No Wind (05). Section nav is 5 sections, not 6.
15. **Day Plan generator** — extends `generateNoWindItinerary()`. Must be reusable. Discovery report: `docs/no-wind-generator-plan.md`.

---

## 11. WHAT'S NEXT (Phase 3 active work)

**Immediate next task:** Day Plan generator build.

Prerequisites:
1. Read `docs/no-wind-generator-plan.md` — confirm it exists and contains mood → data source mapping
2. Confirm DAY PLAN tab placeholder state in TripPlanOutput.tsx
3. Session 1: Edge Function `action: 'day_plan'` routing + multi-mood support for `generateNoWindItinerary()`
4. Session 2: Frontend — mood picker (multi-select), duration, time of day, generate button, timeline result cards

**After Day Plan generator:**
- Explorer hero banner rebuild (Phase 3.5)
- Migration 056 review + apply (tides cache)
- Full pending migration review before any DB push
