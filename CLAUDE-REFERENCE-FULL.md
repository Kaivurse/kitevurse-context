# CLAUDE-REFERENCE-FULL.md — KiteVurse Complete Technical Reference

Generated: 2026-05-25 | Last updated: 2026-06-04 (Phase 2 frontend redesign underway — design system, destination page partial impl, live audit bugs, new tables 037-044, branch update)
Source: schema_tables.json + all migration files + Edge Function source + frontend source + live browser audit
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
- **Every session needs tangible output** — something Dan can click, test, or see.
- **Always end with a commit** unless Dan explicitly says not to.
- **Never ask questions already answered** in this file or earlier in the conversation.
- **When Dan is in flow, keep going.** Do not interrupt with check-ins.
- **Never give Dan a list of things to remember.** Bake it into files, scripts, or automated processes.
- **Verify first, execute second, test third** — always read this file before writing any SQL or referencing table/column names.

---

## DAN'S WORKING PROFILE

### How Dan works
- Best hours: 7am–12pm. Schedule hard decisions and complex sessions here.
- Carries context between sessions mentally — arrives knowing what to do next.
- Needs tangible output every session. Something visible to touch, click, or test.
- Runs on dopamine from visible progress. A working page beats 500 rows in a database every time.

### ADHD compensations (non-negotiable)
- Never give Dan a list of things to remember. Bake everything into files, scripts, and automated processes.
- Never question work Dan says is complete without a specific technical reason.
- Search conversation history before asking any question.
- One task per session. No sprawl.
- When Dan is in flow, do not interrupt. Keep going.

### Strengths to leverage
- Exceptional data analysis skills — spots relationships between data sets others miss.
- Systems integration mindset — thinks in connections, not features.
- Creative problem-solver under pressure.

---

## PHASE PLAN & ROADMAP

### What KiteVurse Is

Trip advice from the most well-traveled kiter you know — who tells you the truth about a spot.

Not a booking site. Not a directory. The honest, AI-powered planning tool the kitesurfing community doesn't have yet.

Core value proposition: 343 real data points per destination, across 21 tables. Structured data, not AI hallucination.

---

### Business Model (Locked)

| Revenue Stream | Mechanic | Timeline |
|---|---|---|
| Booking.com affiliate | 4–6% on accommodation referrals | Post-launch (needs traffic first — apply after 2-3 months live) |
| Skyscanner CPC | Cost-per-click on flight searches | Post-launch |
| SafetyWing referral | $15–25 per signup | Post-launch |
| **KiteVurse Pro** | **$49/year** | **Month 9** |

**Affiliate status (June 2026):** No affiliates applied yet. No social following yet. Must build audience first via share mechanic — then apply. Do not apply early: rejected applications burn goodwill.

---

### The Five Phases

| Phase | Focus | Status |
|---|---|---|
| **1 — Data Complete** | Collection, review queue, ingestion to production | ✅ Complete |
| **2 — Frontend Redesign** | Full UI rebuild — dark editorial design system, all pages rebuilt | 🟡 In Progress (destination page partial) |
| **3 — Trip Engine Upgrade** | 3-call Edge Function, new inputs, streaming output | 🔴 Not started |
| **4 — Soft Launch + Community** | Vercel deploy, Facebook engagement, real users | 🔴 Not started |
| **5 — Agents + Pro + Scale** | Agentic system, KiteVurse Pro, expand to 76 destinations | 🔴 Not started |

---

### PHASE 2 — Frontend Redesign 🟡

#### What Phase 2 Is

Complete replacement of every page users see. The backend (Supabase DB, Edge Functions, RPCs, data collection scripts) is **untouched**. Phase 2 is pure frontend.

**Active branch:** `feat/phase2-redesign`
**Dev server:** `localhost:8080` (`npm run dev`)
**Reference prototype:** `kitevurse-prototype.html` (Dan has locally — definitive visual reference)

#### Architecture decisions (locked for Phase 2)

- **DB vs API split is architectural law.** DB stores destination facts (wind, water, hazards, budgets, airport codes). Claude API generates personalized dynamic content (visa narrative, lifestyle narrative, verdict). Claude API must never generate factual DB fields.
- **RPC-first.** Normalized tables (destination_conditions etc.) require SECURITY DEFINER RPCs. Supplementary tables (destination_wind_seasons, destination_areas, kite_hubs etc.) have public read RLS and query directly.
- **TanStack Query v5 for all data.** No raw useEffect + useState for async data.
- **vaul for all bottom sheets.** No custom sheet components.
- **sonner for all toasts.** No Radix Toast or custom toasts.
- **Schools use get_schools_by_destination_v1 RPC only.** Never query schools table directly.
- **kite_schools table is DROPPED** (migration 044). `schools` is the single source of truth (183 rows).
- **Visa data from visa_requirements table only.** Never from AI, never from destination_daily_living.
- **Save/share conversion event:** "Save this plan" is the primary conversion. Save flow email entry = email capture + lead + conversion, in one action. Magic-link auth (Supabase signInWithOtp) — profile page at `/profile`.

#### Phase 2 implementation status (as of June 4, 2026)

| Page / Feature | Status | Notes |
|---|---|---|
| Design system (Tailwind config, tokens, fonts) | 🟡 Partially implemented | Tokens in tailwind.config.ts; mobile stats band fix committed 2026-06-02 |
| Explorer page | 🔴 Not yet rebuilt | Old Explore.tsx still in place |
| Destination page — hero | 🟡 Partial | Single-column layout; mobile stats band horizontal scroll strip; mobile Plan My Trip CTA added above Wind Season |
| Destination page — wind season | 🟡 Working | Tabs work, rideable days text-only (no dots yet), wind bar present |
| Destination page — My Base | 🟡 Working | 3 tabs render (4th missing), single-column layout (two-column grid bug fixed 2026-06-02), SOCIAL/EVENING open by default |
| Destination page — mobile | 🟡 Partial | Device frame renders, bottom tab bar renders, mobile CTA inserted; no hero image yet |
| Trip plan intake | 🔴 Not yet built | Old TripPlannerModal still in place |
| Trip plan output | 🔴 Not yet built | Old TripPlanOutput still in place |
| Authentication (magic link) | 🔴 Not built | Migration 045 saved_trips not yet run |
| Profile page | 🔴 Not built | |
| Shared view | 🔴 Not built | |

