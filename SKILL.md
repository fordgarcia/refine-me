---
name: "refine-me"
description: "Judges designs and learns the user's taste. Use before generating any UI (Claude Design or Code), when the user critiques, compares or saves designs, wants a DESIGN.md, or on /refine-me context|analyze|compare|save|export|pack|prefs|rules."
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

Typed as `/refine-me <command> <arguments>`; the skill receives the rest of the line as its arguments. Each has plain-language equivalents. `/refine-me` with no recognised command runs `analyze` and says in one line which command it took the input for. Claude never adds or redefines a command on its own; a new command is a skill edit (section 9).

| Command | Plain-language equivalents | Does |
|---|---|---|
| `context [text]` | "here's the context", "this is for…", "the users are…" | Sets or updates this project's brief (section 0.2). With no text, shows it, each field marked Given or Assumed. |
| `analyze [target]` | "analyze this", "what's wrong with this", "refine based on what you've learned" | Full evaluation (section 3), including faults nobody asked about. Target: a design ID (`D-0007`), else the design in view, else the last one saved. Never redesigns by itself. |
| `compare A B` | "which is better", "compare with the previous version" | Section 3 on both (design IDs or designs in view), side by side, a verdict per dimension. No overall winner unless one wins all four. The user's pick is logged (section 6). |
| `save [file] [origin]` | "save this design", "here's my design", "here's a reference" | Saves the design to `designs/` with a new ID and `meta.md` (section 0.4). Saves only; no evaluation. |
| `export [project]` | "make a DESIGN.md", "I'll design this in Claude Design", "give me the design brief" | Writes this project's rules and taste as a `DESIGN.md` (section 8), so they apply where refine-me can't run. Never overwrites a `DESIGN.md` refine-me didn't make. |
| `pack [destination]` | "move my store to Claude Code", "back up refine-me" | Packs the store into the `.refine-me/` layout (`memory/` files, plus `designs/` when there are any) as files or a zip, ready to drop into a project or `~/.claude/refine-me/`. Moves the store; never changes it. |
| `prefs [filter]` | "what have you learned about my taste" | Preferences with confidence, frequency and evidence IDs, then the rules in force. Filter by category (e-commerce, dashboard…) or dimension (geometry, colour…). |
| `confirm P-n` / `reject P-n` | "yes, make that a rule", "no, that's wrong" | Confirm: High confidence, rule added to `rules.md`. Reject: kept in history, never applied. Both are logged. |
| `rules [file or text]` | "use my rules", "import this rule file" | No argument: show the active base rules and their source. With a file or text: import it as the base rules (section 0.3). |

**Runs on its own:** section 1 every time a design is generated (including saving it, section 0.4), and section 2 after every judgment. `analyze` runs only when asked, never after every change, because a full evaluation on each edit costs time and tokens.

---

## 0. The store (do this first, every time)

### 0.1 Where it lives

The store has two folders: **`designs/`** holds what gets judged (section 0.4); **`memory/`** holds only text, what was learned. Create both on first use if missing, without asking.

| Surface | Location |
|---|---|
| Claude Code, in a project | `.refine-me/designs/` and `.refine-me/memory/` at the repo root, plus `~/.claude/refine-me/memory/` for global taste. Read both `memory/` folders; the project wins on conflict. Designs are always per project. |
| Claude Code, no project | `~/.claude/refine-me/memory/`, and `~/.claude/refine-me/designs/<project-slug>/` |
| Chat with memory tools | memory file `/topics/refine-me.md`: one headed section per memory layer, one Context section per project. Memory is text only, so there is no `designs/`: a design is recorded as a `Design ref` with an artifact link or a short description. |
| None of the above | Keep the layers in the conversation; at the end, give the user the files to save. |

When creating `.refine-me/` inside a git repository, add `.refine-me/designs/` to `.gitignore` (screenshots are large, and client work or references must not be committed by accident) and say so in one line. `memory/` is plain text and safe to commit.

Memory layers, in `memory/`:

