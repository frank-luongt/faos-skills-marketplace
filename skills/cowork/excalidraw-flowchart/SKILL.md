---
name: excalidraw-flowchart
description: "Create flowchart visualizations in Excalidraw format for processes, pipelines, or logic flows."
tags: [excalidraw, diagrams, flowchart, visualization]
---

# Create Flowchart - Workflow Instructions

```xml
<critical>The workflow execution engine is governed by: {project_root}/.faos/core/tasks/workflow.xml</critical>
<critical>You MUST have already loaded and processed: {installed_path}/workflow.yaml</critical>
<critical>This workflow creates a flowchart visualization in Excalidraw format for processes, pipelines, or logic flows.</critical>

<workflow>

  <step n="0" goal="Contextual Analysis (Smart Elicitation)">
    <critical>Before asking any questions, analyze what the user has already told you</critical>

    <action>Review the user's initial request and conversation history</action>
    <action>Extract any mentioned: flowchart type, complexity, decision points, save location</action>

    <check if="ALL requirements are clear from context">
      <action>Summarize your understanding</action>
      <action>Skip directly to Step 5 (Plan Flowchart Layout)</action>
    </check>

    <check if="SOME requirements are clear">
      <action>Note what you already know</action>
      <action>Only ask about missing information in Step 2</action>
    </check>

    <check if="requirements are unclear or minimal">
      <action>Proceed with full elicitation in Step 2</action>
    </check>
  </step>

  <step n="1" goal="Assess Depth and Select Visual Pattern">
    <action>Based on the user's request, determine the flowchart depth level per {{helpers}} Depth Assessment:</action>
    <action>Present assessment:
      - Simple/Conceptual (10-20 elements): Quick process, 3-5 steps, few decisions
      - Standard (20-40 elements): Standard workflow, 6-10 steps, some branching
      - Comprehensive/Technical (40-50 elements): Detailed process, 11-20 steps, multiple decision points
    </action>
    <action>State your assessment: "This looks like a [depth] flowchart because [reason]"</action>
    <action>Select the best visual pattern from {{helpers}} Visual Pattern Library. For flowcharts, common patterns are:
      - pipeline: Default for linear sequential flows
      - cycle: For processes with feedback loops or retry logic
      - tree: For decision-heavy flows that branch significantly
      - fan-out: For processes that distribute to parallel paths
      - convergence: For parallel paths that merge back together
    </action>
    <action>State: "I recommend the [pattern] pattern because it argues that [specific argument]"</action>
    <action>Ask: "Does this depth and pattern match your intent? (yes/adjust)"</action>
    <action>WAIT for confirmation</action>
    <check if="user says adjust">
      <action>Revise depth and/or pattern based on feedback</action>
      <action>Confirm revised choices</action>
    </check>
  </step>

  <step n="2" goal="Gather Requirements" elicit="true">
    <action>Ask Question 1: "What type of process flow do you need to visualize?"</action>
    <action>Present numbered options:
      1. Business Process Flow - Document business workflows, approval processes, or operational procedures
      2. Algorithm/Logic Flow - Visualize code logic, decision trees, or computational processes
      3. User Journey Flow - Map user interactions, navigation paths, or experience flows
      4. Data Processing Pipeline - Show data transformation, ETL processes, or processing stages
      5. Other - Describe your specific flowchart needs
    </action>
    <action>WAIT for user selection (1-5)</action>

    <action>Ask Question 2: "How many main steps are in this flow?"</action>
    <action>Present numbered options:
      1. Simple (3-5 steps) - Quick process with few decision points
      2. Medium (6-10 steps) - Standard workflow with some branching
      3. Complex (11-20 steps) - Detailed process with multiple decision points
      4. Very Complex (20+ steps) - Comprehensive workflow requiring careful layout
    </action>
    <action>WAIT for user selection (1-4)</action>
    <action>Store selection in {{complexity}}</action>

    <action>Ask Question 3: "Does your flow include decision points (yes/no branches)?"</action>
    <action>Present numbered options:
      1. No decisions - Linear flow from start to end
      2. Few decisions (1-2) - Simple branching with yes/no paths
      3. Multiple decisions (3-5) - Several conditional branches
      4. Complex decisions (6+) - Extensive branching logic
    </action>
    <action>WAIT for user selection (1-4)</action>
    <action>Store selection in {{decision_points}}</action>

    <action>Ask Question 4: "Where should the flowchart be saved?"</action>
    <action>Present numbered options:
      1. Default location - docs/flowcharts/[auto-generated-name].excalidraw
      2. Custom path - Specify your own file path
      3. Project root - Save in main project directory
      4. Specific folder - Choose from existing folders
    </action>
    <action>WAIT for user selection (1-4)</action>
    <check if="selection is 2 or 4">
      <action>Ask for specific path</action>
      <action>WAIT for user input</action>
    </check>
    <action>Store final path in {{default_output_file}}</action>
  </step>

  <step n="3" goal="Check for Existing Theme" elicit="true">
    <action>Check if theme.json exists at output location</action>
    <check if="theme.json exists">
      <action>Ask: "Found existing theme. Use it? (yes/no)"</action>
      <action>WAIT for user response</action>
      <check if="user says yes">
        <action>Load and use existing theme</action>
        <action>Skip to Step 5</action>
      </check>
      <check if="user says no">
        <action>Proceed to Step 4</action>
      </check>
    </check>
    <check if="theme.json does not exist">
      <action>Proceed to Step 4</action>
    </check>
  </step>

  <step n="4" goal="Create Theme" elicit="true">
    <action>Ask: "Let's create a theme for your flowchart. Choose a color scheme:"</action>
    <action>Present numbered options:
      1. Professional Blue
         - Primary Fill: #e3f2fd (light blue)
         - Accent/Border: #1976d2 (blue)
         - Decision: #fff3e0 (light orange)
         - Text: #1e1e1e (dark gray)

      2. Success Green
         - Primary Fill: #e8f5e9 (light green)
         - Accent/Border: #388e3c (green)
         - Decision: #fff9c4 (light yellow)
         - Text: #1e1e1e (dark gray)

      3. Neutral Gray
         - Primary Fill: #f5f5f5 (light gray)
         - Accent/Border: #616161 (gray)
         - Decision: #e0e0e0 (medium gray)
         - Text: #1e1e1e (dark gray)

      4. Warm Orange
         - Primary Fill: #fff3e0 (light orange)
         - Accent/Border: #f57c00 (orange)
         - Decision: #ffe0b2 (peach)
         - Text: #1e1e1e (dark gray)

      5. Custom Colors - Define your own color palette
    </action>
    <action>WAIT for user selection (1-5)</action>
    <action>Store selection in {{theme_choice}}</action>

    <check if="selection is 5 (Custom)">
      <action>Ask: "Primary fill color (hex code)?"</action>
      <action>WAIT for user input</action>
      <action>Store in {{custom_colors.primary_fill}}</action>
      <action>Ask: "Accent/border color (hex code)?"</action>
      <action>WAIT for user input</action>
      <action>Store in {{custom_colors.accent}}</action>
      <action>Ask: "Decision color (hex code)?"</action>
      <action>WAIT for user input</action>
      <action>Store in {{custom_colors.decision}}</action>
    </check>

    <action>Create theme.json with selected colors, including semantic color mapping per {{helpers}}</action>
    <action>Show theme preview with all colors</action>
    <action>Ask: "Theme looks good?"</action>
    <action>Present numbered options:
      1. Yes, use this theme - Proceed with theme
      2. No, adjust colors - Modify color selections
      3. Start over - Choose different preset
    </action>
    <action>WAIT for selection (1-3)</action>
    <check if="selection is 2 or 3">
      <action>Repeat Step 4</action>
    </check>
  </step>

  <step n="5" goal="Plan Flowchart Layout">
    <action>List all steps and decision points based on gathered requirements</action>
    <action>Apply the selected visual pattern to the layout plan</action>
    <action>Show user the planned structure</action>
    <action>Ask: "Structure looks correct? (yes/no)"</action>
    <action>WAIT for user response</action>
    <check if="user says no">
      <action>Adjust structure based on feedback</action>
      <action>Repeat this step</action>
    </check>
  </step>

  <step n="6" goal="Load Template and Resources">
    <action>Load {{templates}} file</action>
    <action>Extract `flowchart` section from YAML</action>
    <action>Load visual_patterns from {{templates}} for the selected pattern</action>
    <action>Load {{library}} file</action>
    <action>Load theme.json and merge colors with template</action>
    <action>Load {{helpers}} for element creation guidelines</action>
  </step>

  <step n="7" goal="Build Flowchart Elements">
    <critical>Apply the design philosophy from {{helpers}}: This flowchart must ARGUE a point about the process, not just list steps. Follow the selected visual pattern for layout.</critical>
    <critical>Use descriptive string IDs per {{helpers}} ID conventions (e.g., "validate_input_rect" not "rect_3")</critical>
    <critical>Prefer free-floating text for section labels, annotations, and flow descriptions. Target less than 30% containerized text per {{helpers}} Text Strategy.</critical>

    <action>Build ONE section at a time following these rules:</action>

    <substep>For Each Shape with Label:
      1. Generate descriptive string IDs per {{helpers}} ID conventions (e.g., start_flow_ellipse, start_flow_txt, start_flow_grp)
      2. Create shape with groupIds: [group-id]
      3. Calculate text width: (text.length * fontSize * 0.6) + 20, round to nearest 10
      4. Create text element with:
         - containerId: shape-id
         - groupIds: [group-id] (SAME as shape)
         - textAlign: "center"
         - verticalAlign: "middle"
         - width: calculated width
      5. Add boundElements to shape referencing text
      6. Apply semantic color based on element role (primary for process, accent for decision, success for happy path)
    </substep>

    <substep>For Each Arrow:
      1. Determine arrow type needed:
         - Straight: For forward flow (left-to-right, top-to-bottom)
         - Elbow: For upward flow, backward flow, or complex routing
      2. Create arrow with startBinding and endBinding
      3. Set startBinding.elementId to source shape ID
      4. Set endBinding.elementId to target shape ID
      5. Set gap: 10 for both bindings
      6. If elbow arrow, add intermediate points for direction changes
      7. Update boundElements on both connected shapes
      8. Apply semantic color (success for main flow, warning for conditional branches, danger for error paths)
    </substep>

    <substep>Add Free-Floating Text:
      - Section labels above logical groups of steps
      - "Yes"/"No" annotations near decision branch arrows (as free-floating text near the arrow)
      - Phase or stage descriptions
    </substep>

    <substep>Alignment:
      - Snap all x, y to 20px grid
      - Align shapes vertically (same x for vertical flow)
      - Space elements: 60px between shapes
      - Follow the selected visual pattern's layout rules from {{templates}}
    </substep>

    <substep>Build Order:
      1. Start point (circle) with label
      2. Each process step (rectangle) with label
      3. Each decision point (diamond) with label
      4. End point (circle) with label
      5. Connect all with bound arrows
      6. Add free-floating annotations and section labels
    </substep>

    <check if="planned element count exceeds 30">
      <action>Switch to section-by-section generation per {{helpers}} Section-by-Section Generation Protocol</action>
      <action>Identify logical sections (e.g., by flow phase, swim lane, or decision branch)</action>
      <action>Generate each section separately, using namespaced seeds</action>
      <action>After all sections: merge, validate cross-section arrows, verify boundElements</action>
    </check>

    <check if="depth is Comprehensive/Technical">
      <action>Add evidence artifacts per {{helpers}} Evidence Artifacts guide</action>
      <action>Embed real code/JSON/data shapes next to relevant steps (max 5)</action>
    </check>
  </step>

  <step n="8" goal="Optimize and Save">
    <action>Strip unused elements and elements with isDeleted: true</action>
    <action>Save to {{default_output_file}}</action>
  </step>

  <step n="9" goal="Validate JSON Syntax">
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
    <action>Once validation passes, confirm with user: "Flowchart created at {{default_output_file}}. Open to view?"</action>
  </step>

  <step n="10" goal="Visual Preview (Optional)" optional="true">
    <action>Load {{render_validate}}</action>
    <action>Run: python3 {{render_script}} {{default_output_file}}</action>
    <check if="exit code is 2 (Playwright not installed)">
      <action>Skip: "Playwright not available — skipping PNG preview. Flowchart is still valid."</action>
    </check>
    <check if="exit code is 0">
      <action>View the generated PNG file</action>
      <action>Audit against design vision: Does the flowchart ARGUE its point about the process?</action>
      <action>Check: overlapping text, misaligned arrows, spacing issues, color rendering</action>
      <check if="issues found">
        <action>Fix the JSON</action>
        <action>Re-render (max 3 cycles)</action>
      </check>
      <action>Ask: "Keep the PNG preview file? (yes/no)"</action>
    </check>
  </step>

  <step n="11" goal="Validate Content">
    <invoke-task>Validate against checklist at {{validation}} using {faos}/core/tasks/validate-workflow.xml</invoke-task>
  </step>

</workflow>
```