#### Known destination page bugs (from live review June 2, 2026)

Full requirements: see `DESTINATION-PAGE-REQUIREMENTS.md` in the repo root (to be added by Dan after this session).

**Critical bugs:**
1. ~~Stats band class `hidden lg:block` — Stats only show at 1280px+. Mobile and tablet show no stats.~~ **Fixed 2026-06-02** (horizontal scroll strip on mobile/tablet)
2. ~~Plan My Trip CTA only at offsetTop 4205 in a 4389px page — no hero CTA on mobile.~~ **Fixed 2026-06-02** (mobile CTA inserted above Wind Season section)
3. "The Catch" warning completely absent from DOM (`hasCatchText: false`).
4. ~~Wind Season + My Base in side-by-side two-column grid — when My Base spine grows tall, left column becomes empty void.~~ **Fixed 2026-06-02** (single-column layout)
5. ~~WIND RELIABILITY stat shows wind speed "18-24 KT" — wrong. Should show reliability score /100.~~ **Fixed 2026-06-02** (now shows wind_reliability/100)
6. ~~~400px empty white space below invest footer.~~ **Fixed 2026-06-02** (grid removed)
7. No hero image (placeholder only).

**Priority fix order:** Bug 3 (The Catch) next. Then hero H1 + CTA. Then progressive disclosure. Then mobile breakpoints. Then animations.

#### New tables in Phase 2 (supplementary, public read RLS)

These tables are queried directly from frontend (no RPC needed):

**destination_wind_seasons** (migration 037) — per-season wind data replacing the old single-row conditions approach

| Column | Type |
|---|---|
| id | uuid PK |
| destination_id | uuid FK → destinations |
| season_name | text (e.g. "KUZI", "KASKAZI") |
| months | smallint[] (e.g. [6,7,8,9]) |
| avg_wind_knots_low | smallint |
| avg_wind_knots_high | smallint |
| wind_direction | text |
| wind_angle_degrees | smallint |
| shore_angle | text (e.g. "side-shore", "cross-onshore") |
| rideable_days_per_week_low | smallint |
| rideable_days_per_week_high | smallint |
| kite_sizes_70kg | text[] (e.g. ["9m","11m","13m"]) |
| chop | text |
| honest_take | text |
| is_active | boolean |
| created_at / updated_at | timestamptz |

RLS: Public read. Query: `supabase.from('destination_wind_seasons').select('*').eq('destination_id', id).eq('is_active', true)`

**kite_accommodations** — accommodation options near kite spots

Key columns: destination_id FK, name, area_name, distance_to_launch_m, price_per_night_usd_low/high, gear_storage boolean, kite_packages boolean, community_favourite boolean, website, is_active.

**kite_gear_services** — gear rental and services

Key columns: destination_id FK, service_type (rental/repair/storage), name, price_per_day_usd, stranded_risk text (LOW/MEDIUM/HIGH), is_active.

**destination_daily_living** — daily logistics and life

Key columns: destination_id FK, transport_primary, transport_price_range, transport_negotiate boolean, ride_app_available boolean, payment_app_name, payment_app_tourist_ok boolean, usd_accepted boolean, tap_water_safe boolean, sim_worth_it boolean, sim_carrier, sim_where_to_buy, english_sufficient boolean, english_notes, beach_dress, offbeach_dress, tip_restaurant_pct, bargaining boolean, is_active.

**saved_trips** (migration 045 — NOT YET APPLIED) — user saved trip plans

```sql
CREATE TABLE saved_trips (
  id                  uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  destination_id      uuid REFERENCES destinations(id),
  destination_slug    text NOT NULL,
  destination_name    text NOT NULL,
  nationality         text,
  arrival_date        date,
  departure_date      date,
  group_type          text,
  skill_level         text,
  weight_bracket      text,
  gear_situation      text,
  budget_level        text,
  verdict             text CHECK (verdict IN ('GO', 'EYES_OPEN', 'WAIT')),
  share_token         text UNIQUE DEFAULT encode(gen_random_bytes(12), 'base64url'),
  created_at          timestamptz DEFAULT now()
);
```

Migration 045 must be run before auth/save feature can be built.

#### Phase 2 — Edge Function update (NOT YET DONE)

The current `generate-trip-plan` Edge Function makes 10-11 Claude API calls. Phase 3 will rebuild it to 3 targeted calls:

| Call | ID | Tokens | Content |
|---|---|---|---|
| 1 | `verdict` | 400 | GO/EYES_OPEN/WAIT headline + 3 reasons |
| 2 | `on_the_water_narrative` | 350 | 1–2 sentence water conditions framing |
| 3 | `lifestyle_narrative` | 400 | Group-conditional lifestyle narrative |

All 3 fire in parallel. Returns NDJSON. Frontend renders DB data immediately; AI narrative hydrates when it lands. **Do not rebuild the Edge Function until the output page is designed and the intake reorder is implemented.**

---

## DESIGN SYSTEM (Phase 2)

This is the locked design system for all Phase 2 pages. Claude Code must read this section before touching any UI component.

### Colors (tailwind.config.ts)

```typescript
colors: {
  page:          '#f6f4ef',    // page background (warm off-white)
  dark:          '#0a0e14',    // dark backgrounds, nav, verdict zone
  accent:        '#2DABE5',    // primary blue
  'accent-deep': '#006194',    // deeper blue — all CTAs, active states
  ink:           '#0a0e14',    // primary text on light bg
  muted:         '#6b7a8d',    // secondary text
  label:         '#54606c',    // label text
  inverted:      '#e8eaed',    // text on dark backgrounds
  green:         '#7ed3a2',    // GO verdict, positive signals
  amber:         '#d99a3a',    // EYES_OPEN verdict, warnings
  'bd-light':    'rgba(10,14,20,0.09)',    // borders on light bg
  'bd-dark':     'rgba(255,255,255,0.08)', // borders on dark bg
}
```

