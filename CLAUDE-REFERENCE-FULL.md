I'll carefully update only the specified sections: the header date, Section 6 numbers, and the Current Status table. Everything else remains unchanged.

```markdown
# CLAUDE-REFERENCE-FULL.md — KiteVurse Complete Technical Reference

Generated: 2026-05-25 | Last updated: 2026-05-30 (automated update — DB state queried 2026-05-30T07:45:30; schools deduped to 183 rows via migrations 043+044; kite_schools dropped)  
Source: schema_tables.json + all migration files + Edge Function source + frontend source  
Do NOT use Notes/DBSCHEMA.md — it is a pre-normalization snapshot and is stale.

---

## SESSION RULES

### How sessions start
- Dan pastes the latest commit hash at the top of his message
- Claude reads the hash and continues without re-explaining context — the commit is the source of truth
- Never ask Dan to re-explain what was done last session; derive it from the commit and CLAUDE.md

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

If a session ends without a commit, state the uncommitted files explicitly.

### Non-negotiable session rules
- **One task per session.** No sprawl.
- **Every session needs tangible output** — something Dan can click, test, or see. Backend-only is acceptable only if a visible result is impossible.
- **Always end with a commit** unless Dan explicitly says not to.
- **Never ask questions that are already answered** in this file, CLAUDE.md, or earlier in the conversation.
- **When Dan is in flow, keep going.** Do not interrupt with check-ins or confirmation requests.
- **Never give Dan a list of things to remember.** If something needs to happen, bake it into a file, script, or automated process.

---

## DAN'S WORKING PROFILE

### How Dan works
- Best hours: 7am–12pm. Schedule hard decisions and complex sessions here.
- Carries context between sessions mentally — arrives knowing what to do next.
- Needs tangible output every session. Something visible to touch, click, or test. Backend-only sessions feel incomplete and kill momentum.
- Runs on dopamine from visible progress. A working page beats 500 rows in a database every time.

### ADHD compensations (non-negotiable)
- Never give Dan a list of things to remember. Bake everything into files, scripts, and automated processes instead.
- Never question work Dan says is complete without a specific technical reason. Trust his assessment first.
- Search conversation history before asking any question. If the answer exists, use it.
- One task per session. No sprawl.
- When Dan is in flow, do not interrupt with questions. Keep going.

### Strengths to leverage
- Exceptional data analysis skills — he spots relationships between data sets that others miss.
- Systems integration mindset — thinks in connections, not features.
- Creative problem-solver under pressure — stays calm, evaluates, iterates, knows when to cut losses.

### What derails Dan
- Claude asking questions it already knows the answers to.
- Claude casting doubt on completed work.
- Sessions that end without something touchable.
- Manual maintenance tasks (he will forget them — automate everything).

### The goal
Get Dan to flow state as fast as possible and keep him there.
Every system, prompt, and process should serve that goal.

---

## PHASE PLAN & ROADMAP

*Merged from KiteVurse_Master_Roadmap_v2.md (May 24, 2026). Current Status updated May 26, 2026 to reflect Phase 1 complete, Phase 2 starting.*

---

### What KiteVurse Is

Trip advice from the most well-traveled kiter you know — who tells you the truth about a spot.

Not a booking site. Not a directory. The honest, AI-powered planning tool the kitesurfing community doesn't have yet.

---

### Business Model (Locked)

| Revenue Stream | Mechanic | Timeline |
|---|---|---|
| Booking.com affiliate | 4–6% on accommodation referrals | Live at launch |
| Skyscanner CPC | Cost-per-click on flight searches | Live at launch |
| SafetyWing referral | $15–25 per signup | Live at launch |
| **KiteVurse Pro** | **$49/year** | **Month 9** |

---

### The Five Phases

| Phase | Focus | Timeline | Status |
|---|---|---|---|
| **1 — Data Complete** | Finish collection, build review queue, ingest to production | Now → ~June 7 | ✅ Complete |
| **2 — Frontend Integration** | Wire new data into UI, mobile-first rebuild, fix all known bugs | June → mid-July | 🟡 Starting |
| **3 — Trip Engine Upgrade** | Smarter planner using real data, honest voice, new inputs | July | 🔴 Not started |
| **4 — Soft Launch + Community** | Vercel deploy, Facebook engagement, real users | August | 🔴 Not started |
| **5 — Agents + Pro + Scale** | Agentic system, KiteVurse Pro, expand to 76 destinations | Sept → Nov | 🔴 Not started |

---

### PHASE 1 — Data Complete ✅

**Complete as of May 28, 2026 (extended — P16–P19 + new tables added).**

All steps finished:
- Perplexity P7–P15 collected (v4 collector) and parsed to production tables for all 25 destinations
- Perplexity P16–P19 collected and parsed (v5 collector) — new tables kite_schools, kite_accommodations, kite_gear_services, destination_daily_living populated
- P9 parser upgraded: now also writes to `destination_wind_seasons` (migration 037) in addition to `destination_conditions`
- Google Places: 2,264 rows collected, admin review queue built and used, approved rows ingested
- Admin Review Queue: SocialIntelReviewQueue.tsx + AdminReviewQueue.tsx built and live at `/admin/review`
- Ingestion scripts: `scripts/data-collection/kitevurse_places_ingester.py` + `kitevurse_perplexity_parser.py` — complete, in repo
- destination_areas: ~177 rows (P14). kite_hubs: ~139 rows (P15). destination_editorial updated for all 25.
- destination_wind_seasons: 50 rows (P9 — migration 037). kite_schools: 112 rows (P16 — migration 038). kite_accommodations: 141 rows (P17 — migration 039). kite_gear_services: ~163 rows (P18 — migration 040). destination_daily_living: 25 rows (P19 — migration 041).
- Collector upgraded to v5 (`kitevurse_perplexity_collector_v5.py`) — fixed prompts P7/P9/P12/P13 (dual numbers, zero tolerance, confirmed names, per-spot regulations)
- `run_collector.ps1` added as Perplexity collector launcher (mirrors run_parser.ps1 pattern)
- `STANDARDS.md` created at `scripts/data-collection/STANDARDS.md` — coding standards for all data-collection scripts
- All data collection scripts moved into `scripts/data-collection/` in the kite-explorer repo

**Data source split (locked):**
- Google Places → all venue data (gyms, restaurants, massage, accommodation, transport, nightlife, yoga, pharmacies, gear)
- Perplexity P7–P15 → all context/narrative/kite-specific (safety, visa, wind, spot geometry, regulations, logistics, areas, hubs)

**⚠️ Ingestion uses DELETE + INSERT.** Do not build periodic refresh until this is replaced with merge logic — it will overwrite manually curated data.

---

### PHASE 2 — Frontend Integration 🟡

**This is the phase we're starting. It's the longest and most important before launch.**

Goal: All the new data we collected gets surfaced on the site. Site works on mobile. All known bugs fixed.

#### 2A — Mobile-First Rebuild

Mobile was never properly addressed. It's a standing gap on every page.

Every Claude Code prompt from here forward includes mobile responsiveness as a requirement. Not optional. Not an afterthought. Primary viewports to test: iPhone SE (375px) and iPhone 14 (390px).

**Known mobile bugs to fix:**
- BEST FOR / SKIP IF / WATCH OUT illegible on mobile (DestinationDetail hero)
- 6-stat hero band squashes on mobile
- These two ship together as one Claude Code session

#### 2B — Data Wiring: Destination Detail Sections

Wire approved, ingested data into frontend sections that are currently empty or placeholder.

| Section | Data Source | Current State | What Needs Wiring |
|---|---|---|---|
| 01 Conditions | Perplexity P9 (wind/water) | Placeholder / static | Seasonal wind table, kite size guide, water temp |
| 02 Stay & Ride | Google Places (accommodation) | Partial | Real venue cards with ratings, hours, Google Maps links |
| 03 No Wind | Google Places (gyms/yoga/massage/coworking) + Perplexity P7 | Partial | Venue cards per category, sparse data fallback |
| 04 Getting There | Perplexity P13 (logistics) | Known bugs | Airlines, customs tips, gear advice |
| 05 Daily Life | Google Places (restaurants/markets/pharmacies) + Perplexity P7 (safety/culture) | Partial | Venues, scam warnings, culture notes |

**Sparse data display rules (locked):**
- `data_availability.complete = true` → Show full section
- `data_availability.complete = false` + some data → Single "Local Resources" card: *"Limited formal infrastructure — here's what's available"*
- `data_availability.complete = false` + nothing found → 2-sentence Reality Check card: *"No formal yoga or gyms here. This is a pure kite destination."*
- Never show empty cards. Absence of data is honest.

#### 2C — Voice Update: Rewrite All AI-Generated Content

**The problem:** Current trip plan output sounds like AI — overcorrected, constructed, too precise.

**The fix:** Audit every Claude API prompt in the Edge Function (`generate-trip-plan`) and rewrite the system prompt and call-level instructions to enforce human voice.

Claude Code Starting Prompt:
```
Audit and rewrite the voice/tone of all Claude API calls in the
generate-trip-plan Edge Function.

Location: supabase/functions/generate-trip-plan/index.ts

For EVERY call (~10-11 parallel calls), add or update the system prompt:

"Write like a real person. Use contractions. Prefer simple words.
Understate rather than oversell. If you'd never say it conversationally,
remove it. Avoid: 'presents,' 'establishes,' 'ecosystem,' 'optimal,'
'aforementioned,' 'facilitated,' 'leverages.'
Be honest about limitations. Specific details beat vague praise."

Bad: "This destination presents an exceptional opportunity for progression riders"
Good: "This is a solid spot to improve"

Bad: "Meteorological patterns establish reliable 12-16 knot conditions"
Good: "You're looking at consistent 12-16 knot winds"

