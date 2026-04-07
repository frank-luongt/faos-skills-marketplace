---
name: excalidraw-wireframe
description: "Create website or app wireframes in Excalidraw format with proper layout, navigation, and component structure."
tags: [excalidraw, diagrams, wireframe, visualization]
---

# Create Wireframe - Workflow Instructions

```xml
<critical>The workflow execution engine is governed by: {project_root}/.faos/core/tasks/workflow.xml</critical>
<critical>You MUST have already loaded and processed: {installed_path}/workflow.yaml</critical>
<critical>This workflow creates website or app wireframes in Excalidraw format.</critical>

<workflow>

  <step n="0" goal="Contextual Analysis">
    <action>Review user's request and extract: wireframe type, fidelity level, screen count, device type, save location</action>
    <check if="ALL requirements clear"><action>Skip to Step 6</action></check>
  </step>

  <step n="1" goal="Assess Depth and Select Visual Pattern">
    <action>Based on the user's request, determine the wireframe depth level per {{helpers}} Depth Assessment:</action>
    <action>Present assessment:
      - Simple/Conceptual (10-20 elements): Single screen, low fidelity, layout exploration
      - Standard (20-40 elements): Few screens, medium fidelity, key interactions shown
      - Comprehensive/Technical (40-80 elements): Multiple screens, high fidelity, full user flow
    </action>
    <action>State your assessment: "This looks like a [depth] wireframe because [reason]"</action>
    <action>Select the best visual pattern from {{helpers}} Visual Pattern Library. For wireframes, common patterns are:
      - pipeline: For multi-step user flows (onboarding, checkout)
      - side-by-side: For responsive layouts showing desktop vs mobile
      - tree: For site maps or navigation hierarchy
      - cluster: For feature groupings within a single screen
    </action>
    <action>State: "I recommend the [pattern] pattern because it argues that [specific argument]"</action>
    <action>Ask: "Does this depth and pattern match your intent? (yes/adjust)"</action>
    <action>WAIT for confirmation</action>
    <check if="user says adjust">
      <action>Revise depth and/or pattern based on feedback</action>
      <action>Confirm revised choices</action>
    </check>
  </step>

  <step n="2" goal="Identify Wireframe Type" elicit="true">
    <action>Ask: "What type of wireframe do you need?"</action>
    <action>Present options:
      1. Website (Desktop)
      2. Mobile App (iOS/Android)
      3. Web App (Responsive)
      4. Tablet App
      5. Multi-platform
    </action>
    <action>WAIT for selection</action>
  </step>

  <step n="3" goal="Gather Requirements" elicit="true">
    <action>Ask fidelity level (Low/Medium/High)</action>
    <action>Ask screen count (Single/Few 2-3/Multiple 4-6/Many 7+)</action>
    <action>Ask device dimensions or use standard</action>
    <action>Ask save location</action>
  </step>

  <step n="4" goal="Check Theme" elicit="true">
    <action>Check for existing theme.json, ask to use if exists</action>
  </step>

  <step n="5" goal="Create Theme" elicit="true">
    <action>Ask: "Choose a wireframe style:"</action>
    <action>Present numbered options:
      1. Classic Wireframe
         - Background: #ffffff (white)
         - Container: #f5f5f5 (light gray)
         - Border: #9e9e9e (gray)
         - Text: #424242 (dark gray)

      2. High Contrast
         - Background: #ffffff (white)
         - Container: #eeeeee (light gray)
         - Border: #212121 (black)
         - Text: #000000 (black)

      3. Blueprint Style
         - Background: #1a237e (dark blue)
         - Container: #3949ab (blue)
         - Border: #7986cb (light blue)
         - Text: #ffffff (white)

      4. Custom - Define your own colors
    </action>
    <action>WAIT for selection</action>
    <action>Create theme.json based on selection, including semantic color mapping per {{helpers}}</action>
    <action>Confirm with user</action>
  </step>

  <step n="6" goal="Plan Wireframe Structure">
    <action>List all screens and their purposes</action>
    <action>Map navigation flow between screens</action>
    <action>Identify key UI elements for each screen</action>
    <action>Apply the selected visual pattern to the layout plan</action>
    <action>Show planned structure, confirm with user</action>
  </step>

  <step n="7" goal="Load Resources">
    <action>Load {{templates}} and extract `wireframe` section</action>
    <action>Load visual_patterns from {{templates}} for the selected pattern</action>
    <action>Load {{library}}</action>
    <action>Load theme.json</action>
    <action>Load {{helpers}}</action>
  </step>

  <step n="8" goal="Build Wireframe Elements">
    <critical>Apply the design philosophy from {{helpers}}: This wireframe must ARGUE a point about the user experience, not just show rectangles. Follow the selected visual pattern for layout.</critical>
    <critical>Use descriptive string IDs per {{helpers}} ID conventions (e.g., "hero_section_rect" not "rect_1")</critical>
    <critical>Prefer free-floating text for screen titles, annotations, and interaction notes. Target less than 30% containerized text per {{helpers}} Text Strategy.</critical>

    <substep>For Each Screen:
      - Create container/frame with descriptive ID
      - Add header section
      - Add content areas
      - Add navigation elements
      - Add interactive elements (buttons, inputs)
      - Add labels and annotations as free-floating text
    </substep>

    <substep>Build Order:
      1. Screen containers
      2. Layout sections (header, content, footer)
      3. Navigation elements
      4. Content blocks
      5. Interactive elements
      6. Labels and annotations (free-floating)
      7. Flow indicators (if multi-screen)
    </substep>

    <substep>Fidelity Guidelines:
      - Low: Basic shapes, minimal detail, placeholder text
      - Medium: More defined elements, some styling, representative content
      - High: Detailed elements, realistic sizing, actual content examples
    </substep>

    <substep>Apply semantic colors per {{helpers}}:
      - Primary for main interactive elements (CTA buttons, key inputs)
      - Secondary for supporting UI (headers, footers, navigation)
      - Accent for highlighted or featured content areas
      - Neutral for containers and boundaries
    </substep>

    <check if="planned element count exceeds 30">
      <action>Switch to section-by-section generation per {{helpers}} Section-by-Section Generation Protocol</action>
      <action>Identify logical sections (e.g., per screen, per region, or per breakpoint)</action>
      <action>Generate each section separately, using namespaced seeds</action>
      <action>After all sections: merge, validate cross-section arrows, verify boundElements</action>
    </check>

    <check if="depth is Comprehensive/Technical AND fidelity is High">
      <action>Add evidence artifacts per {{helpers}} Evidence Artifacts guide</action>
      <action>Embed real content examples or interaction details next to relevant components (max 3)</action>
    </check>
  </step>

  <step n="9" goal="Optimize and Save">
    <action>Strip unused elements and elements with isDeleted: true</action>
    <action>Save to {{default_output_file}}</action>
  </step>

  <step n="10" goal="Validate JSON Syntax">
    <critical>NEVER delete the file if validation fails - always fix syntax errors</critical>
    <action>Run: node -e "JSON.parse(require('fs').readFileSync('{{default_output_file}}', 'utf8')); console.log('Valid JSON')"</action>
    <check if="validation fails (exit code 1)">
      <action>Read the error message carefully - it shows the syntax error and position</action>
      <action>Open the file and navigate to the error location</action>
      <action>Fix the syntax error (add missing comma, bracket, or quote as indicated)</action>
      <action>Save the file</action>
      <action>Re-run validation with the same command</action>
      <action>Repeat until validation passes</action>
    </check>
    <action>Once validation passes, confirm with user</action>
  </step>

  <step n="11" goal="Visual Preview (Optional)" optional="true">
    <action>Load {{render_validate}}</action>
    <action>Run: python3 {{render_script}} {{default_output_file}}</action>
    <check if="exit code is 2 (Playwright not installed)">
      <action>Skip: "Playwright not available — skipping PNG preview. Wireframe is still valid."</action>
    </check>
    <check if="exit code is 0">
      <action>View the generated PNG file</action>
      <action>Audit against design vision: Does the wireframe ARGUE its point about the UX?</action>
      <action>Check: overlapping text, misaligned elements, spacing issues, color rendering</action>
      <check if="issues found">
        <action>Fix the JSON</action>
        <action>Re-render (max 3 cycles)</action>
      </check>
      <action>Ask: "Keep the PNG preview file? (yes/no)"</action>
    </check>
  </step>

  <step n="12" goal="Validate Content">
    <invoke-task>Validate against {{validation}}</invoke-task>
  </step>

</workflow>
```


