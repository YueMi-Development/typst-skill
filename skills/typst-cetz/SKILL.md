---
name: typst-cetz
description: |
  CeTZ (TikZ-inspired drawing for Typst) patterns for creating diagrams. Use this skill when creating diagrams, flowcharts, graphs, plots, or technical illustrations in Typst documents. Covers coordinate systems, drawing primitives, styling, anchors, marks, trees, and plotting.
license: CC-BY-4.0
metadata:
  author: Ulrich Atz
---

# CeTZ Diagrams

CeTZ is a drawing library for Typst with an API inspired by TikZ and Processing. Reference: https://cetz-package.github.io/docs/

## Basic Structure

```typst
#import "@preview/cetz:0.5.0"

#cetz.canvas({
  import cetz.draw: *
  // Drawing commands here
})
```

Draw functions are imported inside the canvas block scope because some override Typst's built-in functions (e.g., `line`).

### Canvas Options

```typst
// Default: 1 unit = 1cm
#cetz.canvas({...})

// Custom unit length
#cetz.canvas(length: 0.5cm, {...})

// Scale to parent width
#cetz.canvas(length: 50%, {...})
```

Canvas auto-sizes to fit content (no fixed width/height).

## Coordinate Systems

### Cartesian (XYZ)

```typst
// 2D coordinates
(2, 3)        // x=2, y=3
(x: 2, y: 3)  // explicit form

// 3D coordinates
(1, 2, 0.5)   // x, y, z

// With units
(10pt, 2cm)
```

Y-axis points upward (positive direction).

### Previous Coordinate

```typst
line((0, 0), (1, 1))
line((), (2, 0))  // () references last point (1, 1)
```

Initial previous coordinate is `(0, 0, 0)`.

### Relative Coordinates

```typst
// Offset from previous position
line((0, 0), (rel: (1, 1)))

// Non-updating relative (doesn't move "previous")
line((0, 0), (rel: (1, 0), update: false), (rel: (0, 1)))
```

### Polar Coordinates

```typst
// angle and radius from origin
(angle: 45deg, radius: 2)

// Elliptical radius
(angle: 30deg, radius: (2, 1))  // (x-radius, y-radius)
```

### Anchor Coordinates

```typst
// Reference named elements
circle((0, 0), name: "c")
line("c.east", (2, 0))      // from circle's east anchor
line("c.north", "c.south")  // between anchors
```

### Interpolation

```typst
// Point between two coordinates
(a, 50%, b)           // midpoint
(a, 0.25, b)          // 25% from a to b
((0,0), 1cm, (2,2))   // 1cm from first point toward second
```

### Perpendicular Intersection

```typst
// Intersection of horizontal through a and vertical through b
(a, "|-", b)
(a, "-|", b)  // opposite order
```

### Projection

```typst
// Project point onto line
(pt, "_|_", line-start, line-end)
(project: pt, onto: (a, b))
```

### Barycentric

```typst
// Weighted combination of vectors
(bary: (a: 1, b: 2, c: 1))  // weighted average
```

### Tangent

```typst
// Point where line from external point touches shape tangentially
(element: "circle", point: "external-point", solution: 1)  // solution: 1 or 2
```

### Function Coordinate

```typst
// Apply function to resolved coordinate
(v => cetz.vector.add(v, (0, -1)), "element.anchor")
```

## Drawing Functions API

### line

```typst
line(..pts-style, close: false, name: none)
```

- `..pts-style`: Two or more coordinates, plus optional style key-value pairs
- `close`: When `true`, closes the path to form a polygon
- `name`: Optional element identifier
- Anchors: path anchors + `centroid` (for closed non-self-intersecting polygons)

```typst
// Basic line
line((0, 0), (2, 1))

// Multiple points
line((0, 0), (1, 1), (2, 0), (3, 1))

// Closed path
line((0, 0), (1, 1), (1, 0), close: true)

// With arrow
line((0, 0), (2, 0), mark: (end: ">"))
line((0, 0), (2, 0), mark: (start: "<", end: ">"))
```

### circle

```typst
circle(..points-style, name: none, anchor: none)
```