### Typography

```html
<!-- index.html -->
<link href="https://fonts.googleapis.com/css2?family=Inter+Tight:ital,wght@0,400;0,500;0,600;0,700;0,800;1,400&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

```typescript
// tailwind.config.ts
fontFamily: {
  display: ["'Inter Tight'", 'system-ui', 'sans-serif'],
  mono:    ["'JetBrains Mono'", "'Courier New'", 'monospace'],
}
```

**`font-display` (Inter Tight):** All headings, body text, buttons, option labels.
**`font-mono` (JetBrains Mono):** ALL labels, eyebrows, data values, chips, stats, nav labels, section numbers. Anywhere a number, tag, short code, or monospace label appears.

### Breakpoints

| Name | Width | Tailwind | Layout |
|---|---|---|---|
| Mobile | < 768px | (base) | Device frame, bottom tab bar, single column |
| Tablet | 768–1279px | `md:` | No device frame, wider padding, top nav |
| Desktop | ≥ 1280px | `lg:` | Max-width container, top nav |

**Primary test viewports:** 375px (iPhone SE), 390px (iPhone 14 Pro), 768px (iPad), 1280px desktop.

**Critical:** Mobile is the primary use case. Every component must be tested at 375px before committing. Never use `hidden lg:block` for content that kiters need to make decisions.

### Device Frame (mobile only)

```typescript
// src/components/ui/DeviceFrame.tsx
// Mobile: phone frame with rounded corners, shadow, grain overlay
// Tablet+: full width, no frame
// 375px width, height: min(92vh, 812px)
// border-radius: 30px (rounded-device in Tailwind)
// box-shadow: device shadow token
// Grain overlay: SVG fractalNoise, opacity 0.5, mix-blend-multiply
```

The DeviceFrame renders children TWICE — once in mobile frame (`md:hidden`) and once for tablet+ (`hidden md:block`). Both instances are mounted in the DOM. This is intentional. data-testid="device-frame" is present.

### VerdictPill component

```
GO:         bg rgba(126,211,162,0.18), text #7ed3a2
EYES_OPEN:  bg rgba(217,154,58,0.18), text #d99a3a
WAIT:       bg rgba(248,113,113,0.18), text #ef4444
Format: coloured dot (w-1.5 h-1.5) + verdict text in font-mono uppercase tracking-[0.12em]
```

### Verdict calculation (client-side, not AI)

```typescript
function calculateVerdict(season: DestinationWindSeason, conditions: DestinationConditions, month: number): 'GO' | 'EYES_OPEN' | 'WAIT' {
  if (conditions.off_season_months?.includes(month)) return 'WAIT';
  if (season.rideable_days_per_week_low >= 4) return 'GO';
  if (season.rideable_days_per_week_low >= 2) return 'EYES_OPEN';
  return 'WAIT';
}
```

### Kite quiver weight scaling

```typescript
const WEIGHT_OFFSET = { 'under-65': -1, '65-80': 0, '80-95': +1, '95-plus': +2 };
// Apply offset to each size in kite_sizes_70kg array
// Always round to whole metre. Never show decimals.
// e.g. at 80-95kg: ["9m","11m","13m"] → ["10m","12m","14m"]
```

---

## ROUTING ARCHITECTURE (Phase 2)

React Router v6. All routes in `src/App.tsx`.

```typescript
const router = createBrowserRouter([
  { path: '/',                        element: <ExplorePage /> },
  { path: '/destinations/:slug',      element: <DestinationPage /> },
  { path: '/trip/:slug',              element: <TripPlanPage /> },
  { path: '/profile',                 element: <ProfilePage /> },
  { path: '/trip/shared/:token',      element: <SharedTripView /> },
  { path: '/admin',                   element: <Admin /> },
  { path: '/admin/jarvis',            element: <JarvisDashboard /> },
  { path: '*',                        element: <NotFound /> },
]);
```

Always use `useNavigate()` or `<Link>` — never `window.location.href`.

---

## DATA FETCHING PATTERNS (Phase 2)

### RPC vs direct query

**Normalized tables** (SECURITY DEFINER RPC required — direct queries return all nulls):
destination_conditions, destination_sports, destination_safety, destination_budget, destination_transport, destination_tides, destination_gear, destination_lifestyle, destination_infrastructure, destination_editorial, destination_meta

→ Always use `get_destination_detail_by_slug_v1`.

**Supplementary tables** (public read RLS — query directly):
destination_areas, kite_hubs, destination_wind_seasons, kite_accommodations, kite_gear_services, destination_daily_living, visa_requirements, culture_safety, kite_spots

→ Query directly from Supabase client. No RPC needed.

**Schools:** use `get_schools_by_destination_v1(input_slug)`. Never query schools table directly. Sort order handled by RPC: kv_pick → manually_verified → capability_count → name.

### TanStack Query v5 pattern

```typescript
// Standard hook pattern for all data fetching
export function useDestinationDetail(slug: string) {
  return useQuery({
    queryKey: ['destination', slug],
    queryFn: () => supabase.rpc('get_destination_detail_by_slug_v1', {
      input_slug: slug
    }).throwOnError(),
    staleTime: 1000 * 60 * 10,
    enabled: !!slug,
  });
}