Test on: Dakhla (December), Tarifa (July), Boracay (November).
Read the output aloud. If it sounds like a brochure, rewrite it.
Mobile responsiveness: N/A — Edge Function only.
```


### PHASE 3 — Trip Engine Upgrade

**Make the trip planner smarter with all the new data.**

**What the engine has now:**
- 10–11 parallel Claude API calls (~44 seconds)
- Basic personalization (skill level, group size, dates, gear, accommodation style, budget)

**What it needs:**

New modal inputs (planned):
- Nationality (for visa logic) — must be tap selection, not text field
- Trip goals / achievement focus (learn backrolls, chill, explore, progress)
- Rest day style (explore/stay close/cowork/just want wind)

**Trip Personality Banner (Call 0):**
A new first section in Trip Plan Output summarizing who you are and what this trip is built around. Reads like: *"You're an intermediate rider with 7 days and full gear. You want to nail backrolls. Here's the plan."*

**Smarter data injection:**
Once real venue data is ingested (Phase 2), the Edge Function should pull from DB and inject into prompts:
- Real school names + pricing from schools table
- Real accommodation options from venue tables
- Real no-wind activities from Google Places data
- Real safety/culture notes from Perplexity P7

The AI shouldn't be generating venue names or safety info. It should be narrating around real DB data.

**Progression Path (Call 12, conditional):**
Only fires if skill goals were selected. Builds a day-by-day progression arc.

---

### PHASE 4 — Soft Launch + Community

#### Vercel Deployment Checklist
- [ ] Add `vercel.json` with SPA rewrite rules
- [ ] Set env vars: `VITE_KITEVERSE_SUPABASE_URL` + `VITE_KITEVERSE_SUPABASE_ANON_KEY`
- [ ] Confirm correct Supabase project (`xthaiitjoccrrqivjlor` — not the old one)
- [ ] End-to-end test all 25 destination pages
- [ ] Trip plan generation test: Dakhla, Tarifa, Boracay
- [ ] Mobile test on real iPhone (not just Chrome DevTools)

#### Facebook Community Strategy

The kitesurfing community lives on Facebook groups. That's where buying decisions happen.

**The core principle: be a kiter first, founder second.**

For every post that mentions KiteVurse, make 3 genuine community contributions first.

**Post types (rotate weekly):**

| Type | Frequency | What it looks like |
|---|---|---|
| Local knowledge | 2–3×/week | "Just back from Dakhla — thermals kick in reliably around 1pm in December. Happy to answer questions." |
| Honest destination take | 1×/week | "Boracay in November: wind's more consistent than the reputation suggests. Here's what I found." |
| Community question | 1–2×/week | "Anyone been to Phan Rang recently? Trying to nail the best window." |
| KiteVurse soft mention | 1–2×/month | "I built a tool that puts this kind of info together — worth a look if you're planning." |
| Trip plan share | 1–2×/month | Screenshot of a real KiteVurse trip plan for a destination the group is actively discussing |

**Tone rules:**
- First person: "I found," "in my experience," "what I noticed"
- Include honest negatives: "the accommodation there is genuinely limited"
- No marketing language. Ever.
- Drop links only when someone explicitly asked for recommendations

**What NOT to do:**
- Never cold-post "check out my new platform"
- Never respond to every destination question with a KiteVurse link
- Never post the same message across multiple groups
- Never use: "excited to share," "game changer," "revolutionary"

---

### PHASE 5 — Agents + Pro + Scale

#### KiteVurse Agentic System

**Agents to build (in order):**

| Agent | What it really is | Build when |
|---|---|---|
| Community submission form | Automated form → admin queue → your approval | Phase 5A (simplest — start here) |
| Spot verification via schools | Email outreach → follow-up → response parsing | Phase 5B |
| Facebook content scheduling | n8n + Claude API generates post drafts → you approve | Phase 5B (after you know what content works) |
| Analytics monitor | Scheduled report → Claude summarizes → Slack/email | Phase 5C (only when you have real traffic) |
| Apollo lead generation | Not relevant yet — no B2B sales motion defined | Phase 5D |

Start with the community submission form. Lowest risk, highest learning density.

Stack: Claude API + n8n (free, visual, no-code friendly)
Flow: Form submitted → Claude reviews for spam/quality → routes to admin queue → you approve → data hits DB

**The DENNIS dashboard** comes after you have 3–4 working agents. The dashboard is the orchestration layer. Don't build it first. (Note: DENNIS dashboard at `/admin/jarvis` was built early as a placeholder — populate it with real agent data in Phase 5.)

**Learning path:**
1. Read: Anthropic's agent design patterns (docs.anthropic.com)
2. Map the community form flow on paper: Trigger → Perception → Decision → Action → Loop
3. Build it in n8n (1–2 weeks)
4. Add state management for school follow-up agent (2 weeks)
5. Wire to Supabase DB
6. Populate DENNIS dashboard with real data

#### KiteVurse Pro ($49/year)

Don't build until you have 200+ free users actively using trip planning.

Likely scope:
- Side-by-side destination comparison
- Saved trip plans
- Month-by-month wind calendar for all destinations
- Early access to new destinations
- Advanced personalization (progression tracking)

#### Scale to 76 Staged Destinations

76 destinations are already seeded (`is_active = false`).
- Run Perplexity v4 for all 76
- Run Google Places for all 76
- Activate in batches of 10–15
- ⚠️ Fix DELETE + INSERT overwrite problem before building periodic refresh

---

### Architecture Decisions (Locked)

| Decision | What | Why |
|---|---|---|
| Data source split | Google Places = venues. Perplexity = context/narrative. | Never ask Perplexity to re-collect what Google Places has. |
| Structured JSON output | All Perplexity prompts return pure JSON | Eliminates 40% null rate from label mismatch. |
| Data availability flag | Every response includes `data_availability.complete` | Drives frontend display logic for sparse destinations. |
| Sparse data = honest info | Never show empty cards | Absence of data tells users something real about the destination. |
| RPC-first | Never direct table queries | RLS on normalized tables requires SECURITY DEFINER on every RPC. |
| SECURITY DEFINER | Mandatory on all RPCs joining normalized tables | Missing this = silent all-nulls bug with no error message. |
| Slug routing | `/destinations/:slug` always | No ID routes. |
| No text fields in modal | Tap-only Trip Planner | Reduces friction. Enforces structure. |
| Claude API never generates facts | No airport codes, wind data, hazards from AI | Mauritius hallucination (SIR vs MRU) made this non-negotiable. |
| Layer 1 + Layer 2 | GO/NO-GO card + 8-section accordion | Quick answer first, depth second. |
| Human voice | Anti-AI baseline in all content | Users trust honesty and imperfection over polished AI prose. |

---

### Current Status (Updated 2026-05-30 — DB state queried 2026-05-30T07:45:30)

| Item | Status |
|---|---|
| Perplexity v4 collection (P7–P15, all 25 destinations) | ✅ Complete |
| Perplexity v5 collection (P16–P19, all 25 destinations) | ✅ Complete |
| Perplexity parser (P7–P19 → production tables) | ✅ Complete |
| Google Places data (2,388 rows in google_places_raw) | ✅ Collected |
| Google Places admin review queue | ✅ Built + used |
| Google Places ingestion (staging → production) | ✅ Complete (300 approved_social_intel rows) |
| destination_wind_seasons (48 rows, P9 — migration 037) | ✅ Complete |
| destination_areas (177 rows, P14) | ✅ Complete |
| kite_hubs (139 rows, P15) | ✅ Complete |
| kite_schools — DROPPED (migration 044, 2026-05-28; data merged into schools) | ✅ Complete |
| kite_accommodations (141 rows, P17 — migration 039) | ✅ Complete |
| kite_gear_services (161 rows, P18 — migration 040) | ✅ Complete |
| destination_daily_living (25 rows, P19 — migration 041) | ✅ Complete |
| schools (183 rows — deduped via migration 043; single source of truth) | ✅ Complete |
| Data collection scripts in repo (`scripts/data-collection/`) | ✅ Complete |
| run_collector.ps1 launcher + STANDARDS.md | ✅ Complete |
| build_destination_object.py updated for P19 (destination_daily_living included in output) | ✅ Complete |
| Nightly auto-update script (CLAUDE-REFERENCE-FULL.md) | ✅ Complete |
| Public context mirror (kitevurse-context repo) | ✅ Complete |
| DENNIS ops dashboard (`/admin/jarvis`) | ✅ Built (placeholder data) |
| DestinationPage.tsx (dark hero, season switcher, MyBasePicker spine, SchoolCard — migration 046+047) | ✅ Complete (2026-05-29, 25/25 Playwright tests passing) |
| Design system + shared nav components | ✅ Complete (2026-05-29) |
| Mobile rebuild | 🟡 In progress |
| Data wiring (Sections 01–05 frontend) | 🟡 In progress |
| Voice rewrite (Edge Function) | 🔴 Not started |
| P0 bugs (6 items) | 🔴 Not fixed |
| Vercel deployment | 🔴 Not done |
| Facebook engagement | 🔴 Not started |
| Agentic system | 🔴 Not started |
| KiteVurse Pro | 🔴 Not scoped |

**Phase 2 priority order:**
1. Fix P0 mobile bugs (2A + 2D items 5–6) — 1 session
2. Wire data into frontend sections 01–05 — 1 session per section
3. Voice rewrite in Edge Function — 1 session
4. Fix remaining P0 bugs (items 1–4) — ~2 sessions
5. Deploy to Vercel
6. Start Facebook community

---

## 1. DATABASE TABLES

### 1A. Core Normalized Tables (10 tables, 101 rows each)

All 10 tables were created in `migrations/001_create_normalized_tables.sql` and populated in `migrations/002_migrate_data.sql`. Every table has RLS enabled. All are read by the main RPCs via `SECURITY DEFINER`. Row count: **101 rows per table** (25 active + 76 inactive destinations).

---

**destination_conditions** — Wind, water, climate, season data

| Column | Type |
|--------|------|
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
| parsed_data | jsonb (added migration 014) |
| parsed_at | timestamptz (added migration 014) |
| needs_refresh_by | date (added migration 014) |
| open_meteo_data | jsonb (added migration 014) |
| data_source | text (added migration 035) |
| created_at / updated_at | timestamptz |

---

**destination_sports** — Discipline scores and skill level data

| Column | Type |
|--------|------|
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
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| jellyfish_risk_score | smallint |
| algae_risk_score | smallint |
| water_hazard_score | smallint |
| theft_risk_score | smallint |
| scam_risk_score | smallint |
| medical_access_score | smallint |
| crowding_score | smallint |
| parsed_data | jsonb (added migration 014) |
| parsed_at | timestamptz (added migration 014) |
| needs_refresh_by | date (added migration 014) |
| created_at / updated_at | timestamptz |

---

**destination_budget** — Cost and accommodation data

| Column | Type |
|--------|------|
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
|--------|------|
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
|--------|------|
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
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| wetsuit_needed | boolean |
| recommended_wetsuit_thickness | text |
| created_at / updated_at | timestamptz |

---

**destination_lifestyle** — Digital nomad and community data

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| wifi_quality_score | smallint |
| coworking_available | boolean |
| beachfront_availability | boolean |
| digital_nomad_friendliness | smallint |
| facebook_groups_available | boolean |
| whatsapp_groups_available | boolean |
| telegram_groups_available | boolean |
| expat_community_strength | smallint |
| local_scene_size | smallint |
| parsed_data | jsonb (added migration 014) |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**destination_infrastructure** — Schools (JSONB), facilities, gear, rescue

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| kite_schools | jsonb |
| launch_zones | jsonb |
| gear_rental | jsonb |
| gear_storage | jsonb |
| repair_services | jsonb |
| rescue_services | jsonb |
| infrastructure_logistics | jsonb |
| storage_available | boolean |
| beach_assistance_available | boolean |
| wingfoil_schools_available | boolean |
| parsed_data | jsonb (added migration 014) |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**destination_editorial** — Narrative and prose content

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| vibe_summary | text |
| best_for_summary | text |
| not_for_summary | text |
| reality_check | text |
| practical_notes | text |
| daily_life_summary | text |
| common_frustrations | text |
| no_wind_activities | text |
| fitness_options | text |
| cafe_cowork | text |
| local_areas | text |
| cultural_notes | text |
| school_recommendations | text (legacy — schools table is now source of truth) |
| no_wind_activities_structured | jsonb |
| no_wind_verdict | text |
| cultural_notes_universal | text |
| cultural_notes_family | text |
| night_scene | text |
| areas_recommendation | text (added migration 036 — P14 output: overall neighbourhood recommendation paragraph) |
| scene_summary | text (added migration 036 — P15 output: kite scene overview for the destination) |
| proximity_reality | text (added migration 036 — P15 output: honest take on kite spot distance / logistics) |
| created_at / updated_at | timestamptz |

---

**destination_meta** — Data quality and research status

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| confidence_score | smallint |
| data_status | text |
| manually_verified | boolean (default false) |
| ai_generated_fields | text[] |
| source_count | smallint |
| research_priority | smallint |
| needs_research | boolean (default false) |
| research_notes | text |
| personal_experience | boolean (default false) |
| created_at / updated_at | timestamptz |

---

### 1B. Supplementary Content Tables

These tables have **public read** RLS policies (anon can read directly).

---

**schools** — Kite schools and instructors (183 records across 25 destinations — deduped 2026-05-28: 14 near-duplicates removed via migration 043, table merged from kite_schools via migration 042, kite_schools dropped via migration 044)

**Single source of truth for all school data.** Served by `get_schools_by_destination_v1`. `kv_pick` is set manually by Dan (max 1 per destination) — no automated process should ever set it to true.

**Data ownership (enforced by trigger `enforce_perplexity_no_insert` — migration 043):**
- Google Places → inserts new school rows (name, website, phone, google_maps_url, rating, review_count)
- Perplexity P16 → UPDATE only (enriches null fields: lesson_price_usd_from, certifications, accommodation_available). Never inserts. Parser: `parse_p16` in `kitevurse_perplexity_parser.py`.

**data_source values:** `perplexity` (original seeded rows), `perplexity_p16` (migration 042 inserts from deprecated kite_schools), `perplexity_enriched` (updated by parse_p16 enrichment), `google_places` (future Google Places inserts).

| Column | Type |
|--------|------|
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
| notes | text |
| skill_levels_taught | text[] |
| certifications | text[] |
| lesson_price_usd_from | integer |
| accommodation_available | boolean |
| group_size_max | integer |
| children_lessons_available / radio_helmets | boolean |
| gear_brand | text |
| kv_pick | boolean (default false — max 1 per destination, set manually) |
| created_at / updated_at | timestamptz |

---

**destination_wifi** — WiFi and connectivity data (8 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| avg_download_mbps / avg_upload_mbps | numeric |
| avg_latency_ms | integer |
| speed_consistency / outage_frequency / outage_typical_duration | text |
| seasonal_issues / peak_hours_description | text |
| peak_hour_slowdowns | boolean |
| wifi_quality_score / reliability_score / nomad_viability_score | smallint |
| mobile_backup_viable | boolean |
| best_carrier_backup / best_area_for_wifi / best_accommodations / best_cafes_coworking | text |
| video_calls_reliable / streaming_reliable / file_upload_reliable | boolean |
| nomad_verdict / nomad_verdict_notes | text |
| data_source | text |
| manually_verified | boolean |
| last_verified_at | date |
| notes | text |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**nightlife_venues** — Bars, clubs, live music venues (544 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| name | text |
| venue_type | text |
| address / google_maps_url / website / instagram_url / phone | text |
| vibe / atmosphere_notes | text |
| beer_price_usd / cocktail_price_usd / cover_charge_usd / typical_spend_usd | numeric |
| opening_time / closing_time / busy_nights / live_music_nights | text |
| crowd_type / music_genre | text |
| good_for_solo / good_for_groups / kiter_hangout | boolean |
| rating | numeric |
| review_count | integer |
| worth_it_verdict | text |
| tourist_trap / to_avoid | boolean |
| avoid_reason | text |
| data_source | text |
| manually_verified | boolean |
| last_verified_at | date |
| notes | text |
| is_active | boolean |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**no_wind_activities** — Activities for wind-free days (416 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| category | text |
| name | text |
| address / google_maps_url / website / phone / whatsapp / instagram_url | text |
| description | text |
| services_offered | text[] |
| drop_in_price_usd / hourly_price_usd / weekly_price_usd / monthly_price_usd | numeric |
| opening_hours | text |
| morning_available / afternoon_available / evening_available | boolean |
| beginner_friendly / drop_in_welcome | boolean |
| equipment_quality | text |
| ac_available / parking_available | boolean |
| rating | numeric |
| review_count | integer |
| worth_it_verdict | text |
| data_source | text |
| manually_verified | boolean |
| last_verified_at | date |
| notes | text |
| is_active | boolean |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**recovery_services** — Massage, physio, sports recovery (444 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| name | text |
| service_type | text |
| address / google_maps_url / website / phone / whatsapp | text |
| services_offered | text[] |
| description | text |
| price_per_hour_usd / session_price_usd / package_price_usd | numeric |
| opening_hours | text |
| walk_in_friendly / appointment_required / athlete_experience / kite_sport_relevant | boolean |
| why_good_for_kiters | text |
| rating | numeric |
| review_count | integer |
| worth_it_verdict | text |
| data_source | text |
| manually_verified | boolean |
| last_verified_at | date |
| notes | text |
| is_active | boolean |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**restaurants** — Dining options (623 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| name | text |
| meal_type / cuisine_type | text |
| address / google_maps_url / website / phone | text |
| budget_meal_usd / average_meal_usd / upscale_meal_usd | numeric |
| opening_hours / atmosphere | text |
| breakfast_served / lunch_served / dinner_served | boolean |
| reservations_required | boolean |
| good_for_solo / good_for_groups / good_for_families | boolean |
| vegetarian_options / vegan_options / gluten_free_options / seafood_heavy | boolean |
| food_safety_notes | text |
| rating | numeric |
| review_count | integer |
| local_recommendation / tourist_trap | boolean |
| worth_it_verdict / specialties | text |
| data_source | text |
| manually_verified | boolean |
| last_verified_at | date |
| notes | text |
| is_active | boolean |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**sim_card_options** — Mobile data options per destination (0 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| country_code / carrier_name / carrier_market_share / coverage_quality | text |
| tourist_sim_available | boolean |
| week_data_gb / week_cost_usd / month_data_gb / month_cost_usd | numeric |
| unlimited_available | boolean |
| purchase_locations | text[] |
| passport_required | boolean |
| activation_time | text |
| english_support | boolean |
| esim_available | boolean |
| setup_difficulty | text |
| hotspot_allowed | boolean |
| hotspot_speeds / hotspot_reliability | text |
| best_for_short_stay / best_for_long_stay / recommended | boolean |
| data_source | text |
| manually_verified | boolean |
| last_verified_at | date |
| notes | text |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**transportation** — Scooter/car/transport vendors per destination (464 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| vendor_name | text |
| transport_type | text |
| website / whatsapp / phone / email / facebook_url / google_maps_url | text |
| daily_price_usd / weekly_price_usd / monthly_price_usd | integer |
| board_bag_friendly / airport_pickup_available / delivery_available / insurance_included | boolean |
| trust_score / scam_risk_score | integer |
| manually_verified | boolean |
| notes | text |
| is_active | boolean (added migration 035) |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

**visa_requirements** — Entry requirements per destination/passport pair (251 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations (added migration 034, nullable) |
| destination_country_code | text (now nullable — was required) |
| destination_name | text (now nullable — was required) |
| passport_country / passport_country_code | text |
| visa_required | boolean (now nullable — migration 035) |
| visa_free_days | integer |
| visa_type / processing_days | integer |
| cost_usd | numeric |
| apply_where / application_url | text |
| required_documents | text[] |
| passport_validity_months | integer |
| return_ticket_required / proof_accommodation / proof_of_funds | boolean |
| funds_amount_usd | integer |
| e_visa_available | boolean |
| e_visa_processing_days / e_visa_cost_usd | numeric |
| e_visa_url | text |
| e_visa_multiple_entry / voa_available | boolean |
| voa_cost_usd | numeric |
| voa_payment_method | text |
| extension_available | boolean |
| extension_cost_usd | numeric |
| extension_days | integer |
| departure_tax_usd | numeric |
| departure_tax_in_ticket | boolean |
| overstay_fine_usd_per_day | numeric |
| overstay_enforcement | text |
| official_website / embassy_contact | text |
| data_source | text |
| manually_verified | boolean |
| last_verified_at / valid_as_of | date |
| notes | text |
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

---

### 1C. Phase 1 Schema Tables (migration 014)

---

**coworking_spaces** — Co-working spaces and wifi cafés (274 rows)

Key columns: destination_id FK, name, venue_type, address, google_maps_url, website, phone, wifi_speed_mbps_down/up, wifi_reliability, has_power_outlets, has_ac, has_phone_booths, price_model, day_pass_usd, monthly_membership_usd, min_spend_usd, opening_hours, noise_level, best_hours_for_work, rating_for_work, rating, worth_it_verdict, data_source, manually_verified, last_verified_at, notes, is_active, parsed_data, parsed_at, needs_refresh_by, created_at, updated_at.

RLS: Public read enabled.

---

**culture_safety** — Language, safety, customs, tipping per destination (25 rows)

Key columns: destination_id UUID UNIQUE FK, primary_language, english_tourist_areas, english_general_public, essential_phrases (jsonb), overall_safety_rating, kite_beach_safety_rating, safe_zones/avoid_zones (text[]), petty_crime_risk/violent_crime_risk, common_scams (text[]), night_safety_notes, solo_female_safety_notes, dress_code_beach/town, tipping_restaurants_pct/massage_usd/taxi_pct/hotel_usd_per_day, tipping_expected, photography_restrictions, religious_considerations, alcohol_available, lgbtq_safety_notes, data_source, manually_verified, notes, parsed_data, parsed_at, needs_refresh_by, created_at, updated_at.

RLS: Public read enabled.

---

**medical_emergency** — Emergency contacts, hospitals, rescue protocol per destination (0 rows)

Key columns: destination_id UUID UNIQUE FK, hospitals/clinics (jsonb), lifeguard_at_kite_spot, lifeguard_seasonal, water_rescue_boat, water_emergency_number, rescue_response_time_min, kite_school_rescue_protocol, pharmacy_name/address/hours_24hr, anti_inflammatories_otc, emergency_police/ambulance/fire/coast_guard/tourist_police, country_calling_code, standard_insurance_covers_kite, recommended_insurers (text[]), avg_insurance_2week_usd, schools_require_insurance, decompression_chamber_location/km, decompression_relevant, data_source, manually_verified, notes, parsed_data, parsed_at, needs_refresh_by, created_at, updated_at.

RLS: Public read enabled.

---

### 1D. Phase 1 Social Sourcing Tables (migration 015)

---

**social_intel_staging** — Raw ingested posts from all sources (352 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| source | text NOT NULL |
| destination_id | uuid FK → destinations NOT NULL |
| raw_post_text | text NOT NULL |
| post_url | text NOT NULL |
| author | text |
| posted_at | timestamptz |
| ingested_at | timestamptz default now() |
| sha256_hash | text UNIQUE (dedup key) |
| status | text default 'pending' (pending/approved/rejected) |
| admin_notes | text |
| created_at / updated_at | timestamptz |

Indexes: destination_id, status, source.
RLS: Public read. Admin write (is_admin JWT claim).
Unique partial index on (destination_id, source, coalesced_category) WHERE status='pending' (migration 030 — prevents duplicate pending rows).

**Status values by source:**
- `open_meteo` → `approved` (auto-approved, high confidence 9.5)
- `google_places` → `pending` (requires admin review)
- `seabreeze` → `pending` (requires admin review)
- `noaa_copernicus` → `approved` (auto-approved, confidence 9.0)

---

**wind_community_reports** — Parsed wind intelligence (approved staging records promoted here) (0 rows)

Key columns: destination_id FK, report_date, avg_wind_knots, conditions, hazard_flags (text[]), author_name, source, post_url, verified (default false), created_at, updated_at.

RLS: Public read. Admin write.

---

**accommodation_reviews** — Sourced accommodation feedback (0 rows)

Key columns: destination_id FK, accommodation_type, price_per_night_usd, rating INT CHECK (1–5), reviewer_name, review_text, source, post_url, verified, created_at, updated_at.

RLS: Public read. Admin write.

---

**destination_condition_updates** — Timestamped condition reports (0 rows)

Key columns: destination_id FK, condition_type, severity, description, source, post_url, verified_at, created_at, updated_at.

Condition types: wind, water, infrastructure, safety, accommodation, pricing.

RLS: Public read. Admin write.

---

### 1E. Phase 1 Data Source Tables

---

**open_meteo_raw** — Monthly historical weather per destination/year (migration 021) (300 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| year | int (default 2024) |
| month | int CHECK (1–12) |
| wind_knots | numeric (monthly max of daily max, in knots) |
| air_temp_c | numeric |
| water_temp_c | numeric (always NULL — archive API doesn't provide SST) |
| rainfall_mm | numeric |
| fetched_at | timestamptz |

UNIQUE (destination_id, year, month). RLS: Public read.

---

**google_places_raw** — Venue records from Google Places API (migration 022) (2,388 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| category | text (accommodation/restaurants/markets/kite_schools/gyms/massage/transport/gear_rental/pharmacies/yoga/nightlife) |
| place_name | text |
| address | text |
| rating | numeric |
| price_level | int (0=free, 1=inexpensive, 2=moderate, 3=expensive, 4=very_expensive) |
| phone / website / google_maps_url | text |
| hours | text (weekdayDescriptions joined with '; ') |
| fetched_at | timestamptz |

UNIQUE (destination_id, category, place_name). RLS: Public read.

---

**seabreeze_raw** — Forum threads scraped from seabreeze.com.au (migration 023) (26 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| spot_name / spot_url | text |
| thread_title / thread_url | text (canonical dedup key) |
| post_count | int |
| last_post_date | timestamptz |
| top_posts | jsonb ([{author, date, excerpt, sentiment}]) |
| conditions_mentioned | text[] |
| hazards_mentioned | text[] |
| equipment_discussed | text[] |
| raw_html | text (nullable — large) |
| fetched_at / created_at | timestamptz |

UNIQUE (destination_id, thread_url). RLS: Public read.

---

**rome2rio_raw** — Ground transport routes per destination (migration 027) (0 rows)

Key columns: destination_id FK, route_type, origin_name, origin_iata, destination_name, destination_iata, transport_mode, operator_name, duration_hours, price_usd_from, notes, raw_html, fetched_at, created_at.

Dedup: functional unique index with COALESCE(operator_name, '') to handle NULL operator_name safely.

RLS: Public read.

---

**noaa_copernicus_raw** — Monthly wave + SST climatology (migration 028) (300 rows)

Key columns: destination_id FK, year (default 2024), month CHECK(1–12), hs_mean/min/max/std (wave height in meters), tp_mean (wave period seconds), direction (compass), sst_mean/min/max (°C from ERA5 hourly), sst_anomaly_c (nullable), fetched_at.

UNIQUE (destination_id, year, month). RLS: Public read. Status in staging: `approved` (auto-approved, confidence 9.0).

---

**approved_social_intel** — Audit trail for every approved staging batch (migration 029) (300 rows)

| Column | Type |
|--------|------|
| id | uuid PK |
| staging_id | uuid FK → social_intel_staging (ON DELETE SET NULL) |
| destination_id | uuid FK → destinations |
| source / category | text |
| item_count | int |
| confidence_score | numeric |
| admin_notes / approved_by | text |
| approved_at / created_at | timestamptz |

RLS: Public read.

---

### 1F. Facebook Strategy Tables (migration 024)

---

**facebook_posts** — Raw posts crawled from Facebook groups via Apify (0 rows)

Key columns: id uuid PK, group_id/group_name, post_id (text UNIQUE), post_url, post_text, post_author, post_timestamp, comment_count, reaction_count, created_at, updated_at.

---

**facebook_engagement_queue** — Claude drafts + approval workflow (7 rows)

Key columns: id uuid PK, post_id (text), group_id, draft_comment (text), claude_tone, relevance_score (1–10), category, approval_status CHECK (pending/approved/rejected/edited), admin_edits, created_at, updated_at.

---

**facebook_engagement_log** — Permanent audit log of all posted comments (0 rows)

RLS: Public read + insert/update (MVP).

---

**approval_queue** — Universal approval queue for all manual review workflows (migration 025) (6 rows; 4 pending)

| Column | Type |
|--------|------|
| id | uuid PK |
| queue_type | text CHECK (facebook/email/content/social/other) |
| source_id | text |
| source_data | jsonb (shape varies per queue_type) |
| item_title / item_preview | text |
| draft_text | text |
| draft_metadata | jsonb |
| approval_status | text CHECK (pending/approved/rejected/edited/posted — 'posted' added in migration 026) |
| your_edits | text |
| your_decision_at | timestamptz |
| posted_at | timestamptz |
| posted_id / external_url | text |
| created_at / updated_at | timestamptz |

RLS: Public read/insert/update (MVP — tighten in Phase 5).

---

### 1G. Spot & Logistics Tables (migrations 030–032)

---

**kite_spots** — Typed per-spot geographic and safety data (migration 030) (117 rows)

Key columns: id uuid PK, destination_id FK, spot_name, gps_lat/gps_lng, gps_source, spot_type, wind_angle, ideal_wind_direction, wind_direction_range, bottom_type, launch_depth, skill_level, crowd_level_peak (1–10), beach_space_rigging, downwind_hazards, emergency_exit, tide_dependent, data_source (added migration 035), is_active (added migration 035), created_at, updated_at.

One destination may have 1–3 named spots.

---

**spot_regulations** — Operational rules, seasonal restrictions, rescue info per destination (migration 031) (27 rows)

Key columns: id uuid PK, destination_id UUID UNIQUE FK, rescue_operator_names (text[]), rescue_coverage_months, rescue_contact, rescue_response_time_mins, beach_patrol_seasonal/months, permit_required, permit_details/where_to_get, permit_cost_usd, seasonal_restrictions, created_at, updated_at.

One row per destination (UNIQUE on destination_id).

---

**destination_kiter_logistics** — Kite-travel-specific customs, airline policy, gear transport (migration 032) (27 rows)

Key columns: id uuid PK, destination_id UUID UNIQUE FK, customs_kite_gear, kite_bag_declared_value_usd, customs_practical_advice, drone_import_rules, airline_kite_policy, foil_transport_notes, gear_rental_available/notes, repair_services_available/notes, local_gear_shops (jsonb), created_at, updated_at.

---

### 1H. Perplexity Research Output Tables (migration 036)

These two tables are populated by the Perplexity parser (`scripts/data-collection/kitevurse_perplexity_parser.py`) after P14/P15 research runs. RLS: Public read on both. Both populated for all 25 active destinations as of 2026-05-25.

---

**destination_areas** — Neighbourhood/area intel per destination (P14 output) (177 rows)

One row per distinct area within a destination. Parser strategy: DELETE auto-generated rows (manually_verified=false) then INSERT fresh rows from P14 JSON.

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| name | text NOT NULL |
| distance_to_kite_spot_km | numeric |
| vibe | text |
| best_for | text |
| accommodation_options | text |
| restaurants_nightlife | text |
| price_relative | text |
| walkable_to_kite | boolean |
| kiter_verdict | text (BEST BASE / GOOD OPTION / TRADEOFF / SKIP) |
| data_source | text (default 'perplexity') |
| manually_verified | boolean (default false) |
| parsed_data | jsonb |
| parsed_at / needs_refresh_by | timestamptz / date |
| is_active | boolean (default true) |
| created_at / updated_at | timestamptz |

Row count: 177 rows (confirmed 2026-05-30).

---

**kite_hubs** — Kite scene hubs and bases per destination (P15 output) (139 rows)

One row per distinct hub/base. Parser strategy: DELETE auto-generated rows (manually_verified=false) then INSERT fresh rows from P15 JSON. P15 also writes `scene_summary` and `proximity_reality` to `destination_editorial`.

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| name | text NOT NULL |
| type | text (school_base / beach_club / informal_spot) |
| location_description | text |
| school_names | text[] |
| facilities | text[] |
| vibe | text |
| crowd | text (beginner-heavy / mixed / intermediate-advanced) |
| accommodation_on_site | boolean (default false) |
| accommodation_notes | text |
| hours | text |
| kiter_verdict | text (SOCIAL HUB / GOOD BASE / QUIET OPTION / LESSONS ONLY) |
| data_source | text (default 'perplexity') |
| manually_verified | boolean (default false) |
| parsed_data | jsonb |
| parsed_at / needs_refresh_by | timestamptz / date |
| is_active | boolean (default true) |
| created_at / updated_at | timestamptz |

Row count: 139 rows (confirmed 2026-05-30).

---

### 1I. Perplexity P16–P18 Tables (migrations 037–040)

These four tables are populated by `kitevurse_perplexity_parser.py` after P9/P16/P17/P18 research runs using the v5 collector. All have RLS public read. All were populated for all 25 active destinations as of 2026-05-28.

---

**destination_wind_seasons** — Per-season wind profile per destination (P9 output) (48 rows)

One row per wind season. Parser strategy: DELETE auto-generated rows (manually_verified=false) then INSERT fresh rows from P9 JSON. P9 also continues to update `destination_conditions` (the single-averaged profile is preserved for backwards compatibility).

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| season_name | text NOT NULL (e.g. "Kusi", "Trade Wind", "Summer") |
| local_name | text (local name if one exists) |
| months | smallint[] (e.g. {6,7,8,9,10}) |
| avg_wind_knots_low | smallint |
| avg_wind_knots_high | smallint |
| wind_direction | text (compass e.g. "SE", "NE") |
| wind_angle_degrees | smallint (0–359, meteorological FROM) |
| shore_angle | text (e.g. "cross-onshore", "side-shore") |
| rideable_days_per_week_low | smallint |
| rideable_days_per_week_high | smallint |
| kite_sizes_70kg | text[] (e.g. {"9m","11m","13m"}) |
| chop | text (e.g. "low", "choppy") |
| honest_take | text (1-2 sentence plain-language description) |
| data_source | text (default 'perplexity') |
| manually_verified | boolean (default false) |
| parsed_at / needs_refresh_by | timestamptz / date |
| created_at / updated_at | timestamptz |

Row count: 48 rows (confirmed 2026-05-30).

---

**kite_schools** — ⚠️ DROPPED (migration 044, 2026-05-28) — data merged into `schools`.

~~Kite school lesson/amenity intel per destination (P16 output) (113 rows)~~ All data merged into `schools` via migration 042, deduped via migration 043, table dropped via migration 044. P16 Perplexity parser writes to `schools` going forward, not `kite_schools`.

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| school_name | text NOT NULL |
| iko_certified | boolean (default false) |
| price_per_hour_usd | numeric(8,2) |
| equipment_included | boolean (default true) |
| beginner_friendliness | smallint CHECK (1–5) |
| website | text |
| languages | text[] |
| accommodation_packages | boolean (default false) |
| lesson_packages | jsonb ({beginner, intermediate, advanced} text descriptions) |
| data_source | text (default 'perplexity_sonar_pro') |
| manually_verified | boolean (default false) |
| parsed_data / parsed_at / needs_refresh_by | jsonb / timestamptz / date |
| created_at / updated_at | timestamptz |

Row count: 112 rows (confirmed 2026-05-28 — table since dropped).

---

**kite_accommodations** — Kite-community accommodation intel per destination (P17 output) (141 rows)

One row per property. Captures which properties kiters actually stay at, proximity to launch, gear storage, and kite package availability. Parser strategy: DELETE auto-generated rows then INSERT from P17 JSON.

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| property_name | text NOT NULL |
| distance_to_launch_m | integer (metres from property to main kite launch) |
| gear_storage | boolean (default false) |
| community_favourite | boolean (default false — repeatedly cited by kiters in forums/groups) |
| price_per_night_usd_low | integer (budget / cheapest room) |
| price_per_night_usd_high | integer (best / most expensive room) |
| kite_packages | boolean (default false — bundled lesson + stay deals) |
| beach_access | boolean (default false — direct beach access from property) |
| website | text |
| data_source | text (default 'perplexity_sonar_pro') |
| manually_verified | boolean (default false) |
| parsed_data / parsed_at / needs_refresh_by | jsonb / timestamptz / date |
| created_at / updated_at | timestamptz |

Row count: 141 rows (confirmed 2026-05-30).

---

**kite_gear_services** — Gear rental operators and repair shops per destination (P18 output) (161 rows)

One row per operator/shop. `service_type` = 'rental' or 'repair'. Destination-level `stranded_risk` (low/medium/high) is stored on every row for that destination. Parser strategy: DELETE auto-generated rows then INSERT from P18 JSON.

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| operator_name | text NOT NULL |
| service_type | text NOT NULL CHECK ('rental' / 'repair') |
| brands | text[] (rental only — e.g. {'Duotone','Cabrinha'}) |
| fleet_condition | text (rental only — 'excellent'/'good'/'fair'/'poor') |
| foil_available | boolean (rental only — default false) |
| price_per_day_usd | integer (rental only — all-in daily rate) |
| repairs_offered | text[] (repair only — e.g. {'bladder','canopy','bar/lines','board'}) |
| turnaround_days | integer (repair only — typical turnaround) |
| bars_available | boolean (repair only — replacement bars/lines for sale) |
| stranded_risk | text CHECK ('low'/'medium'/'high') — same value on all rows for a destination |
| stranded_risk_notes | text |
| website | text |
| data_source | text (default 'perplexity_sonar_pro') |
| manually_verified | boolean (default false) |
| parsed_data / parsed_at / needs_refresh_by | jsonb / timestamptz / date |
| created_at / updated_at | timestamptz |

Row count: 161 rows (confirmed 2026-05-30).

---

**destination_daily_living** — Practical daily-life logistics per destination (P19 output) (25 rows)

One row per destination. UPSERT on `destination_id`. Covers payment methods, transport, language, SIM cards, water/food safety, dress codes, and tipping. `supermarkets` column is always null (reserved for future sourcing).

| Column | Type |
|--------|------|
| id | uuid PK |
| destination_id | uuid FK → destinations (UNIQUE) |
| payment_app_name | text |
| payment_app_tourist_ok | boolean |
| payment_app_how_to | text |
| usd_accepted | boolean |
| card_reliability | text |
| atm_availability | text |
| transport_primary | text |
| transport_price_range | text |
| transport_negotiate | boolean |
| ride_app_available | text |
| board_bag_transport | text |
| car_rental | text |
| english_sufficient | boolean |
| english_notes | text |
| key_phrases | jsonb (array of {phrase, meaning, why_it_matters}) |
| sim_worth_it | boolean |
| sim_carrier | text |
| sim_where_to_buy | text |
| tap_water_safe | boolean |
| food_safety_notes | text |
| supermarkets | jsonb (always null — reserved for future sourcing) |
| beach_dress | text |
| offbeach_dress | text |
| cultural_context | text |
| bargaining | text |
| photography_notes | text |
| tip_restaurant_pct | integer |
| tip_service_providers | text |
| tipping_importance | text |
| data_source | text (default 'perplexity_sonar_pro') |
| manually_verified | boolean (default false) |
| needs_refresh_by | date |
| created_at / updated_at | timestamptz |

Row count: 25 rows (confirmed 2026-05-30).

---

### 1J. Supporting Tables

**destination_media** — Hero images and photos stored in Supabase Storage.
Key columns: id, destination_id (text — FK joins via `d.destination_id` not `d.id`), image_url, is_hero, created_at.

**airports** — 1,176 large airports + 5 destination-only airports.
airport_iata CHAR(3) FK added to destinations table.

**research_staging** — Research pipeline staging table (exists in live_table_coverage view). (810 rows)
Key columns: destination_slug (text), prompt_id, prompt_name, status (pending_review/approved/parsed/rejected), raw_response, collected_at.

**destination_fitness_summary** — (25 rows)

**destination_fitness_facilities** — (287 rows)

---

## 2. DATABASE FUNCTIONS / RPCs

### Active RPCs (used by frontend)

---

**explore_destinations_multi_sport_v1** (migration 011)

```
Parameters: input_month int, input_sports text[], input_region text, input_skill_level text
Returns: table of destination cards (23 columns)
SECURITY DEFINER: ✅ (required — reads RLS-protected normalized tables)
Status: Working ✅
```

Returns destination cards sorted by `destination_strength_score DESC`. Score = season(35%) + wind_reliability(25%) + sport(25%) + confidence(10%) + data_status(5%).

Reads from: `destination_conditions`, `destination_sports`, `destination_budget`, `destination_editorial`, `destination_meta`, `destination_media`.

Frontend: `src/hooks/useExploreDestinations` → Explore page. Only returns `is_active = true` destinations.

---

**get_destination_detail_by_slug_v1** (migration 006, supabase/migrations version)

```
Parameters: input_slug text
Returns: single row with ~57 aliased fields
SECURITY DEFINER: ✅
Status: Working ✅
```

Reads from: `destinations`, `destination_conditions`, `destination_sports`, `destination_safety`, `destination_transport`, `destination_budget`, `destination_lifestyle`, `destination_infrastructure`, `destination_tides`, `destination_gear`, `destination_editorial`, `destination_media`.

⚠️ All field names are **aliased** — never use raw DB column names. See CLAUDE-REFERENCE.md alias table.

Frontend: `DestinationDetail.tsx`.

---

**get_schools_by_destination_v1**

```
Parameters: input_slug text
Returns: table of school records
SECURITY DEFINER: ✅
Status: Working ✅ (verified on dakhla-morocco, kalpitiya-sri-lanka)
```

Joins schools to destinations via `destination_id` (schools table has no destination_slug column).
Sort: kv_pick DESC, manually_verified DESC, (rescue+rental+storage count) DESC, school_name ASC.

Frontend: `DestinationDetail.tsx` (Section 02 On The Water).

---

**get_fitness_by_destination_v1** (migration 012)

```
Parameters: input_slug text
Returns: table (one row per facility; facility columns NULL if none exist)
SECURITY DEFINER: ✅
Status: Working ✅
```

Joins: `destination_fitness_summary` + `destination_fitness_facilities` via destination_id. Returns summary fields repeated on every row + one facility per row. Sort: priority ASC, distance_from_spot_km ASC. Only returns `is_active = true` destinations.

Frontend: `FitnessSection.tsx` via `useFitnessFacilities` hook.

---

### Data Ingestion RPCs

---

**ingest_open_meteo** (migration 021)

```
Parameters: input_slug TEXT, year_data JSONB ([{month, wind_knots, air_temp_c, rainfall_mm}])
Returns: TABLE(destination_name TEXT, months_upserted INT, status TEXT)
SECURITY DEFINER: ✅
Permissions: authenticated, service_role
```

Upserts 12 rows into `open_meteo_raw`. Writes 1 audit row to `social_intel_staging` (status='approved'). Uses pgcrypto digest() — requires `SET search_path = public, extensions`.

Called by: `fetch-open-meteo` Edge Function.

---

**ingest_google_places** (migration 022)

```
Parameters: input_slug TEXT, input_category TEXT, places JSONB
Returns: TABLE(destination_name TEXT, places_upserted INT, status TEXT)
SECURITY DEFINER: ✅
Permissions: authenticated, service_role
```

Note: parameter is `input_category` (not `category`) to avoid plpgsql ambiguity with `google_places_raw.category` column.

Upserts rows into `google_places_raw`. Writes 1 staging row to `social_intel_staging` (status='pending' — requires admin review). Uses pgcrypto digest() for sha256 dedup.

Called by: `search-google-places` Edge Function.

---

**ingest_reddit_posts** (migration 020 — final architecture)

```
Parameters: input_slug TEXT, posts JSONB ([{raw_post_text, post_url, author, posted_at}])
Returns: TABLE(destination_name TEXT, new_posts_count INT, status TEXT)
SECURITY DEFINER: ✅
Permissions: authenticated, service_role
```

Write-only RPC. No HTTP calls (previous pg_net approach in migrations 013–018 was incorrect and superseded by migration 019). Edge Function owns HTTP; this RPC owns DB. Deduplicates via SHA-256 hash. Inserts into `social_intel_staging` (status='pending').

Called by: `ingest-reddit` Edge Function.

---

**ingest_seabreeze_data** (migration 023)

```
Parameters: input_slug TEXT, threads JSONB
Returns: TABLE(destination_name TEXT, threads_upserted INT, status TEXT)
SECURITY DEFINER: ✅ (inferred — writes to RLS-protected table)
```

Upserts threads into `seabreeze_raw`. Writes staging audit row (status='pending', confidence 7.5).

Called by: `scrape-seabreeze-forums` Edge Function.

---

**ingest_rome2rio_data** (migration 027)

```
Parameters: input_slug TEXT, routes JSONB
Returns: TABLE(...)
SECURITY DEFINER: ✅ (inferred)
```

Upserts routes into `rome2rio_raw`. Writes staging row (status='pending', confidence 7.0).

Called by: `scrape-rome2rio-transport` Edge Function.

---

**ingest_noaa_copernicus_data** (migration 028)

```
Parameters: input_slug TEXT, monthly_data JSONB
Returns: TABLE(...)
SECURITY DEFINER: ✅ (inferred)
```

Upserts 12 monthly rows into `noaa_copernicus_raw`. Writes staging row (status='approved', confidence 9.0).

Called by: `ingest-noaa-copernicus-data` Edge Function.

---

### Admin Review Queue RPCs

---

**get_pending_social_intel_v1** (migration 029b — safe JSON cast fix)

```
Parameters: p_source TEXT DEFAULT NULL, p_limit INT DEFAULT 50, p_offset INT DEFAULT 0
Returns: TABLE(id, source, destination_id, destination_slug, destination_name, category, item_count, confidence_score, post_url, admin_notes, created_at, ...)
SECURITY DEFINER: NOT FOUND — needs investigation
```

Returns pending rows from `social_intel_staging` with destination info. Includes safe JSON cast guards (CASE WHEN raw_post_text LIKE '{%' guard) to handle non-JSON rows without throwing.

Used by: `SocialIntelReviewQueue.tsx`.

---

**approve_social_intel_row** (migration 029 — original v1)

```
Parameters: p_id UUID
Returns: JSON
SECURITY DEFINER: ✅ (required — social_intel_staging has is_admin RLS)
```

Marks staging row approved. Writes audit record to `approved_social_intel`.

---

**approve_social_intel_row_v2** (migration 032 — current version with 3 params)

```
Parameters: p_id UUID, p_excluded_indices INT[] DEFAULT NULL, p_custom_venues JSONB DEFAULT NULL
Returns: JSON
SECURITY DEFINER: ✅
```

Allows excluding individual venues from a Google Places batch before approval. Custom venues are appended to the kept venues. `places_fetched` updated to reflect (kept scraped venues) + (manually added venues). Seabreeze rows: exclusions safely ignored (no venue array).

Migration 031 created the 2-param version. Migration 032 dropped that and recreated with 3 params.

Used by: `SocialIntelReviewQueue.tsx`.

---

**reject_social_intel_row** (migration 029)

```
Parameters: p_id UUID, p_reason TEXT DEFAULT NULL
Returns: JSON
SECURITY DEFINER: ✅
```

---

**flag_social_intel_row** (migration 029)

```
Parameters: p_id UUID, p_note TEXT
Returns: JSON
SECURITY DEFINER: ✅
```

---

**get_all_destinations_for_admin** (migration 033)

```
Parameters: none
Returns: TABLE(id UUID, name TEXT, country TEXT)
SECURITY DEFINER: ✅
```

Returns all 101 destinations (active + inactive) for the Add Venue modal dropdown. Sorted by country → town.

Used by: `Admin.tsx` New Venue modal.

---

**add_manual_venue_v1** (migration 033)

```
Parameters: input_destination_id UUID, input_category TEXT, input_place_name TEXT, input_address TEXT, input_phone TEXT, input_website TEXT, input_rating NUMERIC, input_price_level INT, input_google_maps_url TEXT, input_hours TEXT
Returns: JSON
SECURITY DEFINER: ✅ (google_places_raw has RLS)
```

Writes a single venue row directly to `google_places_raw`, bypassing `social_intel_staging`. For admin-added venues from personal knowledge (no review queue needed — Dan would have to approve his own entries).

Used by: `Admin.tsx` New Venue modal.

---

## 3. SUPABASE EDGE FUNCTIONS

All functions in `supabase/functions/`. All deployed with `--no-verify-jwt` (internal pipeline tools). Service role key env var: `SERVICE_ROLE_KEY` (not `SUPABASE_SERVICE_ROLE_KEY` — Supabase blocks that prefix for custom secrets).

---

### generate-trip-plan

**File:** `supabase/functions/generate-trip-plan/index.ts`  
**Purpose:** Main trip plan generator — streams 10–11 Claude API calls simultaneously via custom `fireAndQueue` pattern.  
**Inputs:** POST body with destination slug + user modal inputs (nationality, skill level, gear situation, who's travelling, accommodation type, achievement goals, top priority, trip dates/length).  
**Outputs:** NDJSON stream — one line per section as calls complete (fastest first). Sections identified by `id` field.  
**External APIs:** Anthropic Claude API (`claude-sonnet-4-6`), streaming mode.  
**DB tables read:** All destination fields via `get_destination_detail_by_slug_v1` RPC. Schools via `get_schools_by_destination_v1`.  
**DB tables written:** None (read-only at request time).  

Call Map:

| Call | Section ID | Max Tokens |
|------|-----------|-----------|
| 1 | `overview` | 700 |
| 2 | `on_the_water` | 800 |
| 3 | `visa` | 600 |
| 4 | `flights` | 700 |
| 5 | `getting_around` | 600 |
| 6 | `accommodation` | 800 |
| 7 | `budget` | 800 |
| 8 | `day_structure` (3 days) | 1200 |
| 9 | `pack` | 600 |
| 10 | `reality_check` | 600 |
| 11 | `day_structure_expanded` (Day 4+, conditional on totalNights > 3) | 2000 |

⚠️ Section ID is `pack` — NOT `packing`. Frontend renders by matching on `id`.

---

### extract-social-intel

**File:** `supabase/functions/extract-social-intel/index.ts`  
**Purpose:** Parses raw social posts into structured intelligence using Claude API.  
**Inputs:** POST `{ destination: string, source: string, posts: [{ raw_text, post_url, author, posted_at }] }`. Max 50 posts.  
**Outputs:** JSON array of `IntelligenceResult` objects.  
**External APIs:** Anthropic Claude API (`claude-sonnet-4-6`, max_tokens=500, temperature=0.2).  
**DB tables read:** None.  
**DB tables written:** None (caller writes to DB).  

Claude extraction fields: intelligence_type, wind_knots, conditions, hazard_flags, rating, review_text, condition_type, severity, sentiment, confidence (0–100), key_quote.

---

### ingest-reddit

**File:** `supabase/functions/ingest-reddit/index.ts`  
**Purpose:** Fetches Reddit posts via RSS/Atom feed for r/kiteboarding, r/wingfoiling, r/kitesurfing. Calls `ingest_reddit_posts` RPC to dedup and store.  
**Inputs:** POST `{ destination_slug: string }`  
**Outputs:** JSON with destination_name, new_posts_count, status.  
**External APIs:** Reddit RSS/Atom (public, no auth). Subreddits: kiteboarding, wingfoiling, kitesurfing.  
**DB tables written:** `social_intel_staging` (via `ingest_reddit_posts` RPC, status='pending').  

⚠️ Reddit blocks cloud IPs (AWS, Vercel, Supabase). This function may 403 from hosted environment. Decision: skip Reddit for Phase 1 (DECISION 5).  
Uses regex-based XML parsing — no external library dependency. Max 100 posts. 90-day lookback window.

---

### fetch-open-meteo

**File:** `supabase/functions/fetch-open-meteo/index.ts`  
**Purpose:** Fetches daily historical weather from Open-Meteo archive API, aggregates to monthly, writes to DB.  
**Inputs:** POST `{ destination_slug: "dakhla-morocco" }` or `{ destination_slug: "all" }`  
**Outputs:** JSON with fetch summary per destination.  
**External APIs:** `https://archive-api.open-meteo.com/v1/archive` (free, no auth). Parameters: wind_speed_10m_max, temperature_2m_mean, rain_sum (daily data, wind_speed_unit=kn).  
**DB tables read:** `destinations` (latitude, longitude).  
**DB tables written:** `open_meteo_raw` (12 monthly rows), `social_intel_staging` (1 audit row, status='approved') — both via `ingest_open_meteo` RPC.  

