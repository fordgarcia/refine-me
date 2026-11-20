---
name: "refine-me"
description: "Judges designs and learns the user's taste. Use before generating any UI (Claude Design or Code), when the user critiques or compares designs, or on /refine-me context|analyze|compare|prefs|rules."
---

# Refine-Me

A design-judgment system with one goal: **good, consistent designs in fewer iterations.** It does three jobs:

1. **Before designing:** load the project context, the rules and the user's taste, and apply them, so the first output is already close.
2. **After every user judgment:** record it, interpret it and update the preferences, so the same correction is never needed twice.
3. **On request:** run a command (below).

The user talks naturally; Claude does the checking, scoring and bookkeeping. Never ask the user to fill in forms or scores. This skill does not exist to make Claude agree with the user. It exists to understand the reasoning behind their judgments.

## What every design must be

In priority order. A later item never excuses failing an earlier one.

1. **Works:** the task can be completed and its result is plainly visible.
2. **Never cut** (section 4.1): accessible, honest, recoverable. Pass/fail, not points.
3. **Defensive (design only):** holds up with the inputs that really happen (section 4.3).
4. **Lean and simple:** every element has a job, each job is done once, no overloaded views (section 4.2).
5. **Consistent and reusable:** the same thing looks, reads and behaves the same everywhere, built from shared components.
6. **Motion-ready, with planned motion:** components and layouts are built to animate, and the design ships with a motion plan (section 4.4).
7. **Pragmatic, not over-engineered:** the smallest thing that fully works (the ladder, section 4.5).

**Simple is not empty.** A design that looks minimal but hides what the task needs is broken, not lean.

**Removing is not restructuring.** A box, border, divider or indent often carries grouping. When one is removed, the grouping it showed must survive through a cheaper cue (section 4.6).

---

## Commands

Typed as `/refine-me <command> <arguments>`; the skill receives the rest of the line as its arguments. Each has plain-language equivalents. `/refine-me` with no recognised command runs `analyze`.

| Command | Plain-language equivalents | Does |
|---|---|---|
| `context [text]` | "here's the context", "this is for…", "the users are…" | Sets or updates this project's brief (section 0.2). With no text, shows it, each field marked Given or Assumed. |
| `analyze [target]` | "analyze this", "what's wrong with this", "refine based on what you've learned" | Full evaluation (section 3), including faults nobody asked about. Target defaults to the design in view, else the last one made. Never redesigns by itself. |
| `compare A B` | "which is better", "compare with the previous version" | Section 3 on both, side by side, a verdict per dimension. No overall winner unless one wins all four. The user's pick is logged (section 6). |
| `prefs [filter]` | "what have you learned about my taste" | Preferences with confidence, frequency and evidence IDs, then the rules in force. Filter by category (e-commerce, dashboard…) or dimension (geometry, colour…). |
| `confirm P-n` / `reject P-n` | "yes, make that a rule", "no, that's wrong" | Confirm: High confidence, rule added to `rules.md`. Reject: kept in history, never applied. Both are logged. |
| `rules [file or text]` | "use my rules", "import this rule file" | No argument: show the active base rules and their source. With a file or text: import it as the base rules (section 0.3). |

**Runs on its own:** section 1 every time a design is generated, and section 2 after every judgment. `analyze` runs only when asked, never after every change, because a full evaluation on each edit costs time and tokens.

---

## 0. The store (do this first, every time)

### 0.1 Where it lives

| Surface | Location |
|---|---|
| Claude Code, in a project | `.refine-me/` at the repo root (project) plus `~/.claude/refine-me/` (global taste). Read both; the project wins on conflict. |
| Claude Code, no project | `~/.claude/refine-me/` |
| Chat with memory tools | memory file `/topics/refine-me.md`: one headed section per layer, one Context section per project |
| None of the above | Keep the layers in the conversation; at the end, give the user the files to save. |

| Layer | File | Rule |
|---|---|---|
| Critique log | `critique-log.md` | **Append-only.** The user's words verbatim plus the interpretation. Never edited or deleted, even when a preference changes. |
| Preferences | `preferences.md` | Curated. Each entry links to its supporting log IDs. |
| Learned rules | `rules.md` | Only from confirmed (High) preferences. |
| Base rules | `rules-base.md` | Imported with `rules`. If absent, section 4 is the base rules. |
| Context | `context.md` | One per project. Facts, not taste. |
| Tool facts | `tool-facts.md` | Rendering or tool quirks learned the hard way, with their workaround. |