- `..points-style`: Position coordinate; if two coords given, distance = radius
- `radius` (style): number or `(x-radius, y-radius)` for ellipse (default: 1)
- Anchors: border, path, center (default)

```typst
// Circle (default radius: 1)
circle((0, 0))
circle((0, 0), radius: 10pt)

// Ellipse
circle((0, 0), radius: (2, 1))  // (x-radius, y-radius)

// Circle through 3 points
circle-through((0, 0), (1, 1), (2, 0))
```

### rect

```typst
rect(a, b, name: none, anchor: "center", ..style)
```

- `a`: Bottom-left corner coordinate
- `b`: Top-right corner; use `(rel: (width, height))` for relative sizing
- `radius` (style): Corner rounding - number, ratio, or per-corner dict with keys `north`, `east`, `south`, `west`, `north-west`, `north-east`, `south-west`, `south-east`, `rest`
- Anchors: border, path, center (default)

```typst
// Two corners
rect((0, 0), (2, 1))

// With radius (rounded corners)
rect((0, 0), (2, 1), radius: 0.2)

// Per-corner radius
rect((0, 0), (2, 1), radius: (north-west: 0.5, rest: 0.1))
```

### arc

```typst
arc(position, start: auto, stop: auto, delta: auto, name: none, anchor: none, ..style)
```

- Must specify 2 of 3: `start`, `stop`, `delta`
- `radius` (style): number or `(x, y)` for elliptical (default: 1)
- `mode` (style): `"OPEN"` (arc only), `"CLOSE"` (chord), `"PIE"` (sector)
- Anchors: `arc-start`, `arc-end`, `arc-center`, `origin`, `chord-center`, border, path

```typst
// Start and stop angles
arc((0, 0), start: 0deg, stop: 90deg, radius: 1)

// Delta angle
arc((0, 0), start: 0deg, delta: 270deg, radius: 1)

// Modes
arc((0, 0), start: 0deg, stop: 270deg, radius: 1, mode: "OPEN")  // default
arc((0, 0), start: 0deg, stop: 270deg, radius: 1, mode: "PIE")   // filled wedge
arc((0, 0), start: 0deg, stop: 270deg, radius: 1, mode: "CLOSE") // chord

// Arc through 3 points
arc-through((0, 0), (1, 1), (2, 0))
```

### bezier

```typst
bezier(start, end, ..ctrl-style, name: none)
```

- `start`, `end`: Start and end coordinates
- `..ctrl-style`: 1 control point = quadratic, 2 = cubic
- Anchors: path anchors + `ctrl-0`, `ctrl-1` (control points)

```typst
// Quadratic (1 control point)
bezier((0, 0), (2, 0), (1, 1))

// Cubic (2 control points)
bezier((0, 0), (3, 0), (1, 1), (2, 1))

// Bezier through 3 points (auto-calculates control points)
bezier-through((0, 0), (1, 1), (2, 0))
```

### Grid

```typst
grid((0, 0), (4, 4))
grid((0, 0), (4, 4), step: 0.5)
grid((0, 0), (4, 4), step: (1, 0.5))  // different x/y steps
```

### content

```typst
content(..args-style, angle: 0deg, anchor: "center", name: none)
```

- Single coord: place at position; two coords: place inside rectangle
- `angle`: Rotation angle or coordinate to point toward
- `padding` (style): Spacing around content
- `frame` (style): `"rect"`, `"circle"`, or `none`
- Anchors: `center`, `mid`, `mid-east`, `mid-west`, `base`, `base-east`, `base-west`

```typst
content((1, 1), [Hello World])
content((1, 1), $x^2 + y^2 = r^2$)
content((1, 1), anchor: "north", [Label])
content((0, 0), (2, 1), [Fits in box])  // content inside rectangle
```

### Other Shape Functions

```typst
// Regular polygon
polygon((0, 0), vertices: 6, radius: 1)

// Star shape
n-star((0, 0), n: 5, inner-radius: 0.5, outer-radius: 1)

// Smooth curves through points
hobby((0, 0), (1, 1), (2, 0))     // Hobby spline
catmull((0, 0), (1, 1), (2, 0))   // Catmull-Rom spline

// Merge multiple paths into one
merge-path({
  line((0, 0), (1, 0))
  arc((), start: 0deg, delta: 180deg, radius: 0.5)
  line((), (0, 0))
})
```