Note: Uses daily data (366 rows) aggregated to monthly in-function — the archive API's monthly aggregation doesn't support wind_speed_10m_max. 120ms delay between destinations.

---

### parse-open-meteo

**File:** `supabase/functions/parse-open-meteo/index.ts`  
**Purpose:** Read-only — queries `open_meteo_raw` and outputs NDJSON for verification.  
**Inputs:** POST `{}` or `{ destination_slug: string, year: number }`  
**Outputs:** NDJSON — one line per destination with monthly + annual aggregates.  
**External APIs:** None.  
**DB tables read:** `open_meteo_raw`.  
**DB tables written:** None.

---

### search-google-places

**File:** `supabase/functions/search-google-places/index.ts`  
**Purpose:** Fetches venue data from Google Places Text Search API (v1) for 11 categories per destination.  
**Inputs:** POST `{ destination_slug: string }` or `{ destination_slug: "all" }`  
**Outputs:** JSON with category fetch summary.  
**External APIs:** `https://places.googleapis.com/v1/places:searchText`. Env var: `GOOGLE_PLACES_API_KEY`. Cost: ~$0.032/search (Advanced fields). 11 categories × 25 destinations = ~$8.80/full run.  
**DB tables read:** `destinations` (town, country).  
**DB tables written:** `google_places_raw` (upsert), `social_intel_staging` (status='pending') — via `ingest_google_places` RPC.  