| Layer | File | Rule |
|---|---|---|
| Critique log | `critique-log.md` | **Append-only.** The user's words verbatim plus the interpretation. Never edited or deleted, even when a preference changes. |
| Preferences | `preferences.md` | Curated. Each entry links to its supporting log IDs. |
| Learned rules | `rules.md` | Only from confirmed (High) preferences. |
| Base rules | `rules-base.md` | Imported with `rules`. If absent, section 4 is the base rules. |
| Context | `context.md` | One per project. Facts, not taste. |
| Tool facts | `tool-facts.md` | Rendering or tool quirks learned the hard way, with their workaround. |
| Skill edits | `skill-edits.md` | While the skill is in development: every proposed edit to the skill with its status (section 9). Created on first use; one copy only (link to it from elsewhere, never duplicate it). |

A new store starts empty. If the user points to an existing store (another `.refine-me/`, an export of their chat memory), copy it into `memory/` before anything else.

### 0.2 Context fields

`context` parses free text into these fields and keeps the user's exact words in a `Raw` line under the table.

| Field | Example |
|---|---|
| Product and module | Apple PH store, Buy iPhone configurator |
| Users | first-time buyers on phones |
| Primary tasks and success | configure and see the exact price; compare sizes |
| Platform and size | phone 390px; desktop 1280px |
| Brand and must-keep | keep the product's colours, type and buttons |
| Icons | the set the user named ("use Phosphor for icons"), else the section 4.7 default |
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

### 0.4 The designs folder

One subfolder per design, `designs/D-<nnnn>/`, numbered per project and never reused:

- `index.html` (or the source file), when there is one.
- `shot.webp` (or `.png`): what was actually seen, because a judgment is about the render, not the source. Take it with a headless browser when one is available; otherwise ask for a screenshot only when the user judges that design. If there is none, write `Shot: none` in `meta.md`.
- `meta.md`:

```
- ID: D-0007
- Origin: claude-code | claude-design | user | reference
- Project: <project slug>
- Date: <yyyy-mm-dd>
- Title: <the name the user would use, e.g. "Noir">
- Source: <file path, artifact URL, canvas name, site URL>
- Parent: <D-id this revises, if any>
- Judged: <F-ids> | unjudged
```

How designs get in:
- **Claude Code outputs:** saved automatically every time a design is delivered (section 1, step 7).
- **Claude Design outputs:** saved when Claude can reach them (an export, a handoff file, or a link the user gives). Never assume the canvas can be read directly; if it can't, ask once for an export when the user judges the design.
- **From the user** (`save`, an attachment, or a file dropped into `designs/`): give it an ID and `meta.md`. If they didn't say whether it is their own design or a reference, record `Origin: reference (assumed)` and confirm in one line at the end of the reply. A reference decides how things look, never what goes on the page (section 4.2), and is never counted as Claude's output. If a reference can't be read (corrupt, blank, wrong file), say so at once and ask for a re-export; nothing is built against it until it arrives, and nothing (DESIGN.md, mockups, the rules) stands in for it.

Rules:
- **Saving is automatic; evaluating is not.** On save, only section 1's checklist runs, and only on Claude's own outputs. `analyze` and `compare` run on request and take design IDs.
- **An unjudged design is not evidence.** Preferences come only from log records. Each record's `Design ref` names its D-ids, and each design's `Judged` line lists its F-ids back.
- **Never delete a design**, judged or not; pruning is the user's call.

---

## 1. Before generating any design

1. Read `context.md`, and the project's `DESIGN.md` if one exists (section 8.4). If context is missing or thin, infer the gaps from the request and the conversation, save them as **Assumed**, and state them in one line before the design.
2. Read the base rules, `rules.md`, `tool-facts.md`, and the `preferences.md` entries for this interface category. Apply them by section 0.3.
3. Build with the ladder (section 4.5). Plan motion together with the components (section 4.4), not after.
4. Run **the checklist** (section 5) on the draft and fix what fails before showing it. Run item 11 (structure) on every subtraction: anything removed for leanness must not take a grouping cue with it.
5. Use methods known to render correctly (`tool-facts.md`). Do not add verification steps the tool forbids. If the user reports something that didn't render as written, log it as a bug and a tool fact, never as taste.
6. End the delivery with two lines:
   `Skipped: <left out on purpose, and the option if they want it>`
   `Not checked / risk: <e.g. real screen reader, translations, real data>`