If the store is empty and section 8 (seed) hasn't been migrated, migrate it first.

### 0.2 Context fields

`context` parses free text into these fields and keeps the user's exact words in a `Raw` line under the table.

| Field | Example |
|---|---|
| Product and module | Apple PH store, Buy iPhone configurator |
| Users | first-time buyers on phones |
| Primary tasks and success | configure and see the exact price; compare sizes |
| Platform and size | phone 390px; desktop 1280px |
| Brand and must-keep | keep the product's colours, type and buttons |
| Constraints | course rubric, WCAG AA, tech stack, deadline |
| Tone goal | premium, calm, playful… |
| References | links, screenshots, liked or disliked designs |
| Out of scope | dark mode, other modules |

- Each field is **Given** (the user said it) or **Assumed** (inferred). A new `context` replaces Assumed values silently, and Given values only after showing old and new.
- Reply with the filled table and at most two questions, only about gaps that would change the design.
- Context never creates preferences. A liked or disliked reference is also logged as a judgment (section 2).

### 0.3 Which rule wins

| # | Layer | Decides |
|---|---|---|
| 1 | Never cut (section 4.1) | Cannot be overridden by anything below. |
| 2 | Project context | A brief such as "keep the product's look" beats everything below for that project. A preference it blocks is recorded as Contextual for that project and kept for others. |
| 3 | Base rules (`rules-base.md`, else section 4) | **What goes on the page**: which elements, sections, controls and states. |
| 4 | Learned rules and High preferences | **How it looks and moves**: palette, type, geometry, density, motion character. |
| 5 | Medium preferences | Applied by default, named in one line ("Leaning dark and premium, from your earlier picks"). |
| 6 | Low preferences | Offered as an option only, never applied silently. |

Preferences are matched by interface category: one recorded for e-commerce does not automatically carry to a developer dashboard. A preference can never add an element the base rules would cut; it can only restyle what stays. An imported rule file replaces sections 4.2 to 4.5 where they conflict, never section 4.1.

---

## 1. Before generating any design

1. Read `context.md`. If it is missing or thin, infer the gaps from the request and the conversation, save them as **Assumed**, and state them in one line before the design.
2. Read the base rules, `rules.md`, `tool-facts.md`, and the `preferences.md` entries for this interface category. Apply them by section 0.3.
3. Build with the ladder (section 4.5). Plan motion together with the components (section 4.4), not after.
4. Run **the checklist** (section 5) on the draft and fix what fails before showing it. Run item 11 (structure) on every subtraction: anything removed for leanness must not take a grouping cue with it.
5. Use methods known to render correctly (`tool-facts.md`). Do not add verification steps the tool forbids. If the user reports something that didn't render as written, log it as a bug and a tool fact, never as taste.
6. End the delivery with two lines:
   `Skipped: <left out on purpose, and the option if they want it>`
   `Not checked / risk: <e.g. real screen reader, translations, real data>`

---

## 2. After every user judgment (the learning loop)

Judgment → log record → interpretation → pattern check → proposal → approval → update.

For every meaningful judgment (criticism, praise, a pick between options, a rejection, "the previous one was better", a repeated correction):

1. **Append a log record**, keeping the user's exact words.
2. **Name the element**: which design, which component or decision.
3. **Signal**: positive, negative, neutral or ambiguous. "Somewhat decent" is weak positive, not approval.
4. **Interpret** as a hypothesis. For praise, list the concrete decisions that may explain it (palette, type, layout, a component). For a pick, the losers are evidence too: what do they share that the winners lack?
5. **Separate causes**: taste, a usability problem, a bug, or a misunderstanding.
6. **Classify the kind**: *what goes on the page* (it can inform the base rules, but only the user changes them) or *how it looks and moves* (a preference). Section 0.3 decides where it can act.
7. **Check history**: support, duplicate or contradiction. On contradiction, look for a context difference before overwriting.
8. **Set confidence** (section 7) and update `preferences.md`.
9. **Promote** to `rules.md` only with approval (`confirm P-n`, or a plain yes), asked in one line at the end of the reply, never as a form.
10. **Apply it** in the very next iteration.

Ask a clarifying question only when the ambiguity would change what gets built or recorded. Otherwise infer, record as Low, and move on.

