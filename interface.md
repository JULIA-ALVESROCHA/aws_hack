# Field Notebook Chatbot — Design & Build Spec

A project-based chatbot that pushes curiosity and critical thinking by assigning
unconventional hobbies. The interface is a physical notebook: left page holds the
4-step project, right page holds the conversation, and the archive is a stack of
past notebooks you can flip back into.

---

## Part 1 — Reading the reference image (StrongMe)

### 1.1 Anatomy

**Top bar (56px, white, 1px bottom hairline)**
- Left: small circular mark + `StrongMe` wordmark, bold, near-black
- Center-left nav: `Home` · `Pricing` · `Blog` — regular weight, ~14px, grey
- Right cluster: globe icon + `English` + chevron · sun/theme toggle · avatar + `Dong` + chevron

**Left rail (~72px, transparent over the app background)**
- A vertical stack of dates, newest at top
- The active entry (`今天 / 27 / 7月`) is a white rounded card (~10px radius) with a hairline border and a whisper of shadow — it is the *only* elevated thing in the rail
- Inactive entries are bare text: day number large-ish, month small, both muted grey
- A year label (`2024`) breaks the list into sections
- This rail is the time machine. It is not a menu.

**The spread**
- One rounded rectangle (~16px) holding two pages, wrapped in a soft cyan glow — like a lightbox around the book
- Pages are cream/off-white, separated by a visible gutter
- Three black binding clips sit *on* the gutter at ~25% / 50% / 75% height. They read as ⊂⊃ staples. This is the single detail that sells "book" — everything else could be any card layout.

**Left page**
- Header row: yellow sun glyph · `Tuesday` (bold, 13px) over `Jul 29, 2025` (grey, 12px) · far right a 28px square button with a sliders icon
- Body: a white card, generous padding, containing a flat illustration with rounded corners
- Caption under the card: small, grey, centered, one line
- Streak block at the bottom: `Mon…Sun` labels in tiny grey, date numbers beneath, today is a filled blue circle with white text
- Then `1 Day Streak` bold centered, and an amber encouragement line

**Right page**
- A soft grey card at top: tiny icon + `Savor the Moment` on the left, `10:00 PM` on the right; body is a centered bold sentence at ~17px with generous leading; footer row of three outline icons (comment, heart, share)
- Below the card: italic grey instruction text with a pencil emoji, then a second, lighter placeholder line
- Then ~6 ruled lines — hairline, warm grey, evenly spaced. These are the whole message: *you write here*
- Bottom: pin icon + `Where are you right now...` placeholder

### 1.2 What actually makes it work

| Device | Why it matters |
|---|---|
| Two pages, no tabs | Two modes visible at once. No state is hidden behind a click. |
| Binding clips | 40px of pixels doing all the metaphor work. Cheap, effective. |
| Paper warm / chrome cool | Page is cream, app shell is grey-white. The page feels like an object *on* a surface. |
| Cards resting on paper | Nesting depth: surface → page → card. Reads physical without skeuomorphic texture. |
| Ruled lines | The only affordance telling you the right page is an input. Better than a border. |
| Rail = archive, not nav | Going "back" is going back *in time*, not to a different screen. |

### 1.3 What to change for your product

StrongMe is a wellness journal — soft, feminine, gentle, pastel. Yours is about
poking at things until they make sense. Same skeleton, different material.

- Ruled lines → **dot grid**. Lab notebook, not diary.
- Illustration card → **specimen card**. The hobby arrives as something taped onto the page.
- Streak/flame → **progress through 4 steps**. Streaks reward showing up; you want to reward finishing.
- Pastel blue glow → drop it. Use a **physical red thread bookmark** instead.
- Chinese caption → your captions are the hobby's odd fact of the day.

---

## Part 2 — Design direction

### 2.1 Concept

**A field notebook kept by an amateur.**

Not "amateur" as unskilled — *amator*, one who does it for love. The person using
this is a naturalist, a tinkerer, a person who noticed something strange and
decided to chase it. The visual world to borrow from: herbarium sheets, lab
notebooks, index cards, specimen labels, manila folders, engineering dot grid,
red cotton bookmark ribbon, wax seals.

Suggested product names: **Amateur** · **Marginalia** · **Curio** · **The Long Way**

### 2.2 Tokens

```css
:root {
  /* Surface */
  --shell:      #E4E0D6;  /* desk the notebook sits on */
  --paper:      #EFEAE0;  /* manila — cooler than cream, deliberately */
  --paper-lit:  #F7F4EC;  /* the active page */
  --grid:       #D6CEBE;  /* dot grid */
  --rule:       #DDD5C6;  /* dividers, card edges */

  /* Ink */
  --ink:        #22252B;  /* iron-gall: blue-black, never pure black */
  --ink-soft:   #6E7178;
  --ink-faint:  #A5A297;

  /* Accents — used with severe restraint */
  --thread:     #8C2F2A;  /* oxblood. bookmark, active step, wax seal. ONLY. */
  --signal:     #C9D93C;  /* highlighter chartreuse. "you are here" + progress. */
}
```