7. Save the design to `designs/` (section 0.4) and name its ID in the delivery ("Saved as D-0007"). In chat, skip this step.

---

## 2. After every user judgment (the learning loop)

Judgment → log record → interpretation → pattern check → proposal → approval → update.

For every meaningful judgment (criticism, praise, a pick between options, a rejection, "the previous one was better", a repeated correction):

1. **Append a log record**, keeping the user's exact words, and add its F-id to the `Judged` line of each design it refers to.
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
- Design ref: <D-ids; in chat, an artifact link or short description>
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
4. **Reference cues:** when a reference is given, it is the ground truth: judge the design against it, not against the rules or mockups made from them. List how it shows structure (indents, gaps, icon columns, label styles, regions, named groups, control anatomy) and check that the design keeps or replaces each one. Where the reference and the rules disagree, the reference wins and the rule gets a Skill gap.
4b. **Run it, don't just look:** when the design runs (a headless browser, a test script), produce every state it touches (hover, focus, pressed, disabled, empty) and do each primary task once, shooting before and after; the result must land where the user looks. Judge states in pairs too (hover on one item while focus is on another, selected + focused). "Unavailable" means the tool can't produce it, not that nobody tried. A task or state judged only from still shots is marked **on spec, unverified**.
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
- A gate result or score for a design Claude hasn't seen rendered is marked **on spec, unverified**: it describes the values intended, not what renders.
- **Self-grading guard:** for Claude's own fresh output, list the three weakest points before the score, and never score above 85 without a user judgment backing it. Self-scores drift high. When the self-score decides something (ship, pick, stop), end with one line asking for a single judgment on the weakest screen; log the answer (section 2).
- **No false precision:** whole numbers only. The rubric is for designs; small choices (a heading, a label, one control) get a verdict with reasons, not weighted decimals.
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
| Skill gaps | While refine-me is in development: for each finding, the rule that allowed it or failed to catch it, and the edit that would catch it next time. Never left out: write `none found` if empty, and append each gap to `skill-edits.md` in the same reply |
| Found by | Findings this round caught by the skill vs. by the user (for example `skill 4 / user 1`). The trend is the measure of "fewer iterations" |

---

## 4. Base rules (default; replaceable with `rules`)

Adapted from Ponytail's "lazy senior designer": lazy about the solution, never about the task.

### 4.1 Never cut (fixed; no import overrides this)

- Text contrast at least 4.5:1 (3:1 at 24px+); control boundaries at least 3:1.
- Visible focus (indicator at least 3:1 against adjacent colours, WCAG 1.4.11) and full keyboard use; real buttons, links and labelled inputs. Every drag (reorder, move, resize) has a keyboard path, and the design shows it: pick, move, drop, and its focus state.
- Targets 44–48px on touch screens; at least 24×24px (WCAG 2.2, 2.5.8) on pointer-and-keyboard desktop UIs. Colour is never the only signal; icon-only controls get a word or label.
- Reduced motion respected.
- Errors say how to fix them; nothing makes users lose or retype work, or get stuck.
- Honest pricing, consent and labels; a way to cancel or undo; a warning before destructive actions; no dark patterns.
- Anything the user explicitly asked for. If that request fails another item here, it is not a taste call: say plainly which item it fails, offer a compliant version, and build theirs only after they confirm they know; record it under `Not checked / risk`.

### 4.2 Lean and simple

