---
# ============================================================
# PROJECT FRONT MATTER — fill in everything between the ---
# Fields marked REQUIRED must be filled in.
# Fields marked OPTIONAL can be deleted if not needed.
# ============================================================

# REQUIRED — shown as the page title and card title
title: "Your Project Title"

# REQUIRED — controls sort order on the index page (1 = top)
order: 99

# OPTIONAL — set to false to hide this project from the index
featured: false

# REQUIRED — one or two sentences shown on the index card AND
# at the top of the project page. Write it in your own voice.
# Think: what is this, and why does it matter?
summary: >
  Write a short summary here. This appears on the project card
  on the index page, so keep it punchy — one or two sentences.

# OPTIONAL — short label shown as a badge on the card.
# Examples: "Live on Steam", "Open Source", "WIP", "Private"
status: "Live on Steam"

# OPTIONAL — changes the badge color. Options:
#   (blank)  = blue (default)
#   open     = green
#   wip      = yellow
#   private  = grey
status_color: ""

# OPTIONAL — tech tags shown on the card and project page
tags:
  - C++
  - Unreal Engine
  - Jenkins

# OPTIONAL — path to a cover image, relative to the site root
# Put images in assets/images/
image: "/assets/images/your-project-cover.png"
image_alt: "Brief description of the image for screen readers"

# OPTIONAL — quick stat numbers shown at the top of the project page.
# Use these for things that are genuinely meaningful, not filler.
# Delete the whole block if you have nothing worth showing.
stats:
  - value: "15"
    label: "Team size"
  - value: "2"
    label: "Steam releases"
  - value: "1 year"
    label: "In development"

# OPTIONAL — links shown as buttons on the project page,
# and the source/live links shown on the index card.
# Delete any you don't need.
link_detail_label: "Play on Steam"
link_detail_url: "https://store.steampowered.com/..."

link_source_label: "View on GitHub"
link_source_url: "https://github.com/..."

link_extra_label: "Read the blog"
link_extra_url: "https://..."
---

<!-- ============================================================
     PROJECT BODY
     Everything below this line is the detail page content.
     Write in Markdown. Use headings to break up sections.

     READING MODE REMINDER: visitors here clicked through because
     they're interested. Write for someone who wants the full
     picture — but still use headings so they can skim.
     ============================================================ -->

## Overview

Write a few paragraphs here giving context for the project. What
is it? What problem does it solve? What was your role?

This is the place to be honest about scope and context — "built
during a BUAS capstone year" or "solo side project" tells the
reader exactly what kind of work this is.

## The challenge

What was technically or organisationally difficult about this?
Write about what you actually had to figure out, not a generic
description of the domain.

<!-- Use callout boxes for challenge/solution pairs if you want
     to make them visually distinct: -->

<div class="callout callout--challenge">
  <div class="callout__label">Challenge</div>
  Describe a specific hard problem you had to solve.
</div>

<div class="callout callout--solution">
  <div class="callout__label">Solution</div>
  Describe what you built or decided and why.
</div>

## What I built

Go into the technical detail here. This is where you can show
off — code snippets, architecture decisions, tradeoffs you made.

```cpp
// Example code block
void YourSystem::DoThing()
{
    // Real code from the project works well here
}
```

### A specific subsystem or decision

Use `###` headings to break up longer sections without adding
too much visual weight.

## Results

What came out of it? Ship it? Learned something specific?
Concrete outcomes beat vague claims — "shipped to 200 concurrent
players on launch day" beats "successfully deployed".

If there were things that didn't go well or things you'd do
differently, this is a good place to be honest about that too.
It reads as more credible than a pure highlight reel.