```
### F-<yyyymmdd>-<n>
- Original: "<verbatim>"
- Design ref: <project / artboard / version / screenshot>
- Category: <e-commerce | dashboard | landing | mobile app | ...>
- Element: <component or decision>
- Signal: positive | negative | neutral | ambiguous
- Interpretation: <hypothesis, in Claude's words>
- Kind: what-goes-on-the-page | how-it-looks-and-moves | bug | context
- Dimension: <preference dimension or rubric criterion>
- Context: <requirements that shaped it>
- Confidence: Low | Medium | High
- Related: <F-ids, supporting or contradicting>
- Status: Pending | Confirmed | Rejected | Contextual
```

```
### P-<n>: <short name>
- Description:
- Reproduce: <concrete moves>   - Avoid: <concrete moves>
- Confidence:   - Frequency: <supporting records>
- Applies to: <categories>   - Exceptions: <contexts where it doesn't hold>
- Evidence: <F-ids>
```

---

## 3. Evaluation (`analyze`, `compare`)

Evaluate, recommend, and wait. Apply changes only when the user asks.

### 3.0 Order of work

1. **Context:** load `context.md`; infer empty fields from the design itself (content, controls, copy, device size) and the conversation, mark them Assumed, save them, and list them at the top of the output.
2. **Gate:** run section 4.1 (Never cut). Each failure goes to Must fix, and any failure caps the band at **Acceptable** whatever the points say.
3. **Checklist:** run section 5, including faults the user didn't mention. Every finding needs a concrete case: *this user, on this screen, with this content, gets this bad result.* No case, no finding. When a finding rests on an assumption, say so ("If users are first-time buyers, the storage step needs a size guide").
4. **Reference cues:** when a reference is given, list how it shows structure (indents, gaps, icon columns, label styles, regions) and check that the design keeps or replaces each one.
5. **Score** the four dimensions below **separately**. Never merge them into one number.

### 3.1 Objective quality: 100 points

Rate 1 to 5. Weighted score = (rating / 5) × weight.

| # | Criterion | Wt | Look at |
|---|---|---:|---|
| 1 | Task and usability | 12 | Task completes; result visible; low cognitive load; discoverable |
| 2 | Visual hierarchy | 12 | One main action per view; priority and flow; primary vs secondary |
| 3 | Consistency and reuse | 12 | Shared components; same action = same look and label; no drift |
| 4 | Interaction and motion | 10 | States designed; motion plan present, purposeful and motion-ready (section 4.4) |
| 5 | Lean | 8 | Element test passes; said once; no overload; Lean count near zero |
| 6 | Typography | 9 | Faces, scale, weights, line height, readability |
| 7 | Spacing and layout | 9 | Rhythm, alignment, grid, whitespace |
| 8 | Colour | 7 | Palette discipline, accent use, balance (contrast itself is in the gate) |
| 9 | Information architecture | 7 | Grouping visible (section 4.6: levels, two cues, label-removal test), dependencies nested, scanning, tables before cards when comparing |
| 10 | Visual identity | 5 | Distinct through type, spacing, material and motion, **not** added elements |
| 11 | Responsiveness | 5 | 320px to wide; overflow; reflow |
| 12 | Attention to detail | 4 | Alignment, borders, shadows, cropping |

Bands: 90+ Exceptional, 80–89 Excellent, 70–79 Good, 60–69 Acceptable, below 60 Poor. Accessibility has no points because it is a gate.

Scoring discipline:
- Every deduction cites something observable ("the 2TB card is off-screen at 390px"), not a feeling.
- What can't be seen is **Unavailable** (a static mockup has no hover; one width says nothing about reflow). Re-total over the available weights and say which were left out.
- **Self-grading guard:** for Claude's own fresh output, list the three weakest points before the score, and never score above 85 without a user judgment backing it. Self-scores drift high.
- Never reward decoration or complexity. Weigh an intentional choice's purpose before deducting.
- Keep apart: confirmed facts (read from code), observations (seen in a render) and interpretations.

### 3.2 Personal alignment

Against `preferences.md`: minimalism, density, geometry, typography, composition, colour, decoration, motion character, personality. Report matches, conflicts and new signals. Never present a preference as a universal rule.

### 3.3 Contextual suitability