- **The element test:** keep an element only if it has a **job** (one sentence: what it helps the user do or decide; "looks nice" is not a job), does it **once** on the screen, and would be **missed** if deleted.
- One message per section, one main action per view; the primary style is used once per screen.
- **Say it once, and say what it is:** don't show the same fact as a bar, a number, a label and a button caption. Check every fact against every region of the same view (header, footer, sidebar row) and across forms (a heading that repeats the logo, a card title that repeats its button). Location: the navigation's selection shows the current page, so a title or breadcrumb repeats it only when the navigation is hidden. A bare number first takes the element test; if it stays, it gets a unit, a label or a position that makes it obvious ("1 pickiso 1" fails).
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
2. **Does the product or design system already have it?** Use it the way nearby screens do. This covers colour roles, logo and identity too: before choosing an accent, read which colours the product already gives to active, mode, system and warning states. If you can't see the rest of the product, ask for one screenshot of its main screen before designing.
3. **Does the browser or OS already do it?** `<dialog>`, `<details>`, `<select>`, `type="date"`, a plain `<table>`, the system font.
4. **Does an installed library already have it?** Never add a library, font or icon pack for one element (for one icon, copy that one SVG from the section 4.7 set).
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
  1. Proximity: the gap between groups is at least twice the gap between items in a group. A control is closer to its own label than to any other row: a fixed label column (about 180–220px) with controls left-aligned after it, or rows capped at about 640px; one label/control pattern per page; nested rows keep their parent's control column.
  2. Alignment: children are indented 12–16px under their heading, or offset by an icon or marker column.
  3. Heading contrast: a heading differs from its items in at least two of size, weight, case and colour, never colour alone.
  4. Common region (a box, fill or rule): only when the first three are not enough.
- **Label-removal test:** hide the headings. If the groups still read as groups, the structure is visible.
- **Dependencies show:** a setting that only matters for some value of another sits directly under it, indented. When it doesn't apply, it is disabled with a short reason, or hidden. A text tag ("vrex only") is not enough on its own.
- **Related items sit together:** order by what they affect; past about 5–7 items, split into named subgroups.
- **Keep depth shallow:** at most two indent levels in one column; deeper content gets its own view.
- **The markup has the same structure:** headings, or `role="group"` with `aria-labelledby`, so screen readers get the same groups.
- **References:** when copying a reference's look, also copy how it shows structure (its indents, gaps, icon columns, label styles), or replace each cue with an equal one.

### 4.7 Icons

**Which set**, the first that applies:
1. **The user named one** ("use Lucide for icons"): record it as the `Icons` field in this project's `context.md`, Given. It is a project fact, not a preference, and does not carry to other projects unless the user says it is their default everywhere (then it is a rule in global `rules.md`).
2. **The product already uses a set:** use it. If you can't see the product's icons, ask (or ask for a screenshot) before going to rung 3 or 4.
3. **The platform has a convention:** Nerd Font glyphs (devicons, codicons) for terminal and IDE interfaces; the platform's own symbols for native apps (Material Symbols on Android, SF Symbols on Apple platforms only).
4. **Otherwise Lucide** (ISC licence, official packages for most frameworks, plain SVG for static pages).

Rules:
- **One family per product.** Never mix sets. Outline and filled versions of one set are mixed only to show state (filled = active).
- **Match the type:** stroke close to the text's weight at that size; icon sizes and the stroke width are tokens (for example 16, 20, 24px, one stroke).
- **Never hand-draw an icon the set already has.** If the set lacks one, use the nearest meaning or a word. A drawn icon must match the set's grid, stroke and corners, and is marked `shortcut:`.
- **Icons support labels, they don't replace them.** Icon-only only for universal meanings (close, search, menu, back), always with `aria-label`; everything else gets a visible word. Decorative icons get `aria-hidden="true"`.
- **Licence:** record the set and its licence in `context.md`. MIT, ISC and Apache 2.0 sets are fine; CC BY sets (Font Awesome Free icons, Codicons) need attribution; read custom licences (for example the Remix Icon License) before using them in anything sold.
- **Delivery:** in code, the set's official package or its SVG files; in static mockups, inline SVG so nothing depends on the network.