// Direct query for supplementary tables
export function useWindSeasons(destinationId: string) {
  return useQuery({
    queryKey: ['wind-seasons', destinationId],
    queryFn: async () => {
      const { data, error } = await supabase
        .from('destination_wind_seasons')
        .select('*')
        .eq('destination_id', destinationId)
        .eq('is_active', true);
      if (error) throw error;
      return data;
    },
    staleTime: 1000 * 60 * 10,
    enabled: !!destinationId,
  });
}
```

### Season matching (client-side utility)

```typescript
// src/lib/season.ts
export function getSeasonForMonth(seasons: DestinationWindSeason[], month: number): DestinationWindSeason | null {
  if (!seasons.length) return null;
  const direct = seasons.find(s => s.months.includes(month));
  if (direct) return direct;
  // Transition month — closest by month proximity
  return seasons.reduce((closest, s) => {
    const dist = Math.min(...s.months.map(m => Math.abs(m - month)));
    const closestDist = Math.min(...closest.months.map(m => Math.abs(m - month)));
    return dist < closestDist ? s : closest;
  });
}
```

---

## TRIP PLANNER — USER INPUTS

### TripPlanInputs type (intake form)

```typescript
// src/types/trip-plan.ts
export type WeightBracket = 'under-65' | '65-80' | '80-95' | '95-plus';
export type GroupType = 'solo' | 'partner' | 'friends' | 'family';
export type SkillLevel = 'learning' | 'getting-there' | 'comfortable' | 'advanced' | 'foiler';
export type GearSituation = 'own-kit' | 'renting' | 'mix';
export type BudgetLevel = 'tight' | 'comfortable' | 'open';

export interface TripPlanInputs {
  nationality:   string;        // ISO country name
  arrival:       string;        // YYYY-MM-DD
  departure:     string;        // YYYY-MM-DD
  group:         GroupType;
  skill:         SkillLevel;
  weight:        WeightBracket;
  gear:          GearSituation;
  budget:        BudgetLevel;
  arrivalMonth:  number;        // 1-12, derived
  nights:        number;        // derived
}
```

### Intake step order (approved design — to be implemented)

**Step 1 — "You":** Group (Solo/Partner/Friends/Family) + Skill (Learning→Foil). Two taps. Cheapest, most identity-affirming inputs first.

**Step 2 — "Your setup":** Weight + Gear + Budget. Three taps. Weight feeds the quiver payoff.

**Step 3 — "When":** Nationality + Dates. The commitment block, last. Dates naturally precede "Build my plan" because they generate the season match.

Rationale: Progressive commitment. Cheapest/most-fun input leads. Heaviest input (date pickers + country search) goes last where sunk-cost carries people through.

---

## AUTH ARCHITECTURE (Phase 2 — not yet built)

Magic link only. No password. No OAuth.

```typescript
// src/lib/auth.ts
export async function sendMagicLink(email: string) {
  return supabase.auth.signInWithOtp({
    email,
    options: { emailRedirectTo: `${window.location.origin}/profile` },
  });
}
```

Auth state: `AuthContext.tsx` wraps app, provides `session` via `onAuthStateChange`.

Conversion flow: "Save this plan" → enter email → magic link → that email IS the lead + conversion in one. Profile page at `/profile` shows saved trips.

**Migration 045 (saved_trips) must be run before this can be built.** See Phase 2 new tables section above.

---

## VOICE & AUTHENTICITY STANDARD

All Claude API-generated content must pass this test. Write like a real person who kites.

**Prohibited vocabulary:** optimal, ecosystem, presents, establishes, aforementioned, leverages, showcases, facilitated, streamlined.

**If you'd never say it to a kiter at the bar, remove it.**

System prompt for all Claude API calls (generate-trip-plan and any new Edge Functions):
```
Write like a real person who kites. Use contractions. Prefer simple words.
Understate rather than oversell.
Avoid: 'presents,' 'establishes,' 'ecosystem,' 'optimal,' 'showcases,' 'leverages.'
Be honest about limitations. Specific beats vague. Short beats long.
If you'd never say it to a kiter at the bar, remove it.
```

Bad: "This destination presents an exceptional opportunity for progression riders"
Good: "This is a solid spot to improve"

---

## CONDITIONAL RENDERING — CRITICAL RULES

- **Null field = omit that row.** Never render "null" or empty string in the UI.
- **No blank sections.** Every section must show real data or a sparse fallback card.
- **Sparse fallback:** if data incomplete, show a 2-sentence "what we know" card rather than nothing.
- **Schools never in trip plan.** Schools section in trip plan output = one CTA line only: `"{N} schools at this destination — Compare all ↗"` → `/destinations/:slug#schools`. Never render school cards in trip plan output.
- **Day 1 no wind always REST.** Hardcoded UX rule regardless of destination data.
- **kv_pick sort via RPC.** Do not re-sort schools after fetching — the RPC sort is authoritative.
- **Airport codes from airports table via destinations.airport_iata JOIN.** Never hardcode. Claude API never generates airport codes.

---

## 1. DATABASE TABLES

### 1A. Core Normalized Tables (10 tables, 101 rows each)

All 10 tables created in `migrations/001_create_normalized_tables.sql`. Every table has RLS enabled. All read by main RPCs via SECURITY DEFINER. Row count: **101 rows per table** (25 active + 76 inactive destinations).

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
| parsed_data | jsonb |
| parsed_at | timestamptz |
| needs_refresh_by | date |
| open_meteo_data | jsonb |
| data_source | text |
| created_at / updated_at | timestamptz |

---

**destination_sports** — Discipline scores and skill level data

Key columns: destination_id FK, disciplines_supported text[], skill_levels_supported text[], beginner_friendliness_score, kitesurf_score, wingfoil_score, kitefoil_score, freestyle_score, freeride_score, wave_riding_score, downwind_score.

---

**destination_safety** — Hazards and risk scores

Key columns: destination_id FK, jellyfish_risk_score, algae_risk_score, water_hazard_score, theft_risk_score, scam_risk_score, medical_access_score, crowding_score.

---

**destination_budget** — Cost and accommodation data

Key columns: destination_id FK, budget_level, average_daily_budget_usd, average_monthly_budget_usd, average_meal_cost_usd, accommodation_cost_score, scooter_rental_monthly_usd, long_term_rental_availability.

---

**destination_transport** — Getting there and getting around

Key columns: destination_id FK, nearest_airport, airport_distance_minutes, transportation_required, scooter_friendly, car_recommended, walkable.

---

**destination_tides** — Tide-specific data

