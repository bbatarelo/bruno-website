# Bruno Batarelo — Website Build Context

This document captures all design decisions, current state, and conventions so work can resume seamlessly after any restart.

---

## Project

**Type:** Jekyll static site  
**Theme:** [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) (`minimal-mistakes-jekyll` gem)  
**Skin:** `default` (set in `assets/css/main.scss`)  
**Hosted at:** `https://bruno.batarelo.net` (GitHub Pages or similar, CNAME set)  
**Ruby version:** see `.ruby-version`  
**Dev server:** `bundle exec jekyll serve` → `localhost:4000`

---

## Site Owner Identity

Bruno Batarelo. Senior backend/software engineer based in Croatia.

**Core positioning (use this as the source of truth for all copy):**
- Senior backend engineer working with Scala, distributed systems, and Rust-oriented systems software
- Transitioning toward Rust as a deliberate next step — not a trend
- Broader engineering background: embedded systems, motorcycle suspension, CNC machining, electronics, audio/MIDI tools, music (guitar, band)
- Dislikes resume-clone personal sites; the site should show full range while keeping the resume focused

**Hero excerpt (homepage):**
> Senior backend engineer working with Scala, distributed systems, and Rust-oriented systems software.

**Hero sub-line:**
> I build reliable software and practical engineering systems — across software, embedded, mechanical, and audio domains.

**About paragraphs (homepage and /about/):**
> I am a senior software engineer with a background in Scala, distributed backend systems, production architecture, and engineering leadership. I am currently focusing on Rust as a natural next step toward reliable, explicit, performance-conscious systems programming.
>
> Outside pure software, I work on practical engineering projects: motorcycle suspension servicing and tuning, CNC machining, electronics, and audio/MIDI tools. I like systems that must work in the real world, whether that means backend services, hardware interfaces, or mechanical components under load.

---

## Visual Design Decisions

| Decision | Choice | Reason |
|---|---|---|
| Homepage layout | `splash` (no sidebar) | Avoids resume-template feel; full-width content |
| Content page layout | `single`, no sidebar | `author_profile: false` globally; sidebar caused name repetition and felt like resume |
| Author sidebar | Disabled globally | Name appeared 3× on homepage (nav, hero, sidebar) — too much |
| Hero | Black overlay (`overlay_color: "#000"`, `overlay_filter: "0.5"`) | Clean, confident; no hero image yet (placeholder path in config) |
| Hero buttons | `btn--light-outline` (via `header.actions`) | Two CTA: Resume + Projects |
| Area cards | No `image_path` (images not yet available) | Broken image icons hurt the page; removed until real images exist |
| Card buttons | `btn--inverse` (outline style) | Cleaner than filled gray buttons |
| Section labels | Uppercase, muted (`color: #888`) | `ABOUT`, `AREAS` as quiet section markers |
| Intro text alignment | Left-aligned, full container width | Centered text felt like a business card; no `max-width` constraint |
| Nav length | 6 items | `Career / Projects / Workshop / Music / Notes / Contact` |
| "Mechanical" renamed | "Workshop" | Feels more natural; covers machining, suspension, electronics, audio tools |

---

## File Structure (current state)

```
_config.yml                 # Site config, author block, plugin list, defaults
_data/
  navigation.yml            # Main nav: Career/Projects/Workshop/Music/Notes/Contact
_pages/
  about.md                  # /about/ — personal intro, who I am
  career.md                 # /career/ — full career narrative, TOC
  resume.md                 # /resume/ — focused Rust-transition resume (skeleton)
  projects.md               # /projects/ — software projects
  projects/
    motorcycle-traction-control-platform.md   # /projects/motorcycle-traction-control-platform/
  mechanical.md             # /workshop/ — suspension, machining, electronics, audio/MIDI
  music.md                  # /music/ — band and musical background
  notes.md                  # /notes/ — blog/notes index (home layout)
  contact.md                # /contact/ — email, GitHub, LinkedIn
index.html                  # Homepage (splash layout)
assets/css/main.scss        # Custom CSS on top of Minimal Mistakes
assets/images/projects/motorcycle-tc-platform/   # Scope captures for the TC platform write-up
Gemfile                     # Ruby deps
.ruby-version               # Ruby version pin
CNAME                       # bruno.batarelo.net
```

---

## _config.yml Key Settings

```yaml
title: "Bruno Batarelo"
theme: minimal-mistakes-jekyll
minimal_mistakes_skin: "default"

author:
  name: "Bruno Batarelo"
  bio: "Senior Scala / backend / distributed-systems engineer transitioning toward Rust, ..."
  location: "Croatia"
  links:
    - GitHub: https://github.com/bbatarelo
    - LinkedIn: https://www.linkedin.com/in/bruno-batarelo-271a869/
    - Email: mailto:bruno@batarelo.net

defaults:
  _pages: layout: single, author_profile: false   # ← sidebar disabled globally
  _posts: layout: single, author_profile: true    # ← posts still get sidebar
```