### 4.8 Control anatomy

- **Pick the control by the values.** 2–5 named options: segmented or radio, all visible. Visual options (themes, layouts, shapes): previews of each option. A continuous range where only the rough amount matters (contrast, volume): a slider with end labels and no number. A numeric value with a unit people reason in (px, ms, %): a slider with an editable number field attached. A long list: a menu.
- **Sliders:** track at least 4px with a 3:1 boundary; the part from the minimum to the thumb filled with the accent; thumb at least 16px; hit area at least 24px tall. Tick marks for the snap steps or for every few units. The number field sits within 8–12px of the track's end, never across the row.
- **Disabled controls** keep their value visible (section 4.3) and dim as one unit, readout included.

---

## 5. The checklist (used by section 1 and section 3)

1. **Task:** can each task be completed? Is the result (price, total, status, confirmation) the most visible thing, shown once? When the design runs, do each task once (section 3.0, 4b).
2. **Gate:** section 4.1 passes, **measured, never estimated**. Click targets: from the code (hit rects, padding) and the render; list every one under the minimum. Contrast: sample the pixels of the lowest-contrast text and control edges and compute the ratio; report the worst pair, on every theme the product ships. A token or comment that claims a ratio ("3:1 on surface") is measured against every background it sits on; a wrong claim is a Must fix on the token, fixed at the root.
3. **Lean:** element test on every element; one main action per view; nothing said twice, states included (no readout next to a control that already shows its state); controls scale with content.
4. **States:** all states in section 4.3 exist for what the design touches, and read correctly in pairs: two items that look lit at once, or a pair that reads as one state, is a finding.
5. **Real content:** longest text, empty, very long list, 320px, 200% zoom. Nothing hidden off-screen without a cue.
6. **Consistency:** same action, same look and label on every screen; no drift (two blues, three button styles, mixed radii, one-off spacing, mixed icon sets or strokes; section 4.7). List each widget kind on two or more screens (scrollbars, selection and hover fills, focus rings, close buttons, toggles) and compare them side by side: one kind, one look. If the context says the product ships several visual languages (rices, skins, platforms; Given only), check look within each, and across them only words, steps, order and keys.
7. **Reuse:** repeated patterns are one component; tokens instead of hard-coded one-offs.
8. **Motion:** plan present; each motion passes the purpose test; tokens used; motion-ready structure; reduced-motion fallback.
9. **Ladder:** nothing built that a higher rung already provides.
10. **Taste:** matches the applicable preferences; not a reskin of a rejected option; no repeat of a logged negative for this category.
11. **Structure:** levels shown match levels in the content; two cues per level; label-removal test passes; dependencies nested and disabled when they don't apply; every removed container's grouping replaced; each control closer to its own label than to anything else; one label/control pattern per page (section 4.6).
12. **Controls:** each control fits its values; slider anatomy and hit areas pass (section 4.8); location shown once; every number says what it counts.

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

## 8. Export: DESIGN.md

`export` turns one project's rules and taste into a `DESIGN.md` file following Google Labs' open DESIGN.md specification (version `alpha`; check github.com/google-labs-code/design.md, `docs/spec.md`, before relying on details, because the format is still changing). Its use: carrying refine-me into places where the skill can't run, mainly Claude Design, and any agent that reads DESIGN.md.

### 8.1 What goes in

One file per project, never global: preferences are scoped by interface category (section 0.3).

