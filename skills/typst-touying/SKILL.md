---
name: typst-touying
description: |
  Touying slide framework for Typst. Use when creating presentations in Typst, adding slide animations, configuring Touying themes, using pause/meanwhile markers, building multi-file presentations, or integrating CeTZ/Fletcher diagrams with animations. Covers all built-in themes (simple, university, metropolis, aqua, dewdrop, stargazer) and custom theme creation.
license: CC-BY-4.0
metadata:
  author: Ulrich Atz
---

# Touying Slide Framework

> Based on Touying 0.7.3 for Typst

Touying is a presentation framework for Typst that provides content/style separation, fast compilation (milliseconds vs Beamer's seconds), and dynamic slide capabilities via `#pause` and `#meanwhile` markers.

**Related skills:**
- `/typst` - Core Typst syntax, styling, math, tables, figures
- `/typst-cetz` - Detailed CeTZ drawing API (coordinates, shapes, anchors, marks, trees, plots)

## Presentation Design Principles

### Typography: limit to 4-5 font sizes

A presentation should use at most 4-5 distinct font sizes, applied consistently within each slide. Draw from a LaTeX-inspired scale (adapt exact values to your design):

| Role | Approximate size | LaTeX analogue |
|---|---|---|
| Presentation title | 28-32pt | `\LARGE` |
| Section divider | 24-28pt | `\Large` |
| Slide title | 20-24pt | `\large` |
| Body text | 16-18pt | `\normalsize` |
| Captions / footnotes | 12-14pt | `\footnotesize` |

Define sizes once in your theme or preamble and reference them everywhere. Never introduce an ad-hoc size; if text needs emphasis, use weight or color, not a sixth size. Every element on a single slide should come from this scale—mixing arbitrary sizes looks unprofessional.

```typst
// Example: define your type scale once
#let title-size = 24pt
#let body-size  = 17pt
#let small-size = 13pt

#set text(size: body-size)
```

### Slide titles: sentence case

All slide titles use **sentence case** (capitalize only the first word and proper nouns). The only exception is the presentation name on the title slide, which may use title case.

- Good: `== Results from the pilot study`
- Good: `== How ESG scores affect returns`
- Bad: `== Results From The Pilot Study`

### Color palette: restrained, intentional

Avoid the default "Python conference" color scheme (saturated blue-orange-green) and overuse of colored background tints. Prefer:

- **Neutral base**: white or very light warm grey (`#f5f5f5`) backgrounds; dark grey (`#333`) or near-black text.
- **One accent color** for emphasis (headings, highlights, links). A second accent is acceptable for contrast (e.g., positive/negative) but not required.
- **Grey for secondary elements**: axis lines, borders, annotations, de-emphasized text. A medium grey (`#888`) or light grey (`#ccc`) works well.
- **Background tints**: use sparingly. A subtle grey tint on a code block or callout is fine; avoid tinting every other slide or coloring full-bleed section dividers in saturated hues.

```typst
// Example: clean, restrained palette
config-colors(
  primary: rgb("#2c3e50"),         // dark blue-grey accent
  secondary: rgb("#7f8c8d"),       // medium grey
  neutral-lightest: rgb("#fafafa"),
  neutral-darkest: rgb("#333333"),
)
```

## Quick Start

```typst
#import "@preview/touying:0.7.3": *
#import themes.simple: *

#show: simple-theme.with(aspect-ratio: "16-9")

= Section Title

== Slide Title

Content here.

#pause

More content revealed on next click.
```

## Code Styles

### Heading-Based (Simple Style)

Use standard Typst headings to create slides:

```typst
= Section Title     // Creates section divider slide
== Slide Title      // Creates content slide
=== Subsection      // Theme-dependent behavior

Content under headings becomes slide content.

#pause              // Progressive reveal

More content.
```

Special markers:
- `== <touying:hidden>` - Creates slide without title
- `#pagebreak()` or `---` - Manual page break

### Block Style (Explicit Functions)

For more control, use explicit slide functions:

```typst
#slide[
  Content here.

  #pause

  More content.
]

#focus-slide[
  Key message
]

#title-slide()
```

Block style enables:
- Theme-specific slide variants
- Callback-style animations with `#only` and `#uncover`
- Custom parameters per slide

## Animation Markers

### Simple Animations

**#pause** - Reveals content progressively:

```typst
#slide[
  First item
  #pause
  Second item (appears on click)
  #pause
  Third item
]
```

Works inline: `First #pause Second`

**#meanwhile** - Shows content simultaneously across subslides:

```typst
#slide[
  Left column content
  #pause
  More left content

  #meanwhile

  Right column (shows with "Left column content")
  #pause
  More right (shows with "More left content")
]
```

### Complex Animations

For advanced control, use callback-style functions:

```typst
#slide(repeat: 3)[
  #let (uncover, only, alternatives) = utils.methods(self)

  #only(1)[Only on subslide 1]
  #only("2-")[On subslides 2 and after]

  #uncover("2-3")[Visible on 2-3, space reserved otherwise]

  #alternatives[Option A][Option B][Option C]
]
```

**Index syntax:**
- `1` - Specific subslide
- `"2-"` - From subslide 2 onward
- `"2-3"` - Subslides 2 through 3
- `"-3"` - Up to subslide 3

**Functions:**
- `only(idx)[content]` - Shows only on specified subslides, no space when hidden
- `uncover(idx)[content]` - Shows on specified subslides, reserves space when hidden
- `alternatives[a][b][c]` - Shows different content per subslide

### Equation Animations

```typst
#touying-equation(`
  a + b &= c \
  #pause
  d + e &= f
`)
```

### Item-by-Item Animation (0.6.3+)

Progressively reveal list items without manual `#pause` between each:

```typst
#slide[
  #item-by-item[
    - First point
    - Second point
    - Third point
  ]
]
```

Works with bullet lists, enumerations, and term lists.

### Animated Code Reveals (0.6.3+)

```typst
#slide[
  #touying-raw(
    ```python
    def hello():
        print("Hello")
        #pause
        return True
    ```
  )
]
```

### Jump Animation Control (0.6.3+)

Unified animation primitive (`#pause` and `#meanwhile` are syntax sugar for this):

```typst
#slide[
  Content on subslide 1
  #jump(2)           // skip to subslide 3
  Content on subslide 3
]

// Relative jumps
#jump(1, relative: true)  // advance by 1
```

### Handout Controls (0.6.3+)

```typst
// Content only in handout mode
#handout-only[Extra details for handout]

// Control which subslide appears in handout
#slide(handout-subslides: 3)[
  // Handout shows subslide 3 state
]
```

### Speaker Notes (0.7.3+)

Speaker notes now attach to the slide above them:

```typst
== My Slide

Content here.

#speaker-note[
  Remember to mention the key finding.
]
```

### Slide Overflow (0.7.1+)

```typst
// Allow content to break across pages
#slide(breakable: true)[
  Very long content...
]

// Clip overflowing content
#slide(clip: true)[
  Content that might overflow...
]
```

### Lazy Layout (0.7.1+)

Equalize heights/widths across columns:

```typst
#slide[
  #lazy-layout(
    columns: (1fr, 1fr),
    [Short content],
    [Much longer content that would normally make this column taller],
  )
]

// Shorthand aliases
#cols[Left][Right]           // alias for side-by-side (0.7.3+)
#lazy-h[Left][Right]        // equalized horizontal
#lazy-v[Top][Bottom]         // equalized vertical
```

### Access Configuration (0.7.1+)

```typst
// Read current Touying config at runtime
#let cfg = touying-get-config()
```

## Cover Customization

Default cover uses `hide()` (invisible but preserves layout).

```typst
// Semi-transparent cover (shows grayed content)
#show: simple-theme.with(
  config-methods(
    cover: utils.semi-transparent-cover.with(alpha: 85%)
  )
)

// For enum/list markers that hide() doesn't conceal
#show: simple-theme.with(
  config-methods(
    cover: (self: none, body) => box(scale(x: 0%, body))
  )
)
```

## Themes

### Simple Theme

```typst
#import "@preview/touying:0.7.3": *
#import themes.simple: *

#show: simple-theme.with(
  aspect-ratio: "16-9",        // or "4-3"
  footer: [Footer text],
  primary: rgb("#004488"),     // Theme color
)

#title-slide()
#centered-slide[Centered content]
#focus-slide[Key point]
```

### University Theme

```typst
#import themes.university: *

#show: university-theme.with(
  aspect-ratio: "16-9",
  config-info(
    title: [Presentation Title],
    subtitle: [Subtitle],
    author: [Author Name],
    date: datetime.today(),
    institution: [Institution],
    logo: image("logo.png", width: 2cm),
  ),
  progress-bar: true,
)

#title-slide()
#matrix-slide[Column 1][Column 2]  // Multi-column
#focus-slide[Important point]
```

Colors: primary `#04364A`, secondary `#176B87`, tertiary `#448C95`

### Metropolis Theme

```typst
#import themes.metropolis: *

#show: metropolis-theme.with(
  aspect-ratio: "16-9",
  footer-progress: true,
  config-info(
    title: [Title],
    author: [Author],
    institution: [Institution],
  ),
)

// Recommended fonts
#set text(font: "Fira Sans", weight: "light", size: 20pt)
#show math.equation: set text(font: "Fira Math")

#title-slide()
#new-section-slide[Section Name]
#focus-slide[Key point]
```

Colors: primary `#eb811b` (orange), secondary `#23373b`

### Aqua Theme

```typst
#import themes.aqua: *

#show: aqua-theme.with(
  aspect-ratio: "16-9",
  config-info(title: [Title], author: [Author]),
)

#title-slide()
#outline-slide()
#focus-slide[Key message]
```

Colors: primary `#003F88`

### Dewdrop Theme

Two navigation modes: sidebar or mini-slides.

```typst
#import themes.dewdrop: *

#show: dewdrop-theme.with(
  aspect-ratio: "16-9",
  navigation: "sidebar",        // or "mini-slides" or none
  config-info(
    title: [Title],
    author: [Author],
  ),
)

#title-slide()
#focus-slide[Key point]
```

### Stargazer Theme

```typst
#import themes.stargazer: *

#show: stargazer-theme.with(
  aspect-ratio: "16-9",
  config-info(
    title: [Title],
    author: [Author],
    date: datetime.today(),
    institution: [Institution],
  ),
)

#title-slide()
#outline-slide()
#focus-slide[Key point]
```

Colors: primary `#005bac`

## Global Settings

### Presentation Info

```typst
#show: simple-theme.with(
  config-info(
    title: [Presentation Title],
    subtitle: [Subtitle],
    author: [Author Name],
    date: datetime.today(),
    institution: [Institution],
    logo: image("logo.png"),
  ),
)
```

Access in templates: `self.info.title`, `self.info.author`, etc.

### Date Formatting

```typst
#show: simple-theme.with(
  config-common(datetime-format: "[year]-[month]-[day]"),
)
```

### Handout Mode

Disables animations, shows final state of each slide:

```typst
#show: simple-theme.with(
  config-common(handout: true),
)
```

## Page Layout

**Warning:** Do not use `set page(..)` directly; Touying resets it. Use `config-page()` instead.

### Configure Page

```typst
#show: simple-theme.with(
  config-page(
    margin: (x: 4em, y: 2em),
    header: align(top)[Header],
    footer: align(bottom)[Footer],
    header-ascent: 0em,
    footer-descent: 0em,
  ),
)
```

### Multi-Column Slides

```typst
#slide(composer: (1fr, 1fr))[
  Left column content
][
  Right column content
]

// Custom proportions
#slide(composer: (2fr, 1fr))[
  Wider left
][
  Narrower right
]
```

## Sections and Outline

### Heading Levels

Default mapping (theme-dependent):
- `= Section` - Section divider
- `== Title` - Slide title
- `=== Subsection` - Varies by theme

Configure with `slide-level` in `config-common`.

### Section Numbering

```typst
#set heading(numbering: "1.1")
#show heading.where(level: 1): set heading(numbering: "1.")
```

### Table of Contents

```typst
#slide[
  #outline()
]

// Auto-fitting columns
#slide[
  #adaptive-columns()[
    #outline()
  ]
]

// Progressive outline (dewdrop theme)
#progressive-outline()
```

## Multi-File Organization

**globals.typ** - Shared imports and configuration:

```typst
#import "@preview/touying:0.7.3": *
#import themes.university: *

#let my-theme = university-theme.with(
  config-info(
    title: [My Presentation],
    author: [Author],
  ),
)
```

**main.typ** - Entry point:

```typst
#import "globals.typ": *

#show: my-theme

#include "sections/intro.typ"
#include "sections/methods.typ"
#include "sections/conclusion.typ"
```

**sections/intro.typ** - Content file:

```typst
#import "../globals.typ": *

= Introduction

== Background

Content here.
```

## Utilities

### Fit to Height/Width

```typst
// Fill remaining vertical space
#utils.fit-to-height(1fr)[
  #image("large-image.png")
]

// Fill horizontal space
#utils.fit-to-width(1fr)[
  #lorem(50)
]
```

Parameters:
- `grow: true/false` - Allow expansion
- `shrink: true/false` - Allow reduction
- `prescale-width` - Assumed container width before scaling

### Counters

```typst
// Current slide number
#utils.slide-counter.display()

// Progress bar
#utils.touying-progress(ratio => {
  box(width: ratio * 100%, height: 2pt, fill: primary)
})
```

### Appendix

```typst
#show: appendix

= Appendix

== Backup Slides

// Use <touying:unoutlined> to hide from outline
```

## CeTZ Integration

Animate CeTZ diagrams with `touying-reducer`. See `/typst-cetz` skill for full CeTZ API documentation.

```typst
#import "@preview/cetz:0.5.0"

#let cetz-canvas = touying-reducer.with(
  reduce: cetz.canvas,
  cover: cetz.draw.hide.with(bounds: true)
)

#slide[
  #cetz-canvas({
    import cetz.draw: *
    rect((0, 0), (2, 2))
    (pause,)
    circle((3, 1), radius: 0.5)
    (pause,)
    line((2, 1), (2.5, 1))
  })
]
```

For advanced animations, use `only` and `uncover` within the canvas by updating the cover method on `self`.

## Fletcher Integration

Animate Fletcher diagrams:

```typst
#import "@preview/fletcher:0.5.8"

#let fletcher-diagram = touying-reducer.with(
  reduce: fletcher.diagram,
  cover: fletcher.hide
)

#slide[
  #fletcher-diagram(
    node((0, 0), [Start]),
    (pause,)
    edge("->"),
    node((1, 0), [End]),
  )
]
```

## Building Custom Themes

```typst
#let my-theme(
  aspect-ratio: "16-9",
  ..args,
  body,
) = {
  show: touying-slides.with(
    config-page(..utils.page-args-from-aspect-ratio(aspect-ratio)),
    config-colors(
      primary: rgb("#004488"),
      secondary: rgb("#88aa00"),
      neutral-lightest: white,
      neutral-darkest: black,
    ),
    config-store(title: none),
    config-common(slide-fn: slide),
    ..args,
  )

  body
}

#let slide(title: auto, ..args) = touying-slide-wrapper(self => {
  // Custom slide implementation
  touying-slide(
    self: self,
    ..args,
  )
})

#let title-slide() = touying-slide-wrapper(self => {
  // Custom title slide
})

#let focus-slide(body) = touying-slide-wrapper(self => {
  // Custom focus slide
})
```

## Common Patterns

### Alert Text

```typst
#show strong: alert  // Makes **text** use primary color

This is **important** text.
```

### Disable Section Slides

```typst
#show: simple-theme.with(
  config-common(new-section-slide-fn: none),
)
```

### Custom Header/Footer

```typst
#show: simple-theme.with(
  header: self => self.info.title,
  footer: self => [#self.info.author #h(1fr) #utils.slide-counter.display()],
)
```

## Resources

- [Touying Documentation](https://touying-typ.github.io/)
- [Touying GitHub](https://github.com/touying-typ/touying)
- [Typst Package Registry](https://typst.app/universe/package/touying)