Two accents, each with exactly one job. Oxblood = position and achievement.
Chartreuse = progress and attention. If a third color appears, delete it.

**Do not** ship this on warm cream + terracotta. That palette is everywhere right
now and it will make the product look like every other AI-built app.

### 2.3 Type

| Role | Face | Notes |
|---|---|---|
| Display | **Newsreader** | Headings, the hobby name, the assistant's opening line. Optical sizing, literary. |
| Marginalia | **Newsreader Italic** | Prompts, instructions, the assistant's asides. Replaces StrongMe's italic sans. |
| Body / UI | **Public Sans** (fallback Inter) | Chat, buttons, forms. Neutral on purpose. |
| Labels | **JetBrains Mono**, 11px, `letter-spacing: .08em`, uppercase | Step numbers, timestamps, specimen labels, badge metadata. This is what makes it read as *lab notebook* rather than *diary*. |

No handwriting font anywhere except, optionally, the signature line on the badge.

Scale: 11 / 13 / 15 / 17 / 22 / 30 / 44. Body 15px at 1.65 leading.

### 2.4 Materials

- Paper texture: SVG `feTurbulence` noise at 2–3% opacity, or a 200×200 tiled PNG. Subtle enough that you doubt it's there.
- Dot grid: 4px dots at 24px pitch, `--grid`, on the right page only.
- Shadows: warm, never grey. `0 2px 12px rgba(60,45,25,.08)`.
- Radius: 14px pages, 8px cards, 4px chips.
- Binding: 3 clips on the gutter, `--ink`, exactly as in the reference. Don't redesign this; it works.

### 2.5 Signature element — the thread

A red cotton bookmark ribbon anchored at the top of the spine, hanging down the
right page and past the bottom edge of the book. It marks your last position.

- Opening the app: the thread is visible in the closed cover, hanging out the bottom
- Switching projects from the archive: the thread **slides** to the new page before the page turn resolves
- Finishing a project: the thread is pulled out and knotted around the wax seal on the badge

One idea, executed three times. That's the whole flourish budget.

### 2.6 Motion

- Book opens on first load: cover rotates on `transform-origin: left`, 600ms, `cubic-bezier(.2,.8,.2,1)`
- Page turn: 450ms, corner curl on hover (subtle — 12px lift at the corner)
- New assistant message: text fades in per-line, 40ms stagger, as if being written
- Everything respects `prefers-reduced-motion` — turns become cross-fades, no curl

---

## Part 3 — Screens

### 3.1 Cover (capa)

The closed notebook, centered on the desk surface.

- Manila board cover, embossed title in blind deboss (shadow only, no ink)
- A mono label plate: `FIELD NOTEBOOK · VOL. 01`
- The red thread hanging from the bottom edge
- One control: `Open` — or the whole cover is clickable
- Subtitle, one line, plain: *Pick up a hobby nobody asked you to have.*

This screen doubles as your marketing cover image. Export it at 1200×630.

### 3.2 Loading (tela de carregamento)

Do not use a spinner.

- The cover opens partway and holds, or a dot grid draws itself in from top-left
- One rotating line of mono text underneath, from a list of ~20 real oddities tied to the hobbies in your catalog — e.g. `Lichen is two organisms pretending to be one.`
- If loading exceeds 3s, add a second line: `Still here. Finding you something strange.`

### 3.3 Login

The inside front cover — a **bookplate**. This is the highest-leverage screen in the flow and it costs nothing extra.

```
┌──────────────────────┬──────────────────────┐
│                      │                      │
│   ┌──────────────┐   │  Sign in             │
│   │ EX LIBRIS    │   │                      │
│   │              │   │  [ Continue with     │
│   │ This journal │   │        Google      ] │
│   │ belongs to   │   │  [ Continue with     │
│   │              │   │        GitHub      ] │
│   │ ___________  │   │                      │
│   └──────────────┘   │  ── or ──            │
│                      │  email  [_________]  │
│                      │  [ Send link ]       │
└──────────────────────┴──────────────────────┘
```

After auth, the name is written onto the bookplate line and the page turns. Small
moment, disproportionate payoff.

Errors speak plainly: `That link expired. Send a new one.` — no apology, no vagueness.

### 3.4 Onboarding form

Requirement: discursive **and** interactive. Structure it as the first four pages
of the notebook, one question per spread. Left page shows the question, right page
is where you answer. Progress is shown by the thickness of pages turned — a thin
stack indicator at the fore-edge, not a progress bar.

