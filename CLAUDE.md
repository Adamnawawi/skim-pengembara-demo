# CLAUDE.md — Suara Anda, Skim Kita (LINDUNG PERKESO quiz funnel)

## Project overview
A mobile-first, single-page quiz funnel for PERKESO Johor's "Suara Anda, Skim Kita"
engagement campaign (18 Oktober 2026). Target audience: Malaysians who commute daily
or work in Singapore. The quiz surfaces personal "protection gaps" across three
pillars, then routes the person to register for the 18 Oct engagement session or
opt in to receive a WhatsApp/email summary.

This is a lead-generation and engagement tool for a government agency, not a
commercial product — tone should feel warm, respectful, and official, never
salesy or manipulative.

## Audience & emotional context
- Malaysians (mostly Johor-based) who cross the Tambak/Second Link daily or
  weekly for work in Singapore — motorcycle, bus, or car commuters.
- Life is split across a border: income earned in Singapore, family and home
  life anchored in Johor/Malaysia.
- Real anxieties this quiz should speak to: commute safety and fatigue,
  being the family's main income earner, having no safety net if injured or
  unable to work, not knowing their rights, feeling unheard in policy design.
- Copy should use "anda" (you) directly, plain Bahasa Malaysia, no corporate
  jargon, no fear-mongering — acknowledge the difficulty, then offer being
  heard/protected as the resolution.

## Tech stack (recommended)
Chosen to match PERKESO's existing infrastructure (Laravel is already used
for the EIS Case Management System PoC) so this can be hosted, handed over,
and maintained on the same stack the team already runs — rather than
introducing a one-off Node/Vercel dependency for a single campaign tool.

- **Backend:** PHP (Laravel) — routing, the scoring endpoint, and data
  persistence
- **Frontend:** Server-rendered Blade templates + Tailwind CSS + Alpine.js
  for the step-by-step reactivity (question transitions, live progress,
  result reveal) — no React, no build-heavy SPA, no Node hosting required
- **Animation:** CSS transitions for step changes; a lightweight library
  such as GSAP only if the bridge-progress motif (see Design system) needs
  finer control than CSS alone can give
- **Data capture:** MySQL, via a Laravel controller/model — matches however
  PERKESO's other internal systems already store data, no external Sheets/
  Apps Script dependency needed
- **Deployment:** Standard LAMP/Laravel hosting — whatever PERKESO IT
  already uses for internal PHP systems, or a subdomain under a PERKESO/
  MYFutureJobs-controlled domain
- **Analytics:** capture `ref` / `utm_source` query params on load and submit
  them with the response, matching the channel-tracking codes already defined
  in the campaign doc (`ngo-a`, `flyer-ciq`, `bas-handal`, etc.)

## Brand & design system
Colors (from PERKESO/L.I.F.T. campaign brand system):
- Navy blue — primary/trust — `#0B2F5C` (approx, confirm against PERKESO
  brand guide)
- PERKESO green — safety/"Lindung" — `#1D9E75` (approx)
- Amber/kuning — warmth, urgency, sunrise — `#EF9F27` (approx)
- White space — generous; avoid a cluttered/salesy government-form look

Typography:
- Headline/hook lines: bold sans-serif (Montserrat Black or similar), high
  contrast, sentence case (avoid ALL CAPS except for the 3-word psychological
  hooks already scripted in the campaign doc)
- Body: regular weight, comfortable line height (1.6–1.7), Bahasa Malaysia

Tone of voice: second person, direct, warm, never corporate filler. Say what
it does. Example: "Kerja Singapura. Tapi... kalau kecemasan, siapa tanggung?"
not "Kami ingin memaklumkan tentang skim perlindungan sosial baharu."

Signature interaction — "the bridge": progress through the quiz is shown as a
small icon (motorcycle/bus) moving across a stylized bridge between a Johor
skyline and a Singapore skyline, rather than a plain progress bar. This ties
the UI metaphor directly to the audience's actual daily journey.

## Content model
Three pillars (see `app/Models/Question.php` or a `config/questions.php`
seed array once scaffolded):
1. Perjalanan & kesejahteraan harian (commute wellness) — 3 questions
2. Kestabilan kewangan keluarga (family financial security) — 3 questions
3. Hak & suara pekerja (worker rights & voice) — 4 questions