Key columns: destination_id FK, tide_dependent boolean, best_tide, tide_notes, tidal_range, tidal_currents, booties_recommended.

---

**destination_gear** — Wetsuit and equipment requirements

Key columns: destination_id FK, wetsuit_needed, recommended_wetsuit_thickness.

---

**destination_lifestyle** — Digital nomad and community data

Key columns: destination_id FK, wifi_quality_score, coworking_available, beachfront_availability, digital_nomad_friendliness, facebook_groups_available, whatsapp_groups_available, expat_community_strength, local_scene_size.

---

**destination_infrastructure** — Facilities, gear, rescue

Key columns: destination_id FK, kite_schools (jsonb — LEGACY: use schools table instead), launch_zones jsonb, gear_rental jsonb, gear_storage jsonb, repair_services jsonb, rescue_services jsonb, storage_available, beach_assistance_available, wingfoil_schools_available.

⚠️ `destination_infrastructure.kite_schools` is a legacy JSONB field. Use the `schools` table for all school data.

---

**destination_editorial** — Narrative and prose content

Key columns: destination_id FK, vibe_summary, best_for_summary, not_for_summary, reality_check, practical_notes, daily_life_summary, no_wind_activities, fitness_options, no_wind_activities_structured jsonb, no_wind_verdict, cultural_notes_family, night_scene, areas_recommendation, scene_summary, proximity_reality.

---

**destination_meta** — Data quality and research status

Key columns: destination_id FK, confidence_score, data_status, manually_verified boolean, ai_generated_fields text[], source_count, personal_experience boolean.

---

### 1B. Supplementary Content Tables (public read RLS — query directly)

---

**schools** — Kite schools and instructors

183 rows across 25 active destinations. `kv_pick` boolean (max 1 per destination — set manually). Sort via `get_schools_by_destination_v1` RPC: kv_pick DESC, manually_verified DESC, capability_count DESC, name ASC.

Key columns: destination_id FK, school_name, slug, disciplines_supported text[], website/whatsapp/email/phone/instagram_url/google_maps_url, storage_available/rescue_available/gear_rental_available/foil_rental_available boolean, rating, manually_verified, skill_levels_taught text[], certifications text[], lesson_price_usd_from, kv_pick boolean.

⚠️ `kite_schools` table was DROPPED in migration 044. `schools` is the ONLY source of school data.

---

**destination_wind_seasons** — Per-season wind data (migration 037)

Full schema in Phase 2 section above. All 25 active destinations have seasons populated. 48 rows total. 2+ seasons per destination for multi-season spots (e.g. KUZI + KASKAZI for Diani Beach, Kenya). Query directly with `destination_id` filter.

---

**destination_areas** — Neighbourhood/area intel per destination (177 rows)

One row per distinct area. Key columns: destination_id FK, name, distance_to_kite_spot_km, vibe, best_for, accommodation_options, restaurants_nightlife, price_relative, walkable_to_kite, kiter_verdict (BEST BASE / GOOD OPTION / TRADEOFF / SKIP), is_active.

Sort: BEST BASE(1) → GOOD OPTION(2) → TRADEOFF(3) → SKIP(4). Max 4 shown in picker.

---

**kite_hubs** — Kite scene hubs (139 rows)

Key columns: destination_id FK, name, type, location_description, school_names text[], facilities text[], vibe, crowd, accommodation_on_site, kiter_verdict (SOCIAL HUB / GOOD BASE / QUIET OPTION / LESSONS ONLY), manually_verified, is_active.

Sort: SOCIAL HUB(1) → GOOD BASE(2) → QUIET OPTION(3) → LESSONS ONLY(4). Max 2 shown in trip plan output.

---

**visa_requirements** — Entry requirements per destination/passport pair

251 rows. Key columns: destination_id FK (nullable), passport_country, visa_required, visa_free_days, visa_type, cost_usd, processing_days, application_url, e_visa_available, valid_as_of.

Query: `supabase.from('visa_requirements').select('*').eq('destination_id', id).eq('passport_country', nationality).single()`

---

**kite_accommodations** — Accommodation near kite spots

141 rows. Key columns: destination_id FK, name, area_name, distance_to_launch_m, price_per_night_usd_low/high, gear_storage boolean, kite_packages boolean, community_favourite boolean, website, is_active.

Sort: community_favourite DESC, distance_to_launch_m ASC. Show max 2 in trip plan.

---

**kite_gear_services** — Gear rental and services

161 rows. Key columns: destination_id FK, service_type (rental/repair/storage), name, price_per_day_usd, stranded_risk (LOW/MEDIUM/HIGH), is_active.

---

**destination_daily_living** — Daily logistics

Key columns: destination_id FK, transport_primary, transport_price_range, transport_negotiate boolean, ride_app_available boolean, payment_app_name, payment_app_tourist_ok boolean, usd_accepted boolean, tap_water_safe boolean, sim_worth_it boolean, sim_carrier, sim_where_to_buy, english_sufficient boolean, english_notes, beach_dress, offbeach_dress, tip_restaurant_pct, bargaining boolean, is_active.

---

**destination_wifi** — WiFi and connectivity (8 rows — see full schema in original ref)
**nightlife_venues** — Bars and clubs (544 rows — see full schema in original ref)
**no_wind_activities** — Activities for wind-free days (416 rows — see full schema in original ref)
**recovery_services** — Massage, physio, sports recovery (444 rows — see full schema in original ref)
**restaurants** — Dining options (623 rows — see full schema in original ref)
**sim_card_options** — Mobile data options (0 rows — see full schema in original ref)
**transportation** — Transport vendors (464 rows — see full schema in original ref)

---

### 1C–1I. Other Tables (unchanged from previous version)

culture_safety (25 rows), medical_emergency (0 rows), coworking_spaces (274 rows) (Phase 1 schema tables — see previous reference for full schemas)

social_intel_staging (352 rows), wind_community_reports (0 rows), accommodation_reviews (0 rows), destination_condition_updates (0 rows) (Phase 1 social sourcing tables)