| Page | Type | Question | Input |
|---|---|---|---|
| 1 | Discursive | Describe one thing you noticed recently that you couldn't explain. | Dot-grid textarea, ~4 lines, no character counter |
| 2 | Interactive | Which of these would you rather spend a Saturday on? | 9 image cards, pick 3, drag to rank |
| 3 | Interactive | How do you like to learn? | Two sliders: *read first ↔ break it first*, *alone ↔ with people* |
| 4 | Discursive | What's something you're bad at and don't mind being bad at? | Dot-grid textarea |
| 5 | Interactive | What can you actually commit? | Chips: 20min/day · a few evenings · one long weekend + budget chips: free · under R$50 · under R$200 |

The two discursive answers are the real signal — feed them verbatim into the
recommendation prompt. The interactive ones are filters.

**Copy rule:** questions in Newsreader display, helper text in Newsreader italic,
inputs in Public Sans. Never explain the form. Just ask.

### 3.5 Hobby reveal

The assistant proposes a hobby + project. It arrives as a **specimen card**: a
white card with a mono label header, taped to the left page at a 1.5° rotation with
two translucent tape corners.

```
┌─────────────────────────────────┐
│ SPECIMEN 001 · EST. 3 WEEKS     │  ← mono label, --ink-faint
├─────────────────────────────────┤
│                                 │
│  Cyanotype printing             │  ← Newsreader, 30px
│                                 │
│  Make photographs with iron      │
│  salts and sunlight — no camera │
│  required.                      │
│                                 │
│  PROJECT                        │
│  A 12-print index of every      │
│  plant on your street.          │
│                                 │
├─────────────────────────────────┤
│  [ Start this ]   [ Something   │
│                     else ]      │
└─────────────────────────────────┘
```

`Something else` reshuffles, but the assistant asks *why* before reshuffling —
that answer is training data for the next suggestion, and it's the first
critical-thinking beat of the product.

### 3.6 Main spread — the workspace

This is the screen everything else exists to reach.

**Left page — the protocol**

- Header: hobby name (Newsreader, 22px) + project title (Public Sans, 13px, `--ink-soft`)
- Four step cards, stacked, 12px gap:
  - Mono step number in the top-left corner: `STEP 02`
  - Title, one line, 15px medium
  - Goal, one line, 13px `--ink-soft`
  - State: **done** = chartreuse checkmark and 60% opacity · **active** = oxblood left border, 3px · **locked** = 40% opacity, no border
- Clicking a step expands it in place (accordion). Expanded content:
  - What you'll need
  - What to do (3–5 bullets)
  - **The question** — a critical-thinking prompt in Newsreader italic, oxblood left rule. This is the point of the product. Give it visual weight.
  - `Mark done` button
- Clicking a step also **re-anchors the conversation** on the right page to that step. Show which step the chat is on with a mono breadcrumb at the top of the right page.
- Bottom of the page: four small squares, one per step, filling with chartreuse. Plus mono text: `2 OF 4`. No flames, no streaks.

**Right page — the conversation**

Do not use chat bubbles. Bubbles fight the paper.

- **Assistant messages**: plain text set directly on the dot grid, in Newsreader, 15px. A small oxblood dot in the left margin marks the start of each turn.
- **Your messages**: indented 32px, Public Sans, `--ink-soft`, with a thin left rule in `--rule`. Reads like your own handwriting in the margin against printed text.
- Timestamps: mono, 10px, `--ink-faint`, right-aligned, only on the first message of a session.
- Input at the bottom of the page: no box. A single hairline with a blinking caret above it. Placeholder in Newsreader italic: *ask, argue, or wonder…*
- Above the input, four quick chips (they teach the interaction model):
  `Why does this work?` · `Challenge me` · `I'm stuck` · `Show me an example`
- Streaming: text writes in line by line.

**Navigation**

- **Horizontal** = spreads within the project: `Protocol + Chat` → `Notes` → `Sources` → `Badge`
- **Vertical** = the left rail, listing past projects newest-first, grouped by year, exactly like the reference. Clicking one turns the whole book to **the last spread you were on** in that project. The thread slides there first.
- Corner curl on the outer bottom corners for next/previous
- Keyboard: `←`/`→` turn pages, `Tab` reaches every step and message, `Esc` closes an expanded step

**Mobile (< 768px)**

The spread collapses to one page. A two-item segmented control at the top:
`Protocol` / `Talk`. Swipe left/right turns pages. The thread stays, anchored to
the top-right corner. Everything else is identical — do not build a separate design.

### 3.7 The badge

The last spread of a completed project. Left page: a summary of what you made and
the four questions you answered. Right page: the badge.