## Styling

### Direct Styling

```typst
// Stroke
line((0, 0), (1, 1), stroke: red)
line((0, 0), (1, 1), stroke: blue + 2pt)
line((0, 0), (1, 1), stroke: (paint: green, thickness: 1.5pt, dash: "dashed"))

// Fill
circle((0, 0), fill: blue)
rect((0, 0), (1, 1), fill: red.lighten(50%))

// Combined
circle((0, 0), fill: yellow, stroke: orange + 2pt)
```

### Global Styles

```typst
set-style(stroke: black + 0.5pt, fill: none)

// Element-specific defaults
set-style(
  circle: (fill: blue.lighten(80%)),
  line: (stroke: red),
)
```

### Style Properties

- `fill`: color or `none`
- `stroke`: color, length, dictionary, or `none`
  - `paint`: stroke color
  - `thickness`: line width
  - `dash`: `"solid"`, `"dashed"`, `"dotted"`, `"dash-dotted"`
  - `cap`: `"butt"`, `"round"`, `"square"`
  - `join`: `"miter"`, `"round"`, `"bevel"`
- `fill-rule`: `"non-zero"` or `"even-odd"` (for self-intersecting paths)

## Anchors

### Named Anchors

Elements have type-specific anchors:

```typst
circle((0, 0), name: "c")
// Available: c.center, c.north, c.south, c.east, c.west, etc.

rect((0, 0), (2, 1), name: "r")
// Available: r.center, r.north, r.south, r.east, r.west,
//            r.north-east, r.north-west, r.south-east, r.south-west
```

### Border Anchors

Angles from center to border:

```typst
"element.0deg"    // east (right)
"element.90deg"   // north (up)
"element.45deg"   // northeast
```

### Path Anchors

Points along an element's path:

```typst
"element.start"   // path start
"element.mid"     // path midpoint
"element.end"     // path end
"element.50%"     // 50% along path
"element.2cm"     // 2cm along path
```

### Positioning with Anchors

```typst
// Place element's anchor at position
circle((0, 0), anchor: "west")   // west anchor at origin

// Connect elements
circle((0, 0), name: "a")
circle((3, 0), name: "b")
line("a.east", "b.west")
```

## Marks (Arrows)

### Basic Marks

```typst
line((0, 0), (2, 0), mark: (end: ">"))
line((0, 0), (2, 0), mark: (start: "<", end: ">"))
line((0, 0), (2, 0), mark: (end: "stealth"))
```

### Mark Symbols

- `">"`, `"<"` - simple arrows
- `"stealth"` - stealth arrow
- `"|"` - bar
- `"o"` - circle
- `"<>"` - diamond
- `"["`, `"]"` - brackets

### Mark Options

```typst
line((0, 0), (2, 0), mark: (
  symbol: ">",        // mark symbol (or use start/end)
  start: "<",         // mark at path start
  end: ">",           // mark at path end
  length: 0.2cm,      // size in pointing direction
  width: 0.15cm,      // size perpendicular to direction
  inset: 0.05cm,      // distance inward for details
  scale: 1,           // multiplier for length/width/inset
  sep: 0.1cm,         // separation between multiple marks
  pos: none,          // absolute/relative position on path
  offset: none,       // advance position instead of override
  anchor: "tip",      // "tip", "center", or "base"
  slant: 0%,          // rotation relative to arrow axis
  harpoon: false,     // draw only top half
  flip: false,        // reverse orientation
  reverse: false,     // change direction
  fill: auto,         // fill color
  stroke: auto,       // stroke color
))
```

## Transformations

```typst
// Scale
scale(2)
scale(x: 2, y: 0.5)

// Rotate
rotate(45deg)
rotate(45deg, origin: (1, 1))

// Translate
translate((2, 1))

// Group with local transform
group({
  rotate(45deg)
  rect((0, 0), (1, 1))
})
```

## Grouping Functions