| Source | Goes in as |
|---|---|
| Real values: `context.md`, the project's CSS or tokens, a brand guide | Front-matter tokens (normative) |
| The reference, when there is one | Its concrete structure in the matching sections (named groups and what's in them, control anatomy, where help goes) plus its path; DESIGN.md never stands in for it |
| Confirmed rules and High preferences for this category | Plain rules in the matching section |
| Medium preferences | Rules marked `(P-n, default)` |
| Low preferences | `Option, not a rule: ... (P-n)`, never phrased as a rule |
| Logged negatives and rejected preferences for this category | Don'ts |
| Base rules (section 4) | A compact summary: gate, lean, structure, ladder; not the full text |

- **Never invent tokens.** Taste such as "dark, high-contrast premium" stays prose until the user or the project gives real values. The spec's only hard requirement is a defined primary colour: if the project has none, ask for it in one line. If none comes, export without it, list the empty sections in `omitted`, and warn that the file won't pass the spec's linter.
- **Contextual entries** (a preference the brief blocks for this project) stay out.

### 8.2 Structure

Front matter: `version: alpha`, `name`, `description`, then only the token groups that have real values: `colors`, `typography` (`fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`), `rounded`, `spacing`, `components` (only `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`; reference tokens as `{colors.primary}`). Hex colours; dimensions in `px`, `em` or `rem`. No duplicate keys or headings: the spec rejects the file.

Body, `##` headings in this order, leaving out empty ones:

1. **Overview**: product, users, main tasks, tone (from `context.md`), and "What every design must be" in five lines.
2. **Colors**, 3. **Typography**, 4. **Layout**, 5. **Elevation & Depth**, 6. **Shapes**, 7. **Components**: what the tokens are for, plus the rules and preferences for that dimension.
8. **Do's and Don'ts**: Do = confirmed rules and the base-rule summary; Don't = the negatives, each one concrete ("alternatives that only change colours").

Then three sections the spec doesn't define. It keeps unknown sections, so they go last:
- **Motion**: duration and easing tokens, the purpose test, reduced-motion fallback (section 4.4). The spec has no motion tokens, so these stay prose.
- **Iconography**: the set, its licence and the rules (section 4.7).
- **refine-me**: generated date, store version, the evidence IDs used, and two lines for the reader: "Designs made from this file: bring them back with `/refine-me save` so the skill keeps learning." and "Hand edits to this file are read back as judgments."

### 8.3 Where it goes

| Surface | Output |
|---|---|
| Claude Code, in a project | `DESIGN.md` at the project root; a copy in `memory/exports/DESIGN-<yyyymmdd>.md` |
| Chat | Sent as a file; the copy goes in the memory file as a dated note |
| For Claude Design | Hand the user the file to upload or paste into their brief. Don't claim Claude Design reads it automatically unless `tool-facts.md` says so. |

### 8.4 Reading DESIGN.md back

- **A `DESIGN.md` refine-me didn't make** (no `## refine-me` section) is the project's own design system: read its tokens into `context.md` as Given brand values (Brand and must-keep). Never overwrite it; export to `.refine-me/DESIGN.refine-me.md` instead, and offer to merge only when the user asks.
- **A `DESIGN.md` refine-me made, then hand-edited:** before the next export or design, compare it with the copy in `memory/exports/`. Each change is a judgment: log it (section 2) with the old and new text as `Original`. A changed token is a context fact (Given); a changed or deleted rule is a candidate preference (Pending). Never silently undo a hand edit.
- **Stale file:** export again after a preference is confirmed or rejected, or when the user asks. Mention in one line when the file is older than the store's latest change.

---

## 9. Growing the skill (while in development)

Every use is also a test of the skill. The `Skill gaps` output row (section 3.5) proposes edits; this section keeps them from bloating or contradicting the skill.

- **Existing rule first.** For each gap, name the rule that should have caught it. If one exists, sharpen it or add an example to it. Propose a new rule only when nothing covers the case. Keep SKILL.md under about 500 lines.
- **Log every proposal** in `skill-edits.md`: `E-<yyyymmdd>-<n>`, the proposed text, the finding that caused it, status `Proposed | Accepted (version) | Rejected`. **Never re-propose a Rejected edit** unless the user asks.
- **One base.** Before proposing, read the currently saved skill. If several sessions propose edits at once, merge them into one proposal against the latest saved version instead of stacking cards.
- **Commands are skill edits.** Never add, rename or redefine a command inside a session; propose it.