open_meteo_raw (300 rows), google_places_raw (2,388 rows), seabreeze_raw (26 rows), rome2rio_raw (0 rows), noaa_copernicus_raw (300 rows), approved_social_intel (300 rows) (Phase 1 data source tables)

kite_spots (117 rows), spot_regulations (27 rows), destination_kiter_logistics (27 rows) (Spot & logistics tables — migrations 030-032)

research_staging (837 rows), destination_media, airports (Supporting tables)

---

### 1J. Pending Tables (NOT YET CREATED)

**saved_trips** (migration 045 — not applied) — see Auth Architecture section above for schema.

---

## 2. DATABASE FUNCTIONS / RPCs

### Active RPCs (used by frontend)

---

**explore_destinations_multi_sport_v1** (migration 011)
Parameters: input_month int, input_sports text[], input_region text, input_skill_level text
Returns: destination cards (23 columns), sorted by destination_strength_score DESC
SECURITY DEFINER: ✅ | Status: Working ✅

---

**get_destination_detail_by_slug_v1** (migration 006)
Parameters: input_slug text
Returns: single row with ~57 aliased fields
SECURITY DEFINER: ✅ | Status: Working ✅

⚠️ All field names are ALIASED — never use raw DB column names.

---

**get_schools_by_destination_v1**
Parameters: input_slug text
Returns: table of school records, sorted kv_pick DESC, manually_verified DESC, capability_count DESC, name ASC
SECURITY DEFINER: ✅ | Status: Working ✅

---

**get_fitness_by_destination_v1** (migration 012)
Parameters: input_slug text
Returns: table (destination_fitness_summary + destination_fitness_facilities)
SECURITY DEFINER: ✅ | Status: Working ✅

---

### Admin Review Queue RPCs

get_pending_social_intel_v1, approve_social_intel_row_v2 (3 params: p_id, p_excluded_indices, p_custom_venues), reject_social_intel_row, flag_social_intel_row, get_all_destinations_for_admin, add_manual_venue_v1

See previous reference version for full parameter signatures.

---

### Data Ingestion RPCs

ingest_open_meteo, ingest_google_places, ingest_reddit_posts, ingest_seabreeze_data, ingest_rome2rio_data, ingest_noaa_copernicus_data

See previous reference version for full parameter signatures.

---

## 3. SUPABASE EDGE FUNCTIONS

### generate-trip-plan (CURRENT — 10-11 calls, to be rebuilt in Phase 3)

File: `supabase/functions/generate-trip-plan/index.ts`
Purpose: Main trip plan generator — streams 10-11 Claude API calls simultaneously
Model: `claude-sonnet-4-6`
Output: NDJSON stream — one line per section

Current call map (10-11 calls):
overview, on_the_water, visa, flights, getting_around, accommodation, budget, day_structure, pack (⚠️ NOT `packing`), reality_check, day_structure_expanded (conditional on nights > 3)

⚠️ Section ID is `pack` NOT `packing`. Do not change this — it was a previously fixed bug.

Phase 3 target: rebuild to 3 calls (verdict, on_the_water_narrative, lifestyle_narrative) all firing in parallel, under 10 seconds total. **Do not rebuild until trip plan output page is designed.**

### Other Edge Functions (unchanged from previous version)

extract-social-intel, ingest-reddit, fetch-open-meteo, parse-open-meteo, search-google-places, parse-google-places, scrape-seabreeze-forums, scrape-rome2rio-transport, ingest-noaa-copernicus-data, classify-facebook-posts, post-facebook-comments

See previous reference version for full documentation.

---

## 4. MIGRATION HISTORY

### Root Folder (`migrations/`) — All Applied ✅
001-006 applied. See previous reference version.

### Supabase Folder (`supabase/migrations/`) — Applied Status

Migrations 006-036 applied. See previous reference version for full list.

| File | Description | Status |
|---|---|---|
| `037_destination_wind_seasons.sql` | Created `destination_wind_seasons` table (seasonal wind data, replaces single-row conditions approach). Public read RLS. Populated for all 25 active destinations. | ✅ Applied |
| `038–043` | Intermediate migrations (content varies — check migration files directly) | ✅ Applied |
| `044_drop_kite_schools.sql` | Dropped `kite_schools` table. `schools` table is the single source of truth for all school data (183 rows). | ✅ Applied |
| `045_saved_trips.sql` | Will create `saved_trips` table for user saved trip plans. Required before auth/save feature. | 🔴 NOT YET APPLIED |
| `049_destination_areas_display_name.sql` | Latest applied migration (as of 2026-06-04). | ✅ Applied |

**⚠️ Do not apply migration 045 until auth (magic link) is implemented.**

---

## 5. FRONTEND PAGES & COMPONENTS

### Pages (`src/pages/`) — Phase 2 State

---

**DestinationPage.tsx** (formerly DestinationDetail.tsx)

Route: `/destinations/:slug`
Branch: `feat/phase2-redesign`
Status: Partially implemented — hero, wind season tabs, my base with spine all render. Several bugs (see Phase 2 bugs section above).

Data fetching (parallel on mount):
```typescript
const { data: dest } = useDestinationDetail(slug);           // RPC — normalized tables
const { data: seasons } = useWindSeasons(dest?.destination_id);    // direct
const { data: hubs } = useKiteHubs(dest?.destination_id);          // direct
const { data: areas } = useDestinationAreas(dest?.destination_id); // direct
const { data: schools } = useSchools(slug);                        // RPC
```

Current data-testids (confirmed from live browser):
device-frame, destination-page, back-to-explore, season-tab-0, season-tab-1, season-card, base-tab-0, base-tab-1, base-tab-2, base-card, spine-stop-gear, spine-stop-sleep, expand-toggle-sleep, spine-stop-eat, expand-toggle-eat, spine-stop-recover, expand-toggle-recover, spine-stop-social, expand-toggle-social, spine-stop-evening, expand-toggle-evening, invest-footer, plan-my-trip-cta

Missing data-testids (must add): bottom-tab-bar, stats-band, hero-h1, the-catch, section-wind, section-my-base