```typst
// Group elements with shared transform/style scope
group({
  rotate(45deg)
  rect((0, 0), (1, 1))
})

// Hide elements (still creates anchors)
hide({ line((0, 0), (2, 2), name: "helper") })

// Find intersections between named elements
intersections("i", "line1", "line2")
circle("i.0", radius: 2pt)  // first intersection

// Define anchor in current group
anchor("my-anchor", (1, 1))

// Copy anchors from another element
copy-anchors("source-element")

// Iterate over element's anchors
for-each-anchor("element", (name, pos) => {
  circle(pos, radius: 2pt)
})

// Assign to layer (for z-ordering)
on-layer("background", {
  rect((0, 0), (5, 5), fill: gray)
})

// Float without affecting bounding box
floating({
  content((10, 10), [Outside])
})
```

## Tree Library

```typst
#import cetz.tree: tree

#cetz.canvas({
  import cetz.draw: *
  import cetz.tree: *

  tree(
    ([Root], ([A], [A1], [A2]), ([B], [B1])),
    direction: "down",
    grow: 1.5,
    spread: 2,
  )
})
```

### Tree Options

- `direction`: `"up"`, `"down"`, `"left"`, `"right"`
- `grow`: distance between levels
- `spread`: distance between siblings
- `draw-node`: custom node drawing callback
- `draw-edge`: custom edge drawing callback

## Plot Library

```typst
#import "@preview/cetz:0.5.0"
#import "@preview/cetz-plot:0.1.3": plot, chart

#cetz.canvas({
  import cetz.draw: *

  plot.plot(
    size: (8, 6),
    x-tick-step: 1,
    y-tick-step: 1,
    {
      plot.add(domain: (0, 4), x => calc.pow(x, 2))
      plot.add(((0, 0), (1, 2), (2, 1), (3, 3)))
    }
  )
})
```

## Common Patterns

### Flowchart

```typst
#cetz.canvas({
  import cetz.draw: *

  rect((0, 0), (2, 1), name: "start", fill: green.lighten(80%))
  content("start", [Start])

  rect((0, -2), (2, -1), name: "process", fill: blue.lighten(80%))
  content("process", [Process])

  line("start.south", "process.north", mark: (end: ">"))
})
```

### Coordinate Axes

```typst
#cetz.canvas({
  import cetz.draw: *

  line((-0.5, 0), (4, 0), mark: (end: ">"))
  line((0, -0.5), (0, 3), mark: (end: ">"))

  content((4, 0), anchor: "west", $x$)
  content((0, 3), anchor: "south", $y$)
})
```

## SVG Paths (0.5.0+)

```typst
// Render SVG path data directly
svg-path("M 0 0 L 2 0 L 1 1 Z")
```

## Perspective Projection (0.5.0+)

```typst
// Native perspective projection mode
#cetz.canvas({
  import cetz.draw: *

  // Apply perspective transform
  set-transform(cetz.matrix.transform-perspective(distance: 10))

  // 3D drawing with perspective
  line((0, 0, 0), (2, 0, 0))
  line((0, 0, 0), (0, 2, 0))
  line((0, 0, 0), (0, 0, 2))
})
```

## Anti-Patterns

| Avoid | Prefer | Reason |
|-------|--------|--------|
| Hard-coded positions | Named elements + anchors | Maintainable |
| Repeated styling | `set-style()` | DRY |
| Complex inline calculations | Interpolation coordinates | Readable |
| Manual arrow drawing | `mark` parameter | Consistent |

## Touying Integration

For animated diagrams in slide presentations, use CeTZ with Touying's `touying-reducer`. See `/typst-touying` skill for full documentation.

```typst
#let cetz-canvas = touying-reducer.with(
  reduce: cetz.canvas,
  cover: cetz.draw.hide.with(bounds: true)
)

#slide[
  #cetz-canvas({
    import cetz.draw: *
    rect((0, 0), (2, 2))
    (pause,)              // Animation marker
    circle((3, 1), radius: 0.5)
  })
]
```

## Resources

- [CeTZ Documentation](https://cetz-package.github.io/docs/)
- [CeTZ GitHub](https://github.com/cetz-package/cetz)
- [cetz-plot Manual](https://github.com/cetz-package/cetz-plot/blob/stable/manual.pdf)