200ms delay between category calls. maxResultCount: 10.

---

### parse-google-places

**File:** `supabase/functions/parse-google-places/index.ts`  
**Purpose:** Read-only — queries `google_places_raw` and outputs NDJSON for verification.  
**Inputs:** POST `{}` or `{ destination_slug: string, category?: string }`  
**Outputs:** NDJSON — one line per venue.  
**External APIs:** None.  
**DB tables read:** `google_places_raw`.  
**DB tables written:** None.

---

### scrape-seabreeze-forums

**File:** `supabase/functions/scrape-seabreeze-forums/index.ts`  
**Purpose:** Scrapes Seabreeze.com.au forums for wind conditions, hazards, and community insights per destination.  
**Inputs:** POST `{ destination_slug: string }` or `"all"`  
**Outputs:** JSON with scrape summary.  
**External APIs:** `seabreeze.com.au` (HTML scraping via Cheerio). Rate limit: 1 req/sec with exponential backoff.  
**DB tables written:** `seabreeze_raw`, `social_intel_staging` (status='pending', confidence 7.5) — via `ingest_seabreeze_data` RPC.  

Note: Uses forum search rather than spot-specific pages because most KiteVurse destinations are international (Seabreeze is primarily Australian). Uses spot mapping JSON for coverage type. NEVER calls Claude API.

