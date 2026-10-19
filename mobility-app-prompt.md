# Prompt: Build a 15-Minute Daily Mobility Trainer App

## Role

You are a senior full-stack engineer working with a mobility/movement-coaching mindset. Build a complete, working, offline-capable web app (PWA) that delivers a **15-minute daily mobility session** where the user picks the area of the body to train. Prioritize correctness of exercise prescriptions, clarity of on-screen guidance, and a frictionless in-session experience (the user's hands are on the floor, phone is 1–2 m away).

If you build this inside an existing repo, read `CLAUDE.md` and `.claude/rules/` first and follow them (TypeScript strict, named exports, kebab-case files, Conventional Commits, tests in `tests/`). Otherwise default to: **React + Vite + TypeScript (strict), Vitest, no backend, localStorage persistence, PWA with service worker for offline use.**

## Product goals

1. One tap from open to "Start session". Daily use must feel faster than deciding what to do.
2. The user picks target area(s); the app generates a **15:00 session** (±15 s) built from a curated exercise library.
3. Every exercise shows **exactly how long to hold or how many reps**, per side, plus a visual guide and short cues.
4. A hands-free guided mode: automatic timer, rep counter, side switching, audio/vibration cues, pause/skip/back.
5. Structure resembles a coached "daily mobility WOD": a short prep, targeted work, then an integration/finish. (The user referenced "GOWOD"-style programming. I could not verify what that program specifically prescribes, so do NOT claim to replicate it. Implement the structure described in this document instead, and state in the README that it is "inspired by" daily mobility-class formats, not affiliated.)

## Non-goals (do not build)

- No accounts, backend, social features, payments, or analytics.
- No medical diagnosis or rehab claims. This is general mobility/fitness, not physiotherapy.
- No stock photos or video embeds from third parties (licensing, offline, bloat).

## Target-area selection

Multi-select chips, 1–3 areas, plus presets:

- **Areas:** Neck, Shoulders, Thoracic spine (upper back), Lower back/Lumbar, Hips, Glutes, Hamstrings, Quads/Hip flexors, Adductors (groin), Calves/Ankles, Wrists/Forearms, Full body.
- **Presets:** "Desk reset" (neck, thoracic, hips, wrists), "Lower-body day", "Upper-body day", "Surprise me" (rotates by least-recently-trained area), "Full body".
- Show an optional **"Avoid" toggle** per area for pain/injury ("exclude exercises that load this area") and a visible note: *stop any movement that causes sharp, pinching, or radiating pain; this app is not medical advice.*
- Filters: equipment (none / wall / chair / band / foam roller / lacrosse ball). Default = none + wall + chair. Never schedule an exercise requiring equipment the user hasn't enabled.
- Level: Beginner / Intermediate / Advanced, which changes hold times, ROM demands, and exercise selection (not just numbers).

## Session architecture (must sum to 15:00)

| Block | Time | Purpose |
|---|---|---|
| 1. Prep | 1:00 | Breathing (e.g., 4s in / 6s out diaphragmatic) + setup cue. Optional skip. |
| 2. Dynamic prep | 2:00 | 2–3 low-intensity dynamic moves/CARs relevant to the selected areas |
| 3. Targeted block | 9:00–10:00 | 4–6 exercises on selected areas: mix of loaded/active end-range work, and static or PNF stretches |
| 4. Integration | 1:30–2:30 | 1–2 full-range movements that tie the area back to function (e.g., deep squat flow, thoracic rotation in lunge) |
| 5. Cool-down | 0:30–1:00 | Easy breathing in a restorative position |

**Time accounting must be explicit and tested:** each exercise's duration = (work time × sides) + (transition time, 5–10 s between exercises, 3 s per side switch). The generator must compute total duration from the data, not from hardcoded guesses, and fill/trim to 15:00 ±15 s by adjusting hold times within the exercise's allowed range, or by swapping exercises. Distribute time across multiple selected areas proportionally (weighted by user priority if provided).

**Selection rules for the generator (deterministic given a seed):**
- Include at least one **active/strength-at-end-range** item (e.g., CARs, 90/90 lift-offs, PAILs/RAILs) in each session, not only passive stretching.
- Avoid repeating the same exercise two days in a row unless the area has ≤ 3 matching exercises. Use a rotation history stored locally.
- Order within the block: positions that share a body orientation are grouped (standing → kneeling → seated → floor/supine) to minimize transitions.
- Respect contraindication tags (see schema) against the user's "Avoid" areas.
- Never schedule the same tissue stretched passively more than ~2× per session.

## Prescription rules (the user explicitly needs reps or hold time for each)

Every exercise must show a clear prescription line, e.g. **"Hold 45 s per side"** or **"8 slow reps per side"**, in large type, before and during the exercise. Use these defaults, stored as data and editable:

- **Static stretch:** 30–60 s per side (Beginner 30 s, Intermediate 45 s, Advanced 60 s). Breathe slowly; deepen on exhales; stretch sensation only, never pain.
- **Dynamic mobility (leg swings, thoracic rotations, cat-cow):** 8–12 controlled reps per side, ~2–3 s per rep. No bouncing.
- **CARs (controlled articular rotations):** 3–5 slow reps per direction per side, ~10–20 s each rep, maximal tension and smallest compensation.
- **Isometric/PNF (PAILs/RAILs):** 10–15 s contraction at ~20–50% effort, then 10 s deeper stretch, 2–3 rounds per side. Cap the total at the exercise's time budget.
- **Positional end-range holds (deep squat, couch stretch, 90/90):** 45–90 s with rocking/breathing cues; Beginner gets a supported variation.
- **Soft-tissue (ball/roller):** 30–60 s per spot; hard-limit spots near joints/spine; flagged "optional equipment".

Treat these numbers as **defaults based on common practice, not settled science**: put them in a clearly labeled data file (`prescriptions.ts`) with a comment stating they are defaults a qualified coach should review. In the README, cite the evidence base honestly: research supports static stretching for improving range of motion at roughly ≥30 s per muscle group, but evidence for injury prevention and for long-term benefits of specific protocols is mixed or limited. Do not invent citations; if you cite a source, include a real one and mark anything you can't verify as "unverified".

## Exercise library (the core asset — do this carefully)

Create **at least 60 exercises** (≥ 5 per area, ≥ 8 for hips and thoracic/shoulders), with real coaching content. Store as typed data, not hardcoded in components.

```ts
interface Exercise {
  id: string;                      // kebab-case
  name: string;
  areas: AreaId[];                 // primary first
  secondaryAreas: AreaId[];
  type: 'static' | 'dynamic' | 'cars' | 'pnf' | 'positional' | 'soft-tissue' | 'integration';
  position: 'standing' | 'kneeling' | 'seated' | 'quadruped' | 'prone' | 'supine' | 'side-lying';
  equipment: Equipment[];
  levels: Level[];                 // which levels it suits
  bilateral: boolean;              // true = repeat per side
  prescription: {
    mode: 'hold' | 'reps';
    hold?: { beginner: number; intermediate: number; advanced: number; min: number; max: number };
    reps?: { beginner: number; intermediate: number; advanced: number; tempo: string };
    sets: number;
  };
  setup: string[];                 // 2–4 short steps to get into position
  execution: string[];             // 2–4 steps
  cues: string[];                  // 2–4 short cues shown during exercise
  breathing: string;
  feelIt: string;                  // where you should feel it
  commonMistakes: string[];        // 2–3
  regression: string;              // easier variation
  progression: string;             // harder variation
  stopIf: string;                  // pain/contraindication warning
  contraindications: AreaId[];     // areas to avoid if user flags pain
  guide: GuideSpec;                // see Visual guides
}
```

Content requirements:
- Cues must be physiologically accurate and specific (e.g., for the couch stretch: "tuck pelvis under, squeeze rear glute, ribs down; don't arch lower back"). No generic filler like "feel the stretch".
- Include well-established staples: cat-cow, thread-the-needle, open books, wall angels, shoulder CARs, hip CARs, 90/90 hip switches, couch stretch, half-kneeling hip flexor stretch, pigeon/figure-4, deep squat hold with prying, adductor rock-back, hamstring (supine strap/band or wall), calf/ankle knee-to-wall, wrist flexor/extensor stretches, cervical retraction and gentle side bend, child's pose variations, world's greatest stretch, etc.
- Mark which exercises are the author's best judgment vs. widely standard; add `reviewStatus: 'needs-professional-review'` to anything not a standard textbook movement, and surface a small "review" note in an About page.
- Write a unit test that validates every exercise against the schema (all required fields non-empty, prescription ranges coherent, bilateral flag consistent, no `any`).

## Visual guides (essential)

Because there is no external image dependency, produce **original SVG illustrations** for every exercise:

- A clean stick-figure/segmented-body style, consistent proportions, one reusable `<BodyFigure>` component driven by joint-angle data (`GuideSpec = { start: Pose, end: Pose, highlight: AreaId[], arrows?: Arrow[] }`).
- **Two-frame animation** (start → end position, ~2 s ease, looping, respecting `prefers-reduced-motion` by showing a static side-by-side instead).
- Highlight the **target muscle/area** with an accent color and a "feel it here" label; draw **arrows** for the direction of movement, and a small **"avoid" marker** for common errors (e.g., red arch on lower back).
- Show **floor/wall/chair/prop** as simple shapes when required.
- Provide a "Setup" tab (static pose + numbered steps) and a "Doing it" tab (animation + cues).
- Honest limits: stick-figure poses are approximate. Include an optional `videoUrl?: string` field and a "Watch a demo" link slot (user-supplied; empty by default) so a coach can later attach real video. Mention in the README that illustrations should be checked by a qualified coach before use as formal instruction.
- Accessibility: all guides have text alternatives (the setup/execution steps), 4.5:1 contrast, keyboard navigable.

## Guided session mode (the main screen)

- Full-screen, high-contrast, large text visible from ~2 m. Dark mode default, light mode optional.
- Top: block name + progress (e.g., "Targeted 3/5") and total time remaining. Center: the animated guide. Bottom: **prescription in large type**, side indicator ("RIGHT side"), and the countdown ring.
- **Hold exercises:** circular countdown; a haptic/beep at start, halfway, and last 3 s; automatically prompts "Switch sides" with 3 s buffer.
- **Rep exercises:** metronome-style tempo guide (visual pulse + optional tick) matching the `tempo`; rep counter that auto-advances, with a manual +/− override.
- Controls: pause/resume, skip, previous, +10 s, "Too intense, use easier version" (swaps to `regression` and recalculates timing), and "Stop". Big tap targets (≥ 48 px). Keyboard shortcuts (space = pause, ←/→ = prev/next).
- Audio: Web Audio beeps (no files), optional speech-synthesis announcements of the next exercise and cues, all toggleable. Screen wake lock while a session runs (fall back gracefully if unsupported).
- Next-up preview during the last 5 s of each item. A 10 s "Get into position" countdown before each new exercise, shown with its setup steps.
- Survives screen lock/tab backgrounding: base timers on timestamps (`performance.now()`/`Date.now()`), not `setInterval` tick counts.

## Other screens

1. **Home:** area picker, level, equipment, "Start 15-min session" button, and a preview list of the generated session with the time budget and a "Shuffle" button.
2. **Session preview:** each exercise with its prescription, editable (swap, remove, reorder) with a live total-time readout that keeps the session within 15:00 ±15 s or warns.
3. **Library:** browse/search/filter all exercises by area, type, equipment, and level; detail view with the full guide and all fields.
4. **Progress:** streak, sessions this week, minutes per area (simple bar chart), log of past sessions. Local only, with Export/Import JSON.
5. **Settings:** audio, haptics, speech, units, default level, reset data.
6. **About/Safety:** disclaimer, how prescriptions were chosen, review status, credits.

## Technical requirements

- TypeScript strict, no `any`, no non-null assertions, named exports, explicit return types on exports, 100-char lines, single quotes, 2-space indentation.
- Architecture: `src/lib/` (generator, timing, prescriptions, history), `src/data/` (exercises, areas), `src/components/`, `src/types/`, `tests/` mirroring `src/`.
- The **session generator and time accounting must be pure functions** with deterministic seeded output, and fully unit-tested (Vitest): total duration within tolerance for every area combination and level, no disallowed equipment, avoid-area exclusion, bilateral doubling, rotation history effect.
- Test the timer engine (pause/resume, skip, backgrounding via fake timers), and add a smoke test that renders the app and starts a session.
- PWA: manifest, icons, service worker precaching, installable, fully offline after first load. Lighthouse PWA/accessibility ≥ 90 as a target.
- Responsive, mobile-first (the main use is a phone propped on the floor), also fine on tablet/desktop.
- No third-party network calls at runtime. Minimal dependencies (React, Vite, a PWA plugin; no heavy UI frameworks).
- Provide scripts: `npm run dev`, `build`, `test`, `lint`, `typecheck`, `format`.

## Process

1. Before coding, write a short plan: data model, generator algorithm with a worked time-budget example, component tree, and the list of exercises you will include per area. Flag any area where you lack confidence in the prescription.
2. Build the data layer + generator + tests first, then the guided-session timer, then UI, then SVG guides, then PWA.
3. Run typecheck, lint, tests, and build; actually run the app and exercise a full session in a browser (Playwright) and report what you verified vs. what you did not.
4. If something is uncertain (exercise validity, evidence, a safe cue), say so in the README and in code comments rather than guessing.

## Deliverables

- Working app in the repo, a README (setup, architecture, how to add an exercise, prescription defaults and their basis, known limitations), and a short `CONTENT_REVIEW.md` listing every exercise flagged for professional review.
- Final message: what was built, what was tested, what was not, and a prioritized list of improvements (e.g., real video demos, coach-reviewed programming, wearable integration).

## Acceptance criteria

- [ ] Selecting any combination of 1–3 areas yields a session of 15:00 ±15 s, verified by tests.
- [ ] Every exercise screen shows a hold time or rep count, side indicator, animated guide, and ≥ 2 cues.
- [ ] A full session runs hands-free through all blocks, including side switches, with audio/haptic cues, and recovers correctly after the tab is backgrounded for 30 s.
- [ ] Equipment and "avoid area" filters are never violated (tested).
- [ ] App is installable and fully works offline.
- [ ] No unverified medical or evidence claims in the UI or README; disclaimer visible before the first session.