Product, audience, task, platform, conventions, brand, constraints. When a preference conflicts with the context, name it and propose a compromise. Example: "You prefer dark luxe, but this module must keep the existing store's look for the course rubric. Keep the layout idea, keep the brand's colours."

### 3.4 Build quality

For designs: shared components versus one-offs, tokens versus hard-coded values, states complete. With code access, also: reuse, simplicity, no needless abstraction or dependency, semantic elements. Never infer code quality from screenshots; mark it Unavailable. Defensive checks are design-only (section 4.3); code-level error handling is out of scope for this skill.

### 3.5 Output (short)

| Output | Content |
|---|---|
| Context used | Given and Assumed fields |
| Gate | Pass, or each Never-cut failure |
| Overall | Score / available points, band, Unavailable criteria |
| Strongest | Three decisions that work, and why |
| Findings | **Must fix / Should fix / Nice to have**, each with *Problem*, *Case*, *Fix* (smallest first, preferably a removal), *If skipped* |
| Motion | Plan present? Gaps? |
| Personal alignment | Matches, conflicts, preference IDs |
| Context fit | Conflicts and compromises |
| Build quality | If verifiable; otherwise Unavailable |
| Lean | `Lean: -N elements possible` |
| Preference insights | New signals, contradictions, proposed promotions |
| Not checked | What this evaluation could not see |
| Skill gaps | While refine-me is in development: for each finding, the rule that allowed it or failed to catch it, and the edit that would catch it next time |

---

## 4. Base rules (default; replaceable with `rules`)

Adapted from Ponytail's "lazy senior designer": lazy about the solution, never about the task.

### 4.1 Never cut (fixed; no import overrides this)

- Text contrast at least 4.5:1 (3:1 at 24px+); control boundaries at least 3:1.
- Visible focus and full keyboard use; real buttons, links and labelled inputs.
- Targets 44–48px on touch screens; at least 24×24px (WCAG 2.2, 2.5.8) on pointer-and-keyboard desktop UIs. Colour is never the only signal; icon-only controls get a word or label.
- Reduced motion respected.
- Errors say how to fix them; nothing makes users lose or retype work, or get stuck.
- Honest pricing, consent and labels; a way to cancel or undo; a warning before destructive actions; no dark patterns.
- Anything the user explicitly asked for.

### 4.2 Lean and simple

- **The element test:** keep an element only if it has a **job** (one sentence: what it helps the user do or decide; "looks nice" is not a job), does it **once** on the screen, and would be **missed** if deleted.
- One message per section, one main action per view; the primary style is used once per screen.
- **Say it once:** don't show the same fact as a bar, a number, a label and a button caption.
- **State lives in the control:** a control that shows its own state (switch, checkbox, selected tab, stepper value) gets no second readout next to it. Add a readout only for what the control can't show, such as a slider's exact number and unit. If the state is unclear without a readout, fix the control (fill, knob position, a mark inside it) instead of captioning it. A word placed beside a switch can even be misread as the side the knob should be on.
- Plain text before containers: no chip, badge or box unless it does something. Showing a group counts as doing something; when you remove a container, keep its grouping with section 4.6's cues.
- Lists before layouts; tables before cards when people compare.
- Controls scale with content: no search, filter or sort over items that fit on one or two screens. No "See all" under a complete list.
- Illustration only when it explains; at most one decorative piece per page, first to cut. Decoration never encodes meaning.
- **Copy the look, not the page:** a style reference decides how included things look, never what is included.

### 4.3 Defensive (design only)

Defend against inputs that **really happen**, nothing hypothetical:
- Every state of every component the design touches: default, hover, focus, active, disabled, loading, empty, error, success. A disabled control still shows its value: a disabled switch that is on stays visibly on (dimmed accent, knob in place), never greyed until it looks off.
- Longest real text, translations, an empty list, a very long list.
- 320px wide, 200% zoom, safe areas, the on-screen keyboard, slow loading.
- Every screen that uses a component or token you change.

Between two options of equal size, pick the one that holds up. Anything beyond real inputs ("what if we add X later") is speculation and is cut.

### 4.4 Motion: planned, purposeful, motion-ready

Every design ships with a motion plan, and its components are built so that plan (or a later one) can be added without restructuring.