- Rendered as a **wax seal** in `--thread`, with the project's initial pressed into it, the thread knotted around it
- Around the seal, mono metadata: hobby · project · completion date · `4 / 4`
- Below: three skills demonstrated, written as claims not buzzwords —
  `Ran a controlled test and changed your mind based on it` beats `Critical Thinking`

**Sharing**

1. **Image export** — render the badge spread to a 1200×630 PNG client-side (`html-to-image` or a server-side `satori` route). This is what actually gets posted.
2. **LinkedIn certification** — deep link straight into the Add Credential form:

```
https://www.linkedin.com/profile/add?startTask=CERTIFICATION_NAME
  &name=Cyanotype%20Printing%20—%20Street%20Flora%20Index
  &organizationName=Amateur
  &issueYear=2026&issueMonth=7
  &certUrl=https://yourapp.com/badge/abc123
  &certId=abc123
```

3. **Public badge page** at `/badge/[id]` — server-rendered with OpenGraph tags so
   the preview card renders in the feed. Make this page good; it's your funnel.

---

## Part 4 — Build notes

### 4.1 Stack

- **Next.js** (App Router) + **TypeScript** + **Tailwind**
- **Framer Motion** for page turns and the thread
- **Supabase** for auth (magic link + OAuth) and Postgres
- **Anthropic API** with streaming, via a route handler — never expose the key client-side
- `@fontsource` or `next/font` for Newsreader, Public Sans, JetBrains Mono

### 4.2 Routes

```
/                          cover
/login                     bookplate
/onboarding/[page]         form pages 1–5
/journal                   redirect to most recent project
/journal/[projectId]/[spread]   the book. spread ∈ protocol|notes|sources|badge
/badge/[badgeId]           public, OG-tagged
```

Putting the spread in the URL is what makes "go back to where I was" free — you
just store `last_spread` per project and route to it.

### 4.3 Schema sketch

```sql
users(id, email, display_name, created_at)

profiles(user_id, curiosity_note, avoidance_note, interests jsonb,
         learn_style jsonb, time_budget, money_budget)

projects(id, user_id, hobby_name, hobby_blurb, project_title,
         status, last_spread, created_at, completed_at)

steps(id, project_id, ordinal, title, goal, materials jsonb,
      instructions jsonb, question, completed_at)

messages(id, project_id, step_id nullable, role, content, created_at)

badges(id, project_id, slug, skills jsonb, issued_at)
```

`messages.step_id` is what lets the conversation re-anchor when you click a step —
filter on it to show that step's thread, or leave it null for general chat.

### 4.4 Assistant behavior

System prompt should enforce:

- **Ask before answering.** On any "why" question, first response is a probe, not
  an explanation. Explain on the second ask.
- **Never do the step for them.** If they ask for the answer, give the next
  smallest thing they could try.
- **One question per turn.** Two questions is an interrogation.
- **Make them predict.** Before any test in a step: *what do you think will happen?*
  After: *were you right? what does the difference tell you?*
- **Short.** 3–5 sentences default. The page is small and the point is that they write.
- Voice: a co-conspirator who is also curious, not a teacher who already knows.

**Hobby catalog** — seed ~40. Each needs a 4-step project that produces an
artifact. Starters: cyanotype printing · spore printing and mycology · lichen
mapping · bookbinding · ham radio and APRS · memory palaces · mudlarking · kite
aerial photography · medieval pigment making · soundscape recording · pinhole
photography · whittling · water clocks · tape loops · kombucha leather · moss
propagation · letterpress · orienteering · beekeeping without bees (bee hotels) ·
star charting by hand.

Store them structured — `hobby(name, blurb, materials, cost_band, time_band,
steps[4]{title, goal, instructions, question})` — and let the model select and
adapt rather than invent from nothing. More reliable, and it keeps step quality
consistent.

### 4.5 Quality floor

- Responsive to 375px
- Visible keyboard focus rings — use `--thread`, 2px, 2px offset
- `prefers-reduced-motion` honored on every animation
- Chat is a live region for screen readers; the dot grid is `aria-hidden`
- Contrast: `--ink` on `--paper` is ~14:1. Don't let `--ink-faint` carry any
  meaning that isn't repeated elsewhere.

---

## Part 5 — Build order

1. Tokens, fonts, and the paper/grid surfaces
2. Static book shell — two pages, gutter, clips, thread. No data.
3. Page-turn mechanics + routing + keyboard nav
4. Cover → loading → login → bookplate
5. Onboarding form, all five pages
6. Hobby catalog seed + recommendation call
7. Main spread: step cards left, streaming chat right, step re-anchoring
8. Archive rail + last-spread restore
9. Badge, PNG export, public page, LinkedIn deep link
10. Mobile collapse, reduced motion, focus states

Steps 1–3 are the whole risk. If the book shell doesn't feel like an object, the
rest doesn't matter. Build it empty and stare at it before adding anything.