Page structure (current implementation):
1. KVNav (dark, sticky)
2. Hero zone (dark): ‹ EXPLORE, destination name H1, region/country, stats band (horizontal scroll strip, all viewports), mobile Plan My Trip CTA above Wind Season
3. Light section: eyebrow, vibe_summary, practical_notes
4. Wind Season section (01): tabs + season card
5. My Base section (02): area picker + vibe text + spine (Kite Zone → Gear → Sleep → Eat → Recover → Social → Evening)
6. Invest footer (dark): CTA

Page structure (target after DESTINATION-PAGE-REQUIREMENTS.md is implemented):
1. KVNav
2. Hero (dark): back, name, region, stats band (visible ALL viewports), The Catch warning, Best For (green), Skip If (amber), Plan My Trip CTA
3. Wind Season section: full-width, single-column
4. My Base section: full-width, single-column, Social/Evening collapsed by default
5. Invest footer
6. BottomTabBar (mobile only, fixed)

---

**ExplorePage.tsx** (formerly Explore.tsx)

Route: `/`
Status: Old implementation still in place. Phase 2 redesign not yet started for this page.

---

**TripPlanPage.tsx** (formerly TripPlan.tsx / TripPlannerModal+Output)

Route: `/trip/:slug`
Status: Old implementation still in place. Phase 2 intake redesign (3-step reordered) and output redesign not yet started.

Phase 2 intake step order (to implement):
- Step 1 — "You": group + skill (two fun taps, fast first step)
- Step 2 — "Your setup": weight + gear + budget (three taps)
- Step 3 — "When": nationality + dates (commitment last, heaviest input)

---

**ProfilePage.tsx** — Not yet built. Requires migration 045 + auth.

**SharedTripView.tsx** — Not yet built.

**Admin.tsx, JarvisDashboard.tsx** — Existing, untouched.

---

### Shared Components (Phase 2)

---

**KVNav.tsx**
- Mobile (< 768px): 50px sticky, dark bg + blur. Logo left, profile avatar (30px circle, initials) right.
- Tablet/Desktop (≥ 768px): 56px. Logo left, links centre (Explore · Destinations), Profile + "Plan My Trip" CTA right.
- Logo: `font-mono text-[13px] font-semibold tracking-[0.06em]` — "Kite" white, "Vurse" accent.

**BottomTabBar.tsx**
- Mobile only (`md:hidden`). Fixed bottom-0. 62px height + safe area.
- Tabs: EXPLORE (→ `/`), PLANNER (→ opens DestinationPickerSheet), PROFILE (→ `/profile`)
- data-testid="bottom-tab-bar" ← MUST ADD (currently missing)
- Active: `text-accent-deep` + `bg-accent/10` pill

**DeviceFrame.tsx**
- See Design System section above.
- data-testid="device-frame" already present.

**VerdictPill.tsx** — See Design System section above.

**DestinationPickerSheet.tsx** — vaul Drawer. Fetches destinations directly (is_active=true). Grouped by region. Real-time client-side search.

---

### Trip Planner Components (Phase 2 — to be built)

**TripPlanIntake** — 3-step reordered intake (see step order above)
**TripPlanOutput** — 6-section output (Water, There & In, Sleep, Lifestyle, No Wind, Costs)
**VerdictZone** — dark zone with VerdictPill, kite quiver, 3 reasons
**ShareModal, SaveModal** — vaul Drawers

---

## 6. DATA INGESTION STATE (as of 2026-06-04)

All Phase 1 data collection complete.

- Normalized tables: 101 rows each (25 active, 76 inactive)
- destination_wind_seasons: 48 rows (all 25 active destinations populated)
- destination_areas: 177 rows
- kite_hubs: 139 rows
- schools: 183 rows across 25 destinations
- kite_accommodations: 141 rows
- kite_gear_services: 161 rows
- visa_requirements: 251 rows
- coworking_spaces: 274 rows
- destination_fitness_summary: 25 rows
- destination_fitness_facilities: 287 rows
- culture_safety: 25 rows
- kite_spots: 117 rows
- spot_regulations: 27 rows
- destination_kiter_logistics: 27 rows
- open_meteo_raw: 300 rows
- noaa_copernicus_raw: 300 rows
- approved_social_intel: 300 rows
- social_intel_staging: 352 rows (0 pending approval)
- google_places_raw: 2,388 rows
- seabreeze_raw: 26 rows
- nightlife_venues: 544 rows
- no_wind_activities: 416 rows
- recovery_services: 444 rows
- restaurants: 623 rows
- transportation: 464 rows
- research_staging: 837 rows
- destination_wifi: 8 rows
- sim_card_options: 0 rows
- medical_emergency: 0 rows
- rome2rio_raw: 0 rows
- wind_community_reports: 0 rows
- accommodation_reviews: 0 rows
- destination_condition_updates: 0 rows
- approval_queue: 6 rows (4 pending)
- facebook_engagement_queue: 7 rows

---

## 7. ENVIRONMENT

### Active Supabase Project: `xthaiitjoccrrqivjlor`

| Env Var | Purpose |
|---------|---------|
| `VITE_KITEVERSE_SUPABASE_URL` | Active project URL |
| `VITE_KITEVERSE_SUPABASE_ANON_KEY` | Active anon key (frontend only) |

### Legacy: `ouqqfpqsbaijbinpyiio` — DO NOT USE

### Tech Stack (confirmed from package.json 2026-05-29)

| Library | Version | Notes |
|---|---|---|
| React | 18.3.1 | |
| TypeScript | 5.8.3 | |
| Vite | 5.4.19 | |
| react-router-dom | 6.30.1 | React Router v6 |
| @tanstack/react-query | 5.83.0 | v5 — all data fetching |
| tailwindcss | 3.4.17 | **v3 NOT v4** |
| @supabase/supabase-js | 2.105.3 | |
| vaul | 0.9.9 | All bottom sheets |
| sonner | 1.7.4 | All toasts |
| lucide-react | 0.462.0 | Icons |
| recharts | 2.15.4 | Charts (costs section) |
| date-fns | 3.6.0 | Date utilities |
| Vitest | 3.2.4 | Unit tests |
| Playwright | 1.60.0 | E2E tests |
| @testing-library/react | 16.0.0 | Component tests |