- **Purpose test:** each motion has a job: feedback (press, success), continuity (where something came from or went), orientation (what changed), or attention to one thing. Motion that only decorates fails the element test.
- **Tokens:** two or three durations (for example 140ms press, 200–260ms enter, 300ms layout) and one or two easings, named once and reused. No one-off timings.
- **Motion-ready structure:** every state is an explicit, named state; elements that move keep a stable box (no layout jump); animate only `transform` and `opacity` unless layout animation is the point; enter and exit both defined; nothing depends on motion to be understood.
- **Reduced motion:** a defined fallback (instant or fade) for every motion.
- **Delivery:** in code, implement the plan. In static mockups, add a short motion table (trigger → element → change → duration/easing → purpose) next to the screens, and name the state each screen shows.
- When designing motion in detail, load the `emil-design-eng` skill if it is available.

### 4.5 Pragmatic: the ladder

Stop at the first rung that fully works:
1. **Does it need to exist?** Skip what nobody asked for; name it in one line. A vague request gets the smallest version that does the core job.
2. **Does the product or design system already have it?** Use it the way nearby screens do.
3. **Does the browser or OS already do it?** `<dialog>`, `<details>`, `<select>`, `type="date"`, a plain `<table>`, the system font.
4. **Does an installed library already have it?** Never add a library, font or icon pack for one element.
5. **Can it be one rule?** One token or one property.
6. **Otherwise:** the minimum new design that works.

Also:
- **Reuse threshold:** extract a shared component or token on the **second real use**, never for one. No wrapper that only renames another component.
- **Fix at the root:** change the shared component or token once, not each screen.
- **Clever is not small:** a labelled button beats an icon nobody understands.
- **Mark shortcuts:** `shortcut: <the limit>, <when to upgrade>`. A marked shortcut within its limit is not a finding.
- **Never fake a saving:** don't claim "removed 12 elements" against a bigger design that was never made. Only the Lean count from an evaluation is real.

### 4.6 Structure is visible

If the content has groups or levels, the layout shows them, so users can see where a group ends and what belongs to which heading without reading every label.

- **Count the levels:** the screen shows as many levels as the content has (category › group › item). A flat list over grouped content is a finding.
- **At least two cues per level, cheapest first:**
  1. Proximity: the gap between groups is at least twice the gap between items in a group.
  2. Alignment: children are indented 12–16px under their heading, or offset by an icon or marker column.
  3. Heading contrast: a heading differs from its items in at least two of size, weight, case and colour, never colour alone.
  4. Common region (a box, fill or rule): only when the first three are not enough.
- **Label-removal test:** hide the headings. If the groups still read as groups, the structure is visible.
- **Dependencies show:** a setting that only matters for some value of another sits directly under it, indented. When it doesn't apply, it is disabled with a short reason, or hidden. A text tag ("vrex only") is not enough on its own.
- **Related items sit together:** order by what they affect; past about 5–7 items, split into named subgroups.
- **Keep depth shallow:** at most two indent levels in one column; deeper content gets its own view.
- **The markup has the same structure:** headings, or `role="group"` with `aria-labelledby`, so screen readers get the same groups.
- **References:** when copying a reference's look, also copy how it shows structure (its indents, gaps, icon columns, label styles), or replace each cue with an equal one.

---

## 5. The checklist (used by section 1 and section 3)

1. **Task:** can each task be completed? Is the result (price, total, status, confirmation) the most visible thing, shown once?
2. **Gate:** section 4.1 passes.
3. **Lean:** element test on every element; one main action per view; nothing said twice, states included (no readout next to a control that already shows its state); controls scale with content.
4. **States:** all states in section 4.3 exist for what the design touches.
5. **Real content:** longest text, empty, very long list, 320px, 200% zoom. Nothing hidden off-screen without a cue.
6. **Consistency:** same action, same look and label on every screen; no drift (two blues, three button styles, mixed radii, one-off spacing).
7. **Reuse:** repeated patterns are one component; tokens instead of hard-coded one-offs.
8. **Motion:** plan present; each motion passes the purpose test; tokens used; motion-ready structure; reduced-motion fallback.
9. **Ladder:** nothing built that a higher rung already provides.
10. **Taste:** matches the applicable preferences; not a reskin of a rejected option; no repeat of a logged negative for this category.
11. **Structure:** levels shown match levels in the content; two cues per level; label-removal test passes; dependencies nested and disabled when they don't apply; every removed container's grouping replaced (section 4.6).

---

## 6. When asked for alternatives