---

## Homepage Structure (`index.html`)

```
layout: splash
author_profile: false
excerpt: "Senior backend engineer working with Scala..."   ← renders in hero below name
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg   ← doesn't exist yet, overlay_color takes over
  actions:
    - Resume → /resume/
    - Projects → /projects/

--- (front matter end)

<section class="home-subhero">   ← italic sub-line
<section class="home-about">     ← left-aligned "ABOUT" heading + two paragraphs
<section class="home-areas">     ← "AREAS" heading (immediately above feature_row)

{% include feature_row %}        ← three cards: Software Systems / Workshop / Audio & Music

<section class="home-contact">   ← Croatia · Email · GitHub · LinkedIn · Resume
```

**Key lesson:** In Minimal Mistakes splash layout, `page.excerpt` (top-level front matter) renders in the hero, NOT `header.excerpt`. Always put the hero text at the top level.

---

## CSS (`assets/css/main.scss`)

Custom classes added after `@import "minimal-mistakes"`:

- `.home-subhero` — italic sub-line below the hero, separated by a bottom border
- `.home-about` — left-aligned About section with muted uppercase heading
- `.home-areas` — muted uppercase "AREAS" heading immediately above the feature cards
- `.home-contact` — horizontal contact strip with top border

No `max-width` constraints on text — intentionally uses the full splash layout container width to match the feature_row cards below.

---

## GitHub Account

The correct account is **`bbatarelo`** (`git@github.com:bbatarelo/bruno-website.git`).
An earlier draft of the site linked to `brunobt`, which is a different, near-empty
account — corrected site-wide on 2026-08-24.

---

## Navigation

```yaml
Career       → /career/
Projects     → /projects/
Workshop     → /workshop/   (page: _pages/mechanical.md, permalink: /workshop/)
Music        → /music/
Notes        → /notes/
Contact      → /contact/
```

Note: `/about/` exists as a page but is not in the main nav (covered by homepage content). Can be added back if needed.

---

## Earplay — Career Fact and Planned Subpage

**Reactive Studios, later Earplay.** Bruno is cofounder, co-owner and CTO.

Key facts, confirmed by owner 2026-08-25:

- Founded 2013. Built an iOS/Android interactive-audio title, then a B2B
  authoring and publishing platform for voice assistants (Alexa, Google Home),
  on a Scala/Play backend.
- **It ran in parallel with every consulting/contracting gig.** Throughout MLG,
  McKinsey, Moneyfarm, Flaminem and Blyott, Earplay was operational and Bruno
  was actively participating in it. The overlapping date ranges on `/resume/` are
  correct and intentional — not an error to be "cleaned up" later.
- This also explains the Jun 2020 – Sep 2021 window between McKinsey and
  Flaminem, which is not an employment gap.
- Last customers were offboarded in **Q4 2025**. The company still formally
  exists and Bruno is still formally CTO, but it is no longer trading.

**Planned:** Earplay deserves its own subpage on the site — a decade of running a
real product for real customers is under-served by a resume entry. Not started;
to be picked up in a later session. Likely shape: a project/company page under
`/projects/` or its own top-level entry, similar in treatment to
`/projects/motorcycle-traction-control-platform/`.

---

## What's Placeholder / Not Yet Done

| Item | Status |
|---|---|
| Hero image | `/assets/images/header.jpg` — does not exist; overlay_color covers it |
| Feature card images | No `image_path` set — cards render text-only |
| Resume content | ✅ Rebuilt 2026-08-25 from the 2025 PDF; `/career/` rewritten as six eras |
| Projects content | "Past & Ongoing" has three entries: E-MU Tracker Pre driver, onset-matcher, Motorcycle Traction-Control Research Platform |
| Bio photo | Not set; would go in `_config.yml` under `author.avatar` |
| LinkedIn URL | ✅ `https://www.linkedin.com/in/bruno-batarelo-271a869/` — confirmed by owner 2026-08-25 |
| Notes/blog posts | `_posts/` directory does not yet exist; `/notes/` will auto-list posts once created |
| Earplay subpage | Requested by owner, not started — see "Earplay" section above |

---

## Decisions Still Open

- Whether to add `/about/` back to the nav
- Whether to add a "Selected Projects" section to the homepage once projects are documented
- Skin — currently `default` (light); the `dark` skin was tried but reverted. Can switch anytime via `minimal_mistakes_skin` in `_config.yml`
- Whether to add social icons to the contact strip on the homepage

---

## How to Continue

1. Run `bundle exec jekyll serve` from the project root
2. Browse `localhost:4000` to see live state
3. All content pages are in `_pages/` — edit Markdown directly
4. Homepage is `index.html` — front matter controls hero/cards, body HTML controls sections
5. CSS customizations are at the bottom of `assets/css/main.scss`
6. Navigation is `_data/navigation.yml`