All Radix UI primitives and shadcn/ui components are installed. Use them — don't install alternatives.

### Dev Environment

```
Repo:        C:\Users\danwi\kite-explorer
Dev server:  localhost:8080 (npm run dev)
Branch:      feat/phase2-redesign
Shell:       PowerShell (never use &&)
```

### Design Tokens

```
Page bg:        #f6f4ef
Dark bg:        #0a0e14
Accent:         #2DABE5
Accent deep:    #006194
Primary text:   #0a0e14
Muted text:     #6b7a8d
Label text:     #54606c
Inverted:       #e8eaed
Green:          #7ed3a2
Amber:          #d99a3a
Border light:   rgba(10,14,20,0.09)
Border dark:    rgba(255,255,255,0.08)
```

---

## APPENDIX: Critical Rules Summary

1. **SECURITY DEFINER** — every RPC reading RLS-protected normalized tables must declare SECURITY DEFINER + `SET search_path = public`. Without it: returns all nulls with no error.

2. **pgcrypto** — any RPC using `digest()` must include `extensions` in search_path.

3. **Never generate factual data via Claude API** — airport codes, wind data, hazards, school names, visa requirements must come from the DB. Claude API generates narrative only.

4. **Anon key only in frontend** — `VITE_KITEVERSE_SUPABASE_ANON_KEY`. Never expose service_role key.

5. **Slug-based routing only** — no ID routes.

6. **RPC-first for normalized tables** — never direct query. But supplementary tables (destination_wind_seasons, destination_areas etc.) DO have public read RLS and query directly.

7. **Schools RPC only** — `get_schools_by_destination_v1`. kite_schools table is DROPPED. Do not reference it.

8. **destination_media JOIN** — joins on `d.destination_id` (text) not `d.id` (uuid).

9. **Section ID is `pack` not `packing`** — previously fixed bug, do not revert.

10. **approve_social_intel_row_v2 signature** — 3 params: (p_id UUID, p_excluded_indices INT[] DEFAULT NULL, p_custom_venues JSONB DEFAULT NULL). The 2-param version was dropped in migration 032.

11. **Mobile is primary.** Every component must be tested at 375px before committing. `hidden lg:block` on decision-critical content is a bug, not a feature.

12. **TanStack Query v5 for all data.** No raw useEffect + useState for async data. No exceptions.

13. **vaul for all bottom sheets.** No custom sheet components.

14. **Tailwind v3 (NOT v4).** Syntax differences matter.

---

## 8. DATA COLLECTION SCRIPTS (unchanged — Phase 1 complete)

All data collection scripts live in `scripts/data-collection/`. Launchers: `run_ingester.ps1`, `run_parser.ps1`. Python scripts: `kitevurse_places_ingester.py`, `kitevurse_perplexity_parser.py`, `kitevurse_perplexity_collector_v4.py`.

Parser prompt map P7–P15 all complete. See previous reference version for full details.

---

## 9. MAINTENANCE SCRIPTS (unchanged)

Nightly auto-update at 5:00 AM via Windows Task Scheduler. Scripts in `scripts/maintenance/`. See previous reference version for setup and usage.

---

## 10. PROJECT SYSTEM PROMPTS (Phase 2 updated)

Fetch instruction for all Claude Code sessions:
```
At the start of every session, fetch and read:
https://raw.githubusercontent.com/Kaivurse/kite-explorer/main/CLAUDE-REFERENCE-FULL.md
This is the single source of truth for all KiteVurse context.
```

### 10B — Frontend / UI Dev (Phase 2 updated)

```
At the start of every session, fetch and read:
https://raw.githubusercontent.com/Kaivurse/kite-explorer/main/CLAUDE-REFERENCE-FULL.md
This is the single source of truth for all KiteVurse context.

You are the frontend developer for KiteVurse (Phase 2 redesign).
Stack: React 18 + TypeScript + Vite + Tailwind CSS v3 + TanStack Query v5 + vaul + sonner.
Repo: C:\Users\danwi\kite-explorer. Dev server: localhost:8080. Shell: PowerShell (never &&).
Active branch: feat/phase2-redesign.

MANDATORY FIRST STEP: Read CLAUDE-REFERENCE-FULL.md and CLAUDE-DECISIONS.md before touching any file.

Rules:
- Mobile responsiveness is MANDATORY. Primary viewports: 375px and 390px. Test before committing.
- `hidden lg:block` is NEVER acceptable for decision-critical content (stats, CTAs, warnings).
- TanStack Query v5 for all data. No raw useEffect + useState for async data.
- vaul for all bottom sheets. sonner for all toasts. No alternatives.
- Supplementary tables (destination_wind_seasons, destination_areas, kite_hubs, etc.) query directly — no RPC.
- Normalized tables (destination_conditions, destination_editorial, etc.) use get_destination_detail_by_slug_v1 RPC.
- Schools: get_schools_by_destination_v1 only. kite_schools table is DROPPED — do not reference it.
- Design tokens, colors, typography in CLAUDE-REFERENCE-FULL.md Design System section.
- VerdictPill, DeviceFrame, VerdictBadge, RideableDaysDots spec in CLAUDE-REFERENCE-FULL.md.
- Tailwind v3 NOT v4.
- All interactive elements: min 44×44px touch target.
- End every session with a HANDOFF block.
```

### 10A, 10C–10H — All other prompts (unchanged from previous version)

---

## FACEBOOK COMMUNITY STRATEGY (unchanged)

See previous reference version for full post rotation, tone rules, and what NOT to do.

Revenue streams: Booking.com affiliate (post-launch, 2-3 months traffic first), Skyscanner CPC, SafetyWing ($15-25/signup), KiteVurse Pro ($49/year Month 9). No affiliates until the site has a following.