## Quality Checklist

# Create Flowchart - Validation Checklist

## Design Quality

- [ ] Flowchart makes a clear visual argument about the process (not just listing steps)
- [ ] Isomorphism Test: Flow structure communicates without labels
- [ ] Education Test: Unfamiliar viewer understands the process in 30 seconds
- [ ] Visual pattern applied consistently (pipeline/cycle/tree/fan-out/convergence)
- [ ] Depth level matches user intent (Simple/Standard/Comprehensive)
- [ ] Free-floating text used for section labels and annotations (target <30% containerized)
- [ ] Semantic colors encode element roles (primary for process, accent for decision)
- [ ] All IDs are descriptive strings (e.g., "validate_input_rect" not "rect_3")

## Generation Quality

- [ ] If >30 elements: section-by-section generation used with namespaced seeds
- [ ] Descriptive string IDs follow `{component}_{role}_{type}` pattern
- [ ] Cross-section arrows have valid startBinding and endBinding
- [ ] Evidence artifacts present for Comprehensive/Technical depth (if applicable)
- [ ] Evidence artifacts use dark rectangle styling with monospace text
- [ ] Element count within limit (under 50)

## Element Structure

- [ ] All shapes with labels have matching `groupIds`
- [ ] All text elements have `containerId` pointing to parent shape
- [ ] Text width calculated properly (no cutoff)
- [ ] Text alignment set (`textAlign` + `verticalAlign`)

## Layout and Alignment

- [ ] All elements snapped to 20px grid
- [ ] Consistent spacing between elements (60px minimum)
- [ ] Vertical alignment maintained for flow direction
- [ ] No overlapping elements

## Connections

- [ ] All arrows have `startBinding` and `endBinding`
- [ ] `boundElements` array updated on connected shapes
- [ ] Arrow types appropriate (straight for forward, elbow for backward/upward)
- [ ] Gap set to 10 for all bindings

## Theme and Styling

- [ ] Theme colors applied consistently
- [ ] All shapes use theme primary fill color
- [ ] All borders use theme accent color
- [ ] Text color is readable (#1e1e1e)

## Composition

- [ ] Element count under 50
- [ ] Library components referenced where possible
- [ ] No duplicate element definitions

## Output Quality

- [ ] No elements with `isDeleted: true`
- [ ] JSON is valid
- [ ] File saved to correct location

## Functional Requirements

- [ ] Start point clearly marked
- [ ] End point clearly marked
- [ ] All process steps labeled
- [ ] Decision points use diamond shapes
- [ ] Flow direction is clear and logical