## Quality Checklist

# Create Wireframe - Validation Checklist

## Design Quality

- [ ] Wireframe makes a clear visual argument about the user experience
- [ ] Isomorphism Test: Layout structure communicates hierarchy without labels
- [ ] Education Test: Unfamiliar viewer understands the interface in 30 seconds
- [ ] Visual pattern applied consistently (pipeline/side-by-side/tree/cluster)
- [ ] Depth level matches user intent (Simple/Standard/Comprehensive)
- [ ] Free-floating text used for screen titles and annotations (target <30% containerized)
- [ ] Semantic colors encode element roles (primary for CTA, secondary for navigation)
- [ ] All IDs are descriptive strings (e.g., "hero_section_rect" not "rect_1")

## Generation Quality

- [ ] If >30 elements: section-by-section generation used with namespaced seeds
- [ ] Descriptive string IDs follow `{component}_{role}_{type}` pattern
- [ ] Cross-section arrows have valid startBinding and endBinding
- [ ] Evidence artifacts present for High fidelity Comprehensive depth only (max 3)
- [ ] Evidence artifacts use dark rectangle styling with monospace text

## Layout Structure

- [ ] Screen dimensions appropriate for device type
- [ ] Grid alignment (20px) maintained
- [ ] Consistent spacing between UI elements
- [ ] Proper hierarchy (header, content, footer)

## UI Elements

- [ ] All interactive elements clearly marked
- [ ] Buttons, inputs, and controls properly sized
- [ ] Text labels readable and appropriately sized
- [ ] Navigation elements clearly indicated

## Fidelity

- [ ] Matches requested fidelity level (low/medium/high)
- [ ] Appropriate level of detail
- [ ] Placeholder content used where needed
- [ ] No unnecessary decoration for low-fidelity

## Annotations

- [ ] Key interactions annotated
- [ ] Flow indicators present if multi-screen
- [ ] Important notes included
- [ ] Element purposes clear

## Technical Quality

- [ ] All elements properly grouped
- [ ] Text elements have containerId
- [ ] Snapped to grid
- [ ] No elements with `isDeleted: true`
- [ ] JSON is valid
- [ ] File saved to correct location