---

### scrape-rome2rio-transport

**File:** `supabase/functions/scrape-rome2rio-transport/index.ts`  
**Purpose:** Fetches ground transport routes from Rome2Rio for KiteVurse destinations.  
**Inputs:** POST `{ destination_slug: string }` or `"all"`  
**Outputs:** JSON with transport summary.  
**External APIs:** `rome2rio.com` (HTML scraping). Primary: `__NEXT_DATA__` JSON extraction (Next.js SSR). Fallback: CSS selectors. 2s between requests + exponential backoff.  
**DB tables written:** `rome2rio_raw`, `social_intel_staging` — via `ingest_rome2rio_data` RPC.  

Route types: airport_to_destination, local_transport, inter_destination. NEVER calls Claude API.

---

### ingest-noaa-copernicus-data

**File:** `supabase/functions/ingest-noaa-copernicus-data/index.ts`  
**Purpose:** Fetches monthly wave + SST climatology from Open-Meteo Marine API (ERA5 backing data).  
**Inputs:** POST `{ destination_slug: string }` or `"all"`  
**Outputs:** JSON with ingest summary.  
**External APIs:** `https://marine-api.open-meteo.com/v1/marine` (free, no auth). Fields: wave_height_max, wave_direction_dominant, wave_period_max (daily), sea_surface_temperature (hourly → monthly mean). Year: 2024.  
**DB tables read:** `destinations` (latitude, longitude).  
**DB tables written:** `noaa_copernicus_raw` (12 monthly rows), `social_intel_staging` (status='approved', confidence 9.0) — via `ingest_noaa_copernicus_data` RPC.  