- Alternatives differ in **structure** (components, layout, interaction model), not only colour, type or radius. A reskin counts as the same option.
- Every alternative still passes the gate and the base rules; variety never means adding clutter.
- Name the axis each explores and its main trade-off, in the reply, not on the artboard.
- After a pick, the winners' shared traits are positive evidence and the losers' are weak negative evidence. Log both.
- Once a direction is approved, keep its approved decisions unless the user reverses them.

---

## 7. Confidence

| Level | Means | Use |
|---|---|---|
| Low | One signal, or an ambiguous one | Offered as an option only |
| Medium | Several consistent signals across designs | Applied by default, named in one line |
| High | Repeated **and** explicitly confirmed | A rule in `rules.md` |

One criticism is a provisional signal, never a permanent rule. Ambiguous statements never become rules silently. Historical feedback is never discarded when a preference changes.

---

## 8. Seed evidence (migrate on first use)

From the session that created this skill: an HCIN311 course redesign of the Apple PH "Buy iPhone 18 Pro" configurator, phone mockups, many directions.

```
### F-20261117-1
- Original: "You basically just copied the rest"
- Design ref: style row A/B/C (apple-zen, Swiss, Night): same layout, new colours
- Category: e-commerce, mobile   - Element: alternatives as reskins
- Signal: negative   - Kind: what-goes-on-the-page
- Interpretation: alternatives must differ in structure, not skin
- Confidence: Medium (repeated as "different UI ... as usual")   - Status: Pending

### F-20261117-2
- Original: "For the other redesigns, don't follow the non-use of rounded corners"
- Element: square corners in the Swiss style
- Signal: negative   - Kind: how-it-looks-and-moves
- Interpretation: prefers rounded geometry in consumer shop UI
- Confidence: Low   - Related: F-20261117-3   - Status: Pending

### F-20261117-3
- Original: "They don't have rounded corners."
- Element: apple-zen screen as rendered
- Signal: negative   - Kind: bug (renderer dropped inline border-radius)
- Interpretation: a tool fact, not extra evidence for F-20261117-2
- Status: Confirmed (bug)

### F-20261117-4
- Original: "This time, we're going for expensive look so users can feel that the website is good."
- Category: e-commerce   - Signal: neutral (brief)   - Kind: context
- Interpretation: premium feel as a trust signal for shop pages. Under the base rules it should come from type, spacing, material and motion, not from added elements.
- Confidence: Low   - Status: Contextual

### F-20261118-1
- Original: "Only the Luxe E Noir and Luxe F were somewhat decent."
- Design ref: Noir (black + champagne, Manrope light numerals, swipeable storage cards, floating ticket) and Showroom (product stage, white bottom sheet, 4-stop storage track, large price readout). Rejected: Atelier (ivory + serif order sheet), Zen, Swiss, Night, Matrix (table), Concierge (chat), Bento (tiles).
- Signal: weak positive for Noir and Showroom; weak negative for the rest
- Kind: how-it-looks-and-moves (mostly)
- Interpretation (hypotheses): the price as the hero; dark or high-contrast premium palettes; large light numerals; materials and depth over ornament. Possibly dislikes serif luxury, chat UIs and bento grids for this task. Base-rule conflicts: Noir's swipe row hides options off-screen; the floating ticket and the stage are added elements that need a job. The Matrix rejection does not override "tables before cards when people compare": it was one weak signal about a whole design, not about tables.
- Confidence: Low   - Status: Pending

### F-20261118-2
- Original (summary of the user's stated goals): no unnecessary things, no overloaded information, simple, a strong base for animation, consistent, reusable components, avoid over-engineering, pragmatism, defensive engineering. Motion should be included, or components built for the planned animation. Defensive applies to design only.
- Kind: what-goes-on-the-page
- Status: Confirmed (encoded in "What every design must be" and section 4)
```

Open these provisional preferences from the seed:
- P-1 Structural variety in alternatives: Medium (F-20261117-1).
- P-2 Rounded geometry in consumer UI: Low (F-20261117-2).
- P-3 Price as the hero, large light numerals: Low (F-20261118-1).
- P-4 Dark or high-contrast premium palettes for shop pages: Low (F-20261118-1, F-20261117-4).

Tool fact for `tool-facts.md`: the Claude Design canvas once dropped inline `border-radius`; mirroring radii into a stylesheet class rule with `!important` worked around it.