Each question has answers: Ya (0 gap) / Kurang pasti (0.5 gap) / Tidak (1 gap).
Pillar gap % = (sum of gap values ÷ number of questions in pillar) × 100.
Bar color: <30% success/green, 30–59% warning/amber, ≥60% danger/red.

5 profile questions (non-scored) personalize the result copy and determine
the final CTA branch: attend the 18 Oct morning session in person, or opt in
to receive a summary remotely.

Full question text lives in `Cadangan_soalan.docx` — treat that as the
source of truth for wording; do not paraphrase government-facing copy
without approval.

## Page/flow structure
Single Blade view with Alpine.js managing step state client-side (no full
page reloads between questions); only the final submit hits the server.
1. Hook screen (headline + CTA), captures `ref`/`utm_source` silently on load
2. Step 1 — quick info capture (nama, WhatsApp, kawasan Johor)
3. Steps 2–11 — 10 scored questions, one per screen, bridge-progress visible
4. Steps 12–16 — 5 profile questions
5. Result screen — per-pillar gap bars, weakest-pillar callout referencing
   their chosen topic, and a CTA branch based on their attendance choice
   (scoring can be computed client-side in Alpine for the instant reveal,
   then re-validated server-side on submit before writing to MySQL)
6. Confirmation state — different copy for "attending in person" vs
   "remote summary"

## Data handling & compliance
- Collects: name, WhatsApp number, area, quiz answers, profile answers,
  channel ref code. No NRIC or financial account data — keep it that way.
- Since this is a Malaysian government agency collecting personal data,
  treat this as PDPA-relevant: state clearly on the info-capture screen what
  the data will be used for (event registration + a one-time summary
  follow-up), and do not add any secondary marketing use without a separate
  consent line.
- Persist submissions via a Laravel `FormRequest` + Eloquent model into
  MySQL; do not log WhatsApp numbers to application logs or expose them in
  any client-side analytics/tracking scripts.

## Static demo (pre-IT, management review)
Before this touches PERKESO's server or database, use a no-backend version
to get management sign-off on design and flow.

- Same design system, same content, same scoring logic as the full build —
  just HTML/CSS/JS with no PHP, no MySQL, no build step. Submissions don't
  need to persist anywhere real; the flow just needs to run end to end
  client-side.
- Package it as a single self-contained `index.html` (inline CSS/JS) so it
  has no broken links when moved — easiest to email as a plain attachment.
- Sharing options, both free: attach the `.html` file directly (opens by
  double-click, no server needed), or drop the folder into Netlify Drop /
  GitHub Pages for a shareable link ahead of the meeting.
- Test on an actual phone screen before presenting — this audience is
  mobile-first, and a laptop-only demo undersells the design.
- Once approved, continue from the same CLAUDE.md into the real Laravel +
  MySQL build (see Tech stack and Suggested build order below) — the static
  version is a preview shell, not a separate codebase to maintain long-term.
- For a temporary hosted (non-static) demo instead of a local file, see the
  hosting cost comparison discussed separately (Hostinger shared PHP hosting
  is the recommended option, ~RM9–15/month, cancel once IT takes over).

## Non-goals for this build
- No payment flow, no e-commerce
- No user accounts/login
- No dark patterns — the result screen is informational and inviting, not a
  hard paywall or high-pressure countdown

## Suggested build order for Claude Code
1. Scaffold a Laravel project with Tailwind CSS and Alpine.js wired in
   (via Laravel Mix/Vite, or plain CDN includes if a build step is
   unwanted on the target server)
2. Build the hook screen and bridge-progress component in isolation first
   (this is the visual signature — get it right before wiring the rest)
3. Build the quiz step engine as an Alpine.js component (state machine over
   the question list, all client-side, no reloads)
4. Wire client-side scoring logic + result screen
5. Add a Laravel route/controller + migration to persist submissions to
   MySQL, with server-side re-validation of the score before saving
6. Add ref/UTM capture and pass-through
7. Mobile QA pass (this audience is majority mobile, often on 4G at CIQ/bus)
8. Deploy to PERKESO's existing PHP/Laravel hosting, connect to a real
   short link/QR per channel code
