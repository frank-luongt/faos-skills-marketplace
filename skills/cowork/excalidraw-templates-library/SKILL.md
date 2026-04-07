---
name: excalidraw-templates-library
description: "Shared Excalidraw element templates, color palettes, and styling conventions for consistent diagram creation."
tags: [excalidraw, templates, diagrams, design-system]
---

# Excalidraw Templates Library

Shared templates for consistent diagram styling.

```yaml
flowchart:
  viewport:
    x: 0
    y: 0
    zoom: 1
  grid:
    size: 20
  spacing:
    vertical: 100
    horizontal: 180
  elements:
    start:
      type: ellipse
      width: 120
      height: 60
      label: "Start"
    process:
      type: rectangle
      width: 160
      height: 80
      roundness: 8
    decision:
      type: diamond
      width: 140
      height: 100
    end:
      type: ellipse
      width: 120
      height: 60
      label: "End"

diagram:
  viewport:
    x: 0
    y: 0
    zoom: 1
  grid:
    size: 20
  spacing:
    vertical: 120
    horizontal: 200
  elements:
    component:
      type: rectangle
      width: 180
      height: 100
      roundness: 8
    database:
      type: rectangle
      width: 140
      height: 80
    service:
      type: rectangle
      width: 160
      height: 90
      roundness: 12
    external:
      type: rectangle
      width: 140
      height: 80

wireframe:
  viewport:
    x: 0
    y: 0
    zoom: 0.8
  grid:
    size: 20
  spacing:
    vertical: 40
    horizontal: 40
  elements:
    container:
      type: rectangle
      width: 800
      height: 600
      strokeStyle: solid
      strokeWidth: 2
    header:
      type: rectangle
      width: 800
      height: 80
    button:
      type: rectangle
      width: 120
      height: 40
      roundness: 4
    input:
      type: rectangle
      width: 300
      height: 40
      roundness: 4
    text:
      type: text
      fontSize: 16

dataflow:
  viewport:
    x: 0
    y: 0
    zoom: 1
  grid:
    size: 20
  spacing:
    vertical: 120
    horizontal: 200
  elements:
    process:
      type: ellipse
      width: 140
      height: 80
      label: "Process"
    datastore:
      type: rectangle
      width: 140
      height: 80
      label: "Data Store"
    external:
      type: rectangle
      width: 120
      height: 80
      strokeWidth: 3
      label: "External Entity"
    dataflow:
      type: arrow
      strokeWidth: 2
      label: "Data Flow"

# -------------------------------------------------------------------
# Visual Pattern Layouts
# Referenced by all diagram workflows for pattern-based layout decisions.
# See .faos/core/resources/excalidraw/excalidraw-helpers.md for detailed
# descriptions and usage guidance for each pattern.
# -------------------------------------------------------------------
visual_patterns:
  fan_out:
    description: "One source distributing to many targets"
    use_for:
      - "event broadcasting"
      - "load distribution"
      - "API gateway routing"
      - "message fanout"
    layout:
      source_position: "left-center"
      target_arrangement: "vertical-stack-right"
      min_targets: 3
      max_targets: 7
      target_spacing: 80
      arrow_type: "straight"
    example_structure:
      elements: "1 source + N targets + N arrows"
      typical_count: "8-16 elements"

  convergence:
    description: "Many sources flowing into one target"
    use_for:
      - "data aggregation"
      - "merge points"
      - "unified API"
      - "log collection"
    layout:
      source_arrangement: "vertical-stack-left"
      target_position: "right-center"
      min_sources: 3
      max_sources: 7
      source_spacing: 80
      arrow_type: "straight"
    example_structure:
      elements: "N sources + 1 target + N arrows"
      typical_count: "8-16 elements"

  tree:
    description: "Hierarchical parent-child relationships"
    use_for:
      - "org charts"
      - "class inheritance"
      - "directory structures"
      - "decision trees"
    layout:
      root_position: "top-center"
      child_direction: "downward"
      level_spacing: 120
      sibling_spacing: 100
      arrow_type: "straight"
    example_structure:
      elements: "root + children + grandchildren + arrows"
      typical_count: "15-40 elements"

  cycle:
    description: "Repeating process with feedback loop"
    use_for:
      - "CI/CD pipelines"
      - "retry loops"
      - "iterative refinement"
      - "event loops"
    layout:
      arrangement: "rectangular-loop"
      min_stages: 3
      max_stages: 8
      stage_spacing: 140
      arrow_type: "straight"
      entry_position: "top-left"
      exit_position: "bottom-right"
    example_structure:
      elements: "N stages + N arrows (loop) + entry/exit arrows"
      typical_count: "10-24 elements"

  pipeline:
    description: "Strict sequential transformation (assembly line)"
    use_for:
      - "data pipelines"
      - "ETL processes"
      - "build stages"
      - "request processing"
    layout:
      direction: "left-to-right"
      stage_spacing: 160
      arrow_type: "straight"
      label_position: "above-arrow"
    example_structure:
      elements: "N stages + (N-1) arrows + labels"
      typical_count: "8-20 elements"

  cluster:
    description: "Loosely related items grouped in a boundary (cloud)"
    use_for:
      - "microservice clusters"
      - "feature sets"
      - "technology stacks"
    layout:
      boundary_type: "rectangle"
      boundary_stroke: "dashed"
      item_spacing: 60
      padding: 40
    example_structure:
      elements: "1 boundary + N items + optional title"
      typical_count: "6-20 elements"

  side_by_side:
    description: "Two or more alternatives shown in parallel (comparison)"
    use_for:
      - "before/after"
      - "option A vs option B"
      - "current vs proposed architecture"
    layout:
      column_count: 2
      column_spacing: 200
      row_alignment: "top"
      separator: "dashed-vertical-line"
    example_structure:
      elements: "2 groups + separator + labels"
      typical_count: "12-30 elements"

  gap_break:
    description: "Intentional separation showing a boundary or disconnection"
    use_for:
      - "security boundaries"
      - "network zones"
      - "team ownership boundaries"
      - "version boundaries"
    layout:
      gap_width: 80
      separator_type: "dashed-line"
      label_position: "center-of-gap"
    example_structure:
      elements: "2 groups + separator + boundary label"
      typical_count: "10-25 elements"

  lines_as_structure:
    description: "Arrows and connections ARE the primary content"
    use_for:
      - "sequence diagrams"
      - "network topologies"
      - "dependency graphs"
    layout:
      node_arrangement: "minimize-crossing"
      line_style_encodes: "meaning"
      line_thickness_encodes: "importance"
      line_color_encodes: "category"
    example_structure:
      elements: "N nodes + M connections (M >> N)"
      typical_count: "15-50 elements"

```