200ms between destinations. NEVER calls Claude API.

---

### classify-facebook-posts

**File:** `supabase/functions/classify-facebook-posts/index.ts`  
**Purpose:** Classifies raw Facebook posts using Claude API, stages draft comments in `approval_queue` for manual review.  
**Inputs:** POST `{ posts: [...] }` (from Apify webhook) OR `{ post_ids: [...] }` (UUIDs from facebook_posts table).  
**Outputs:** JSON with queue summary.  
**External APIs:** Anthropic Claude API (relevance scoring, draft comment generation). Assigns relevance_score (1–10), category, tone. Returns validated JSON.  
**DB tables read:** `facebook_posts` (when post_ids mode).  
**DB tables written:** `approval_queue` (primary, status='pending'), `facebook_engagement_queue` (secondary audit trail, non-blocking).  

Duplicate post_ids skipped unless `force_reprocess=true`.

---

### post-facebook-comments

**File:** `supabase/functions/post-facebook-comments/index.ts`  
**Purpose:** Posts approved comments to Facebook via Graph API. Marks rows as 'posted'.  
**Inputs:** POST `{}` (queries pending approved items internally).  
**Outputs:** JSON with post summary.  
**External APIs:** Facebook Graph API v20.0. Env vars: `FACEBOOK_USER_TOKEN` (personal access token with publish_to_groups permission).  
**DB tables read:** `approval_queue` (WHERE queue_type='facebook' AND approval_status IN ('approved', 'edited') AND posted_at IS NULL).  
**DB tables written:** `approval_queue` (sets approval_status='posted'). Requires migration 026 ('posted' added to CHECK constraint).  

Text priority: `your_edits` (Danny's edits) → `draft_text` (original Claude draft).

---

## 4. MIGRATION HISTORY

### Root Folder (`migrations/`) — All Applied ✅

| File | Description | Status |
|------|-------------|--------|
| `001_create_normalized_tables.sql` | Created all 10 normalized destination tables + indexes | ✅ Applied |
| `002_migrate_data.sql` | Copied data from destinations → 10 normalized tables. ⚠️ Ran 4× — DISTINCT ON in RPC CTEs deduplicates | ✅ Applied |
| `003_create_rpc_v2.sql` | Created `get_destination_detail_by_slug_v2` (not used by frontend) | ✅ Applied |
| `004_add_nowind_columns.sql` | Added no-wind structured columns to destinations | ✅ Applied |
| `005_create_fitness_tables.sql` | Created `destination_fitness_summary` and `destination_fitness_facilities` tables | ✅ Applied |
| `006_rewrite_v1_to_join_normalized.sql` | Rewrote `get_destination_detail_by_slug_v1` to join normalized tables | ✅ Applied (superseded by supabase/migrations/006) |

---

### Supabase Folder (`supabase/migrations/`) — Applied Status per File

| File | Description | Status |
|------|-------------|--------|
| `006_update_rpc_v1_normalized_joins.sql` | Updated `get_destination_detail_by_slug_v1` — supersedes root/006 | ✅ Applied |
| `011_update_explore_rpc_normalized_joins.sql` | Rewrote `explore_destinations_multi_sport_v1` to join normalized tables. Uses DISTINCT ON CTEs to handle duplicate rows. | ✅ Applied |
| `012_add_fitness_rpc.sql` | Created `get_fitness_by_destination_v1` RPC (SECURITY DEFINER) | ✅ Applied |
| `013_create_rpc_ingest_reddit_posts.sql` | Original ingest_reddit_posts with pg_net HTTP approach — **SUPERSEDED** by migration 019. The pg_net-within-transaction approach is architecturally incorrect. | ✅ Applied (superseded) |
| `014_phase1_schema_update.sql` | Created coworking_spaces, culture_safety, medical_emergency tables. Added parsed_data/parsed_at/needs_refresh_by to 12 tables. Added open_meteo_data column to destination_conditions. Recreated staging_pending, staging_coverage, live_table_coverage views. | ✅ Applied |
| `015_create_social_sourcing_tables.sql` | Created 4 social sourcing tables: social_intel_staging, wind_community_reports, accommodation_reviews, destination_condition_updates | ✅ Applied |
| `016_update_ingest_reddit_posts_use_vault.sql` | Added Supabase Vault for API key storage (interim approach, superseded by 019) | ✅ Applied (superseded) |
| `017_fix_ingest_reddit_posts_search_path.sql` | search_path fix for pgcrypto (superseded by 019) | ✅ Applied (superseded) |
| `018_fix_ingest_reddit_posts_pgnet_scalar.sql` | Fixed pg_net scalar return type (superseded by 019) | ✅ Applied (superseded) |
| `019_restructure_ingest_reddit_posts_write_only.sql` | **Correct final architecture**: write-only RPC, no HTTP. Edge Function → Reddit API → ingest_reddit_posts RPC → DB. | ✅ Applied |
| `020_fix_ingest_reddit_posts_search_path_extensions.sql` | Added `extensions` to search_path for pgcrypto digest() | ✅ Applied |
| `021_open_meteo_tables.sql` | Created open_meteo_raw table + ingest_open_meteo RPC (SECURITY DEFINER) | ✅ Applied |
| `022_google_places_tables.sql` | Created google_places_raw table + ingest_google_places RPC (SECURITY DEFINER) | ✅ Applied |
| `023_seabreeze_tables.sql` | Created seabreeze_raw table + ingest_seabreeze_data RPC | ✅ Applied |
| `024_facebook_strategy_tables.sql` | Created facebook_posts, facebook_engagement_queue, facebook_engagement_log tables | ✅ Applied |
| `025_universal_approval_queue.sql` | Created approval_queue table (generic queue for all manual approvals) | ✅ Applied |
| `026_approval_queue_posted_status.sql` | Added 'posted' status to approval_queue CHECK constraint | ✅ Applied |
| `027_rome2rio_tables.sql` | Created rome2rio_raw table + ingest_rome2rio_data RPC | ✅ Applied |
| `028_noaa_copernicus_tables.sql` | Created noaa_copernicus_raw table + ingest_noaa_copernicus_data RPC | ✅ Applied |
| `029_social_intel_review_rpcs.sql` | Created approved_social_intel table + get_pending_social_intel_v1, approve_social_intel_row (v1), reject_social_intel_row, flag_social_intel_row RPCs | ✅ Applied |
| `029b_fix_safe_json_cast.sql` | Fixed `get_pending_social_intel_v1` — wrapped ::jsonb casts in CASE WHEN guards for non-JSON rows | ✅ Applied |
| `030_kite_spots.sql` | Created kite_spots table (per-spot geographic and safety data) | ✅ Applied |
| `030_dedup_staging.sql` | Deduped social_intel_staging (453 → 300 rows). Added partial unique index on (destination_id, source, coalesced_category) WHERE status='pending' | ✅ Applied |
| `031_spot_regulations.sql` | Created spot_regulations table (one row per destination) | ✅ Applied |
| `031_approve_with_exclusions.sql` | Created approve_social_intel_row_v2 (2 params: id + excluded_indices) | ✅ Applied (superseded by 032) |
| `032_destination_kiter_logistics.sql` | Created destination_kiter_logistics table (customs, airline policy, gear transport) | ✅ Applied |
| `032_approve_v2_custom_venues.sql` | Dropped 2-param v2, recreated approve_social_intel_row_v2 with 3 params (+ p_custom_venues JSONB) | ✅ Applied |
| `033_add_manual_venue.sql` | Created get_all_destinations_for_admin() + add_manual_venue_v1() RPCs | ✅ Applied |
| `034_visa_requirements_add_destination_id.sql` | Added destination_id UUID FK (nullable) to visa_requirements. Added UNIQUE(destination_id, passport_country_code). | ✅ Applied |
| `035_ingestion_column_fixes.sql` | Added data_source/is_active to kite_spots; is_active to transportation; data_source to destination_conditions; made visa_required nullable | ✅ Applied |
| `036_destination_areas_kite_hubs.sql` | Created `destination_areas` (P14 — neighbourhood intel, one row per area) + `kite_hubs` (P15 — kite scene hubs, one row per hub). Added `areas_recommendation`, `scene_summary`, `proximity_reality` columns to `destination_editorial`. RLS public read on both new tables. Idempotent: DROP/CREATE policies, column fixups for reruns. | ✅ Applied |
| `037_destination_wind_seasons.sql` | Created `destination_wind_seasons` (P9 — per-season wind profile, one row per season per destination). RLS enabled, public read. Written by `parse_p9` in the parser. 48 rows across 25 destinations. | ✅ Applied |
| `038_create_kite_schools.sql` | Created `kite_schools` (P16 — Perplexity-sourced school lesson/amenity intel). Separate from `schools` (Google Places). RLS public read. 112 rows across 25 destinations. | ✅ Applied |
| `039_create_kite_accommodations.sql` | Created `kite_accommodations` (P17 — kite-community accommodation intel: proximity to launch, gear storage, kite packages). RLS public read. 141 rows across 25 destinations. | ✅ Applied |
| `040_create_kite_gear_services.sql` | Created `kite_gear_services` (P18 — gear rental operators + repair shops per destination). `service_type` = 'rental'/'repair'. Includes `stranded_risk` (low/medium/high) at destination level. RLS public read. 161 rows across 25 destinations. | ✅ Applied |
| `041_destination_daily_living.sql` | Created `destination_daily_living` (P19 — payment, transport, language, SIM, water safety, dress codes, tipping per destination). UPSERT on destination_id, one row per destination. RLS public read. 25 rows across 25 destinations. | ✅ Applied |
| `042_merge_kite_schools_into_schools.sql` | Merged `kite_schools` into `schools`: reset all 101 kv_pick=true flags; updated websites on exact matches; inserted 96 unmatched kite_schools rows (kv_pick=false, manually_verified=false); marked kite_schools deprecated. schools: 197 rows. | ✅ Applied |
| `043_schools_dedup_and_process_rules.sql` | (1) Added `data_source` text column to schools; (2) set data_source='perplexity' for original rows (slug IS NOT NULL), 'perplexity_p16' for migration 042 inserts; (3) copied missing websites from duplicate rows to keepers; (4) deleted 14 near-duplicate rows (keep row with notes, delete row without notes — same school, different Perplexity name normalisation); schools: 197→183 rows; (5) added `enforce_perplexity_no_insert` trigger — blocks any INSERT with data_source='perplexity'. Google Places owns inserts; Perplexity P16 does UPDATE only. | ✅ Applied |
| `044_drop_kite_schools.sql` | Dropped `kite_schools` table. Data fully merged into `schools` (migration 042), deduped (043), table dropped in 044. No RPCs, Edge Functions, or parsers reference kite_schools any more. | ✅ Applied |

**⚠️ Migrations NOT Applied:**
- `supabase/migrations/099` (if it exists) — drops source columns. DO NOT APPLY.
- Migrations 008–010 in supabase/migrations/ — redundant. Safe to delete.

---

## 5. FRONTEND PAGES & COMPONENTS

### Pages (`src/pages/`)

---

**Explore.tsx** — Destination discovery page

Route: `/` (or `/explore`)  
RPCs called: `explore_destinations_multi_sport_v1` (via `useExploreDestinations` hook)  
Filters: month (default current), sport (default kitesurfing), region, skill level  
Components: KVNav, ExploreBanner, ExploreFilters, ExploreResults  
Known bugs: none active

---

**DestinationDetail.tsx** — Full destination page

Route: `/destinations/:slug`  
RPCs called:
- `get_destination_detail_by_slug_v1(slug)` — all destination data
- `get_schools_by_destination_v1(slug)` — school cards in Section 02
- `get_fitness_by_destination_v1(slug)` — fitness section (via useFitnessFacilities hook)

Two tabs: Destination Info + Trip Plan  
5 numbered sections: 01 Conditions, 02 On The Water, 03 Getting There, 04 Where to Stay, 05 Daily Life  
Components: TripPlannerModal (launches trip plan flow)  

Known bugs (from CLAUDE.md):
- Schools section still rendering old card style (OnTheWaterSection)
- Visa NDJSON chunk not rendering in GettingThereSection
- Scooter Hire card has hardcoded `~$12/day` price
- FROM AIRPORT card spans full width when only card in row

Helper components defined inline in DestinationDetail.tsx:
- `LightSection` — numbered section wrapper
- `DataCard` — white info card with label/value/sub
- `RiskBadge` — coloured pill for Low/Moderate/High/Rare/Seasonal

---

**Admin.tsx** — Admin review dashboard

Route: `/admin` (implied)  
Two tabs: Content Queue + Social Intel  
Components: AdminReviewQueue, SocialIntelReviewQueue  
No RPCs called directly — components handle their own data fetching.

---

**TripPlan.tsx** — Trip plan output page

Route: `/trip-plan` or similar  
Used after TripPlannerModal submits — renders TripPlanOutput component.  
No direct RPC calls — data comes from generate-trip-plan Edge Function stream.

---

**JarvisDashboard.tsx** — DENNIS ops admin dashboard

Route: `/admin/jarvis`  
Auth: PIN `2601`, stored in `localStorage` as `kv_jarvis_auth` (not Supabase auth — internal tool only)  
Display name: **DENNIS** (renamed from JARVIS in commit `197f2bb`)  
Zero Supabase queries — all data is hardcoded placeholder content.  
8 tabs: Overview, Agents, API Costs, Revenue, Sales, Marketing, Queue, Pro Preview  
Cost log (API Costs tab) persists in `localStorage` as `kv_dennis_costs`. Budget tracker turns red at 80% ($40).  
Components: 11 files in `src/components/jarvis/` (see below).

---

**Index.tsx** — Landing page / redirect

**NotFound.tsx** — 404 page

---

### Key Components (`src/components/`)

---

**AdminReviewQueue.tsx**

Purpose: Reviews Facebook, email, content, social approval queue.  
Data: Queries `approval_queue` table directly (not via RPC). Filters by queue_type.  
Actions: Approve, reject, edit draft text, mark as posted.

---

**SocialIntelReviewQueue.tsx**

Purpose: Reviews pending Google Places venues and Seabreeze threads.  
RPCs called:
- `get_pending_social_intel_v1` — lists pending staging rows
- `approve_social_intel_row_v2(id, excluded_indices, custom_venues)` — approve with optional venue exclusions + manual additions
- `reject_social_intel_row` — reject
- `flag_social_intel_row` — flag for later

Detail view: Shows venue cards with checkboxes (checked = keep, unchecked = exclude). Inline add-venue form. Approve button shows `Approve (8 of 10)` when venues excluded.

---

**trip-planner/TripPlannerModal.tsx**

Purpose: 5-step modal collecting user trip preferences.  
Inputs collected: nationality (step 1), skill level (step 2), achievement goals (step 3), gear situation / who's travelling / accommodation type (step 4), top priority / trip dates/length (step 5).  
On submit: calls `generate-trip-plan` Edge Function. Renders TripPlanOutput.

---

**trip-planner/TripPlanOutput.tsx**

Purpose: Renders streaming trip plan sections.  
Section mapping (by NDJSON `id` field):

| id | Component |
|----|-----------|
| `overview` | OverviewSection |
| `on_the_water` | OnTheWaterSection |
| `flights` + `visa` | GettingThereSection |
| `accommodation` | WhereToStaySection |
| `day_structure` + `day_structure_expanded` | DayByDaySection |
| `budget` | BudgetSection |
| `pack` | PackingSection (⚠️ NOT `packing`) |
| `reality_check` | RealityCheckSection (falls back to `destination.reality_check` DB field) |
| `getting_around` | GettingAroundSection |

---

**trip-planner/TripPlanInteractive.tsx** — Trip plan interaction layer (expand/collapse sections)

**trip-planner/NationalityBanner.tsx** — Nationality-based personalization banner (Call 0 output)

**explore/DestinationCard.tsx** — Card component for Explore grid  
**explore/DestinationGrid.tsx** — Grid layout for destination cards  
**explore/ExploreFilters.tsx** — Filter bar (month, sport, region, skill level)  
**explore/ExploreBanner.tsx** — Dark hero banner on Explore page  
**explore/ExploreResults.tsx** — Results count + grid wrapper  

**destination/FitnessSection.tsx** — Fitness section for DestinationDetail (calls get_fitness_by_destination_v1)  

**jarvis/** — DENNIS ops dashboard components (all in `src/components/jarvis/`):

| File | Purpose |
|------|---------|
| `PinEntry.tsx` | PIN lock screen (PIN: 2601, shake animation on wrong entry) |
| `JarvisHeader.tsx` | DENNIS mono header + live clock + 4 health indicator pills |
| `TabBar.tsx` | 8-tab horizontal scrollable nav; Pro Preview has accent left-border |
| `OverviewTab.tsx` | 4 KPI cards + live countdown to July 1 2026 + activity log |
| `AgentsTab.tsx` | 4 PENDING agent cards + runs table + status legend |
| `ApiCostsTab.tsx` | Claude/Supabase/Google Places/Perplexity cost metrics + manual cost log (localStorage) + budget tracker |
| `RevenueTab.tsx` | 4 affiliate stream cards + total MRR + chart placeholder |
| `SalesTab.tsx` | Affiliate pipeline + B2B log + school funnel bars + outreach log |
| `MarketingTab.tsx` | FB groups + content table + community form + SEO + calendar |
| `QueueTab.tsx` | Community submissions + school responses side by side |
| `ProPreviewTab.tsx` | 4 blurred/overlaid sections + $49/yr CTA card |

All data is hardcoded placeholder. No Supabase queries from any jarvis component.

---

**KVNav.tsx / KVFooter.tsx** — Site navigation and footer  
**NavLink.tsx / SiteFooter.tsx** — Additional nav components  

**ui/** — shadcn/ui library components (accordion, button, card, dialog, etc.) — do not modify unless explicitly needed.

---

## 6. DATA INGESTION STATE

Data sourced from live DB query (2026-05-30T07:45:30).

### Normalized Tables (core destination data)
- **101 rows each** across all 10 normalized tables
- 25 `is_active = true` (live on Explore page)
- 76 `is_active = false` (seeded/staged, not live)
- No duplicates — `COUNT(DISTINCT destination_id) = 101` confirmed

### social_intel_staging
- **352 rows** (queried 2026-05-30T07:45:30)
- 0 pending rows remaining (pending_social_intel = 0)
- Pre-dedup: 453 rows (doubled by ingesters running 2-3×)
- By source: 413 google_places + 39 seabreeze + 1 reddit

### google_places_raw
- **2,388 rows** confirmed (queried 2026-05-30T07:45:30)
- Data exists for all 11 categories across active destinations
- Admin review queue: SocialIntelReviewQueue.tsx

### open_meteo_raw
- **300 rows confirmed** — 12 rows per destination × 25 destinations
- Fetched via `fetch-open-meteo` Edge Function using Open-Meteo archive API (free, daily data aggregated to monthly)
- All 25 active destinations covered after coordinates were populated for 15 missing spots

### noaa_copernicus_raw
- **300 rows confirmed** — 12 rows per destination × 25 destinations
- Fetched via `ingest-noaa-copernicus-data` Edge Function using Open-Meteo Marine API (ERA5 reanalysis)
- Wave: hs_mean/min/max/std, tp_mean, compass direction. SST: sst_mean/min/max
- Status in staging: `approved` (auto-approved, confidence 9.0) — does not need admin queue review

### seabreeze_raw
- **26 rows** (queried 2026-05-30T07:45:30)
- Staging rows in social_intel_staging sourced from seabreeze

### approved_social_intel
- **300 rows** confirmed (queried 2026-05-30T07:45:30)

### research_staging
- **810 rows** (queried 2026-05-30T07:45:30)
- Populated by Perplexity collector scripts (v1–v4). Contains raw JSON responses for P7–P15.
- P7–P13: collected by v3 collector. P14–P15: collected by v4 collector (active, in `scripts/data-collection/`)
- All 25 active destinations × prompts P7–P15 parsed and written to production tables (completed 2026-05-25)

### destination_areas
- **177 rows** confirmed (queried 2026-05-30T07:45:30)
- Populated by `kitevurse_perplexity_parser.py` parse_p14 (2026-05-25)
- `destination_editorial.areas_recommendation` also populated for all 25

### kite_hubs
- **139 rows** confirmed (queried 2026-05-30T07:45:30)
- Populated by `kitevurse_perplexity_parser.py` parse_p15 (2026-05-25)
- `destination_editorial.scene_summary` + `proximity_reality` populated for all 25

### destination_wind_seasons
- **48 rows** confirmed (2026-05-30)
- Populated by `kitevurse_perplexity_parser.py` parse_p9 (v5 collector run — migration 037)
- One row per named wind season per destination (destinations with dual monsoon seasons get 2 rows)
- Fields: season_name, months[], avg_wind_knots_low/high, wind_direction, shore_angle, rideable_days_per_week, kite_sizes_70kg, honest_take

### schools (single source of truth — deduped)
- **183 rows** (2026-05-30 — deduped from 197 via migration 043: 14 near-duplicates removed)
- Served by `get_schools_by_destination_v1`
- `kv_pick` = false on all rows — must be set manually by Dan (max 1 per destination)
- **data_source breakdown:** 101 `perplexity` (original), 82 `perplexity_p16` (genuinely new from kite_schools merge), enriched rows set to `perplexity_enriched`
- Trigger `enforce_perplexity_no_insert` (migration 043) blocks any INSERT with data_source='perplexity'
- P16 parser now UPDATE-only — enriches certifications, lesson_price_usd_from, accommodation_available where null

### kite_schools
- **DROPPED** — migration 044 (2026-05-28). Data was merged into `schools` in migration 042, deduped in 043, table dropped in 044.

### kite_accommodations
- **141 rows** confirmed (2026-05-30)
- Populated by `kitevurse_perplexity_parser.py` parse_p17 (P17 Kite Accommodations prompt — migration 039)
- Kite-community accommodation intel: which properties kiters use, proximity to launch, gear storage, kite packages.

### kite_gear_services
- **161 rows** confirmed (2026-05-30)
- Populated by `kitevurse_perplexity_parser.py` parse_p18 (P18 Gear & Repair prompt — migration 040)
- One row per operator or repair shop. service_type = 'rental' or 'repair'. Includes destination-level stranded_risk.

### destination_daily_living
- **25 rows** confirmed (2026-05-30)
- Populated by `kitevurse_perplexity_parser.py` parse_p19 (P19 Daily Living prompt — migration 041)
- One row per destination (UPSERT on destination_id). Covers payment, transport, language, SIM, water/food, dress, tipping. supermarkets always null.

### kite_spots
- **117 rows** (queried 2026-05-30T07:45:30)

### visa_requirements
- **251 rows** (queried 2026-05-30T07:45:30)

### sim_card_options
- **0 rows** — not yet collected

### rome2rio_raw
- **0 rows** — not yet collected

### medical_emergency
- **0 rows** — not yet collected

### wind_community_reports / accommodation_reviews / destination_condition_updates
- **0 rows each** — not yet populated

### airports
- 1,176 large airports + 5 destination-only airports
- All 25 active destinations wired with airport_iata FK

### approval_queue
- **6 rows** total; **4 pending** (queried 2026-05-30T07:45:30)

### facebook_engagement_queue
- **7 rows** (queried 2026-05-30T07:45:30)

---

## 7. ENVIRONMENT

### Active Supabase Project: `xthaiitjoccrrqivjlor`

| Env Var | Purpose | Status |
|---------|---------|--------|
| `VITE_KITEVERSE_SUPABASE_URL` | Active Supabase project URL | ✅ Use this |
| `VITE_KITEVERSE_SUPABASE_ANON_KEY` | Active anon key (publishable) | ✅ Use this |

### Legacy Supabase Project: `ouqqfpqsbaijbinpyiio` — DO NOT USE FOR NEW WORK

| Env Var | Purpose | Status |
|---------|---------|--------|
| `VITE_SUPABASE_URL` | Old project URL | ❌ Legacy |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Old anon key | ❌ Legacy |
| `VITE_SUPABASE_PROJECT_ID` | Old project ID | ❌ Legacy |

### Edge Function Secrets (set in Supabase Vault / Edge Function env)

| Secret | Used By |
|--------|---------|
| `SERVICE_ROLE_KEY` | All write Edge Functions (NOT `SUPABASE_SERVICE_ROLE_KEY` — blocked prefix) |
| `SUPABASE_URL` | Edge Functions needing the project URL |
| `GOOGLE_PLACES_API_KEY` | search-google-places |
| `FACEBOOK_USER_TOKEN` | post-facebook-comments |
| `ANTHROPIC_API_KEY` | generate-trip-plan, extract-social-intel, classify-facebook-posts |

### Design Tokens

```
Page bg:        #f6f4ef
Dark bg:        #0a0e14
Accent:         #2DABE5
Primary text:   #0a0e14
Muted text:     #6b7a8d
Inverted text:  #e8eaed
Green stat:     #7ed3a2
Border light:   rgba(10,14,20,0.09)
Border dark:    rgba(255,255,255,0.08)
```

Fonts: `'Inter Tight', system-ui, sans-serif` (display/headings). `'JetBrains Mono', 'Courier New', monospace` (labels/numbers/eyebrows).

### Dev Server

`npm run dev` → `localhost:8080`

---

## APPENDIX: Critical Rules Summary

1. **SECURITY DEFINER** — every RPC that reads from RLS-protected normalized tables must declare `SECURITY DEFINER` and `SET search_path = public`. Without it: returns all nulls/zeros with no error.

2. **pgcrypto** — any RPC using `digest()` for SHA-256 must include `extensions` in search_path: `SET search_path = public, extensions`.

3. **Never generate factual data via Claude API** — airport codes, wind data, hazards, etc. must come from the database. Claude API only generates narrative content.

4. **Anon key only in frontend** — `VITE_KITEVERSE_SUPABASE_ANON_KEY`. Never expose service_role key.

5. **Slug-based routing only** — no ID routes in the frontend.

6. **RPC-first** — never query tables directly from the frontend.

7. **ingest_reddit_posts signature** — final signature is `(input_slug TEXT, posts JSONB)`. The old single-param pg_net version was superseded in migration 019.

8. **approve_social_intel_row_v2 signature** — current is 3 params: `(p_id UUID, p_excluded_indices INT[] DEFAULT NULL, p_custom_venues JSONB DEFAULT NULL)`. The 2-param version was dropped in migration 032.

9. **Section ID** — `pack` not `packing`. This was a bug that was fixed.

10. **destination_media JOIN** — joins on `d.destination_id` (text) not `d.id` (uuid). Critical for hero image queries.

---

## 8. DATA COLLECTION SCRIPTS

All data collection scripts live in `scripts/data-collection/` within this repo (migrated from `C:\Users\danwi\KiteVurse_DataCollection\` on 2026-05-26).

### Launchers (entry points)

| Script | Purpose |
|--------|---------|
| `run_ingester.ps1` | Loads service role key from Windows Credential Manager → runs `kitevurse_places_ingester.py` |
| `run_parser.ps1` | Loads service role key from Windows Credential Manager → runs `kitevurse_perplexity_parser.py` |
| `run_collector.ps1` | Loads Perplexity API key from Windows Credential Manager (`KiteVurse:PERPLEXITY_API_KEY`) + service role key → runs `kitevurse_perplexity_collector_v5.py`. Pass all CLI args through. |

Key mechanism: both scripts read `Supabase CLI:supabase` from Windows Credential Manager, exchange it for the project service role key via Supabase Management API, set `$env:SUPABASE_SERVICE_ROLE_KEY_KV`, then `Set-Location $PSScriptRoot` before calling Python. Works from any working directory.

### Python Scripts

| Script | Status | Purpose |
|--------|--------|---------|
| `kitevurse_places_ingester.py` | ✅ Active | Moves approved Google Places rows from `google_places_raw` → production venue tables. Only processes combos in `approved_social_intel`. |
| `kitevurse_perplexity_parser.py` | ✅ Active | Reads `research_staging` → writes P7–P15 to production tables. Resume-safe via `perplexity_parser_progress.json`. |
| `kitevurse_perplexity_collector_v5.py` | ✅ Active | Calls Perplexity API, writes to `research_staging`. Credentials from env vars (never hardcoded). Single PROMPTS dict with `enabled` boolean. P11, P14, P15 active by default. Resume-safe via `progress_v5.json`. |
| `kitevurse_perplexity_collector_v4.py` | Retired | Replaced by v5. Had hardcoded credentials, bare relative path for OUTPUT_FOLDER, split PROMPTS/_DISABLED_PROMPTS dicts, and discarded staging insert return value. |
| `kitevurse_perplexity_collector_v3.py` | Retired | P7–P13, v3 (improved JSON extraction) |
| `kitevurse_perplexity_collector_v2.py` | Retired | P7–P13, v2 |
| `kitevurse_perplexity_collector.py` | Retired | P1–P6, v1 |
| `_fix_encoding.py` | One-time util | Replaced emoji with ASCII in parser.py — already applied |

### Parser Prompt Map (current — P7–P18 all complete)

| Prompt | Name | Target Table(s) |
|--------|------|-----------------|
| P7 | Culture & Safety | `culture_safety` (UPSERT on destination_id) |
| P8 | Visa Requirements | `visa_requirements` (DELETE+INSERT per passport) |
| P9 | Wind & Water | `destination_conditions` (UPDATE) + `destination_wind_seasons` (DELETE+INSERT per season) |
| P10 | Destination Profile | `destination_editorial` (UPDATE) |
| P11 | Spot Geometry | `kite_spots` (DELETE+INSERT per spot) |
| P12 | Regulations | `spot_regulations` (DELETE+INSERT, UNIQUE on destination_id) |
| P13 | Logistics | `destination_kiter_logistics` (DELETE+INSERT, UNIQUE on destination_id) |
| P14 | Neighborhood & Area Intel | `destination_areas` (DELETE+INSERT) + `destination_editorial.areas_recommendation` (UPDATE) |
| P15 | Kite Hubs & Scene | `kite_hubs` (DELETE+INSERT) + `destination_editorial.scene_summary` + `proximity_reality` (UPDATE) |
| P16 | Kite Schools | `schools` — UPDATE only (enriches certifications, lesson_price_usd_from, accommodation_available where null; never inserts; trigger enforced in migration 043) |
| P17 | Kite Accommodations | `kite_accommodations` (DELETE+INSERT, manually_verified=false rows only) |
| P18 | Gear & Repair | `kite_gear_services` (DELETE+INSERT, manually_verified=false rows only) |
| P19 | Daily Living | `destination_daily_living` (UPSERT on destination_id, one row per destination) |

Run from kite-explorer root: `.\scripts\data-collection\run_parser.ps1 --prompt 19 --dest dakhla-morocco`

### Collector Architecture (v5)
All prompts live in a single PROMPTS dict with an `enabled` boolean. No _DISABLED_PROMPTS dict. Disabled prompts have `enabled: False` and are skipped automatically by the run loop. Every prompt entry declares `max_tokens` and `temperature` explicitly — no function defaults. All file paths are resolved relative to `Path(__file__).parent`. Credentials are loaded from env vars (`PERPLEXITY_API_KEY`, `SUPABASE_SERVICE_ROLE_KEY_KV`, `SUPABASE_URL_KV`) with a hard `sys.exit(1)` at startup if any are missing. Staging insert return value is checked — a failed insert marks the call key in `failed_calls` so `--retry-failed` picks it up on the next run. A token proximity warning fires when response length exceeds 90% of `max_tokens × 3.5` chars. Coding standards for all data-collection scripts are documented in `scripts/data-collection/STANDARDS.md`.

**v5 prompt inventory:** P7 (Culture & Safety), P8 (Visa), P9 (Wind & Water — now also writes destination_wind_seasons), P10 (Destination Profile), P11 (Spot Geometry), P12 (Regulations — fixed: per-spot format), P13 (Logistics — fixed: confirmed names), P14 (Neighborhood & Area Intel), P15 (Kite Hubs & Scene), P16 (Kite Schools → kite_schools), P17 (Kite Accommodations → kite_accommodations), P18 (Gear & Repair → kite_gear_services), **P19 (Daily Living → destination_daily_living)**. P7/P9/P12/P13 were also fixed in v5 for dual numbers, zero tolerance, confirmed names, and per-spot regulations respectively.

### Destination Object Builder

`build_destination_object.py` — queries all 40 tables for a destination slug and writes a single nested JSON to `scripts/data-collection/outputs/<slug>_destination_object.json`. Includes `destination_daily_living` (P19) as of 2026-05-28.

Run: `.\scripts\data-collection\run_destination_object.ps1 --slug diani-beach-kenya`

**When a new table is added to the DB, update this script**: use `one()` for single-row tables (UNIQUE on destination_id), `multi()` for multi-row tables, then add the variable to the output dict.

---

## 9. MAINTENANCE SCRIPTS

###