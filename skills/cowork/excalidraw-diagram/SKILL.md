---
name: excalidraw-diagram
description: "Create system architecture diagrams, ERDs, UML diagrams, or general technical diagrams in Excalidraw format."
tags: [excalidraw, diagrams, diagram, visualization]
---

# Create Diagram - Workflow Instructions

```xml
<critical>The workflow execution engine is governed by: {project_root}/.faos/core/tasks/workflow.xml</critical>
<critical>You MUST have already loaded and processed: {installed_path}/workflow.yaml</critical>
<critical>This workflow creates system architecture diagrams, ERDs, UML diagrams, or general technical diagrams in Excalidraw format.</critical>

<workflow>

  <step n="0" goal="Contextual Analysis">
    <action>Review user's request and extract: diagram type, components/entities, relationships, notation preferences</action>
    <check if="ALL requirements clear"><action>Skip to Step 6</action></check>
    <check if="SOME requirements clear"><action>Only ask about missing info in Steps 2-3</action></check>
  </step>

  <step n="1" goal="Assess Depth and Select Visual Pattern">
    <action>Based on the user's request, determine the diagram depth level per {{helpers}} Depth Assessment:</action>
    <action>Present assessment:
      - Simple/Conceptual (10-20 elements): High-level overview, explain a concept
      - Standard (20-40 elements): Document a system, show a process
      - Comprehensive/Technical (40-80 elements): Detailed audit, implementation guide
    </action>
    <action>State your assessment: "This looks like a [depth] diagram because [reason]"</action>
    <action>Select the best visual pattern from {{helpers}} Visual Pattern Library. For technical diagrams, common patterns are:
      - pipeline: For request processing chains, data transformation flows
      - tree: For class hierarchies, org structures, inheritance
      - fan-out: For API gateways, event distribution, load balancing
      - cluster: For microservice groups, technology stacks
      - lines-as-structure: For sequence diagrams, dependency graphs
      - side-by-side: For current vs proposed, option comparisons
    </action>
    <action>State: "I recommend the [pattern] pattern because it argues that [specific argument the diagram makes]"</action>
    <action>Ask: "Does this depth level and visual pattern match your intent? (yes/adjust)"</action>
    <action>WAIT for confirmation</action>
    <check if="user says adjust">
      <action>Revise depth and/or pattern based on feedback</action>
      <action>Confirm revised choices</action>
    </check>
  </step>

  <step n="2" goal="Identify Diagram Type" elicit="true">
    <action>Ask: "What type of technical diagram do you need?"</action>
    <action>Present options:
      1. System Architecture
      2. Entity-Relationship Diagram (ERD)
      3. UML Class Diagram
      4. UML Sequence Diagram
      5. UML Use Case Diagram
      6. Network Diagram
      7. Other
    </action>
    <action>WAIT for selection</action>
  </step>

  <step n="3" goal="Gather Requirements" elicit="true">
    <action>Ask: "Describe the components/entities and their relationships"</action>
    <action>Ask: "What notation standard? (Standard/Simplified/Strict UML-ERD)"</action>
    <action>WAIT for user input</action>
    <action>Summarize what will be included and confirm with user</action>
  </step>

  <step n="4" goal="Check for Existing Theme" elicit="true">
    <action>Check if theme.json exists at output location</action>
    <check if="exists"><action>Ask to use it, load if yes, else proceed to Step 5</action></check>
    <check if="not exists"><action>Proceed to Step 5</action></check>
  </step>

  <step n="5" goal="Create Theme" elicit="true">
    <action>Ask: "Choose a color scheme for your diagram:"</action>
    <action>Present numbered options:
      1. Professional
         - Component: #e3f2fd (light blue)
         - Database: #e8f5e9 (light green)
         - Service: #fff3e0 (light orange)
         - Border: #1976d2 (blue)

      2. Colorful
         - Component: #e1bee7 (light purple)
         - Database: #c5e1a5 (light lime)
         - Service: #ffccbc (light coral)
         - Border: #7b1fa2 (purple)

      3. Minimal
         - Component: #f5f5f5 (light gray)
         - Database: #eeeeee (gray)
         - Service: #e0e0e0 (medium gray)
         - Border: #616161 (dark gray)

      4. Custom - Define your own colors
    </action>
    <action>WAIT for selection</action>
    <action>Create theme.json based on selection, including semantic color mapping per {{helpers}}</action>
    <action>Show preview and confirm</action>
  </step>

  <step n="6" goal="Plan Diagram Structure">
    <action>List all components/entities</action>
    <action>Map all relationships</action>
    <action>Apply the selected visual pattern to the layout plan</action>
    <action>Show planned layout</action>
    <action>Ask: "Structure looks correct? (yes/no)"</action>
    <check if="no"><action>Adjust and repeat</action></check>
  </step>

  <step n="7" goal="Load Resources">
    <action>Load {{templates}} and extract `diagram` section</action>
    <action>Load visual_patterns from {{templates}} for the selected pattern</action>
    <action>Load {{library}}</action>
    <action>Load theme.json and merge with template</action>
    <action>Load {{helpers}} for guidelines</action>
  </step>

  <step n="8" goal="Build Diagram Elements">
    <critical>Apply the design philosophy from {{helpers}}: This diagram must ARGUE a point, not just DISPLAY components. Follow the selected visual pattern for layout.</critical>
    <critical>Use descriptive string IDs per {{helpers}} ID conventions (e.g., "auth_service_rect" not "rect_1")</critical>
    <critical>Prefer free-floating text for section labels, annotations, and callouts. Target less than 30% containerized text per {{helpers}} Text Strategy.</critical>

    <substep>For Each Component:
      - Generate descriptive string IDs per {{helpers}} ID conventions (e.g., auth_service_rect, auth_service_txt, auth_service_grp)
      - Create shape with groupIds
      - Calculate text width
      - Create text with containerId and matching groupIds
      - Add boundElements
      - Apply semantic color based on the component's role (primary/secondary/accent) per {{helpers}}
    </substep>

    <substep>For Each Connection:
      - Determine arrow type (straight/elbow)
      - Create with startBinding and endBinding
      - Update boundElements on both components
      - Apply semantic color to arrows (success for happy path, warning for conditional, danger for error)
    </substep>

    <substep>Add Free-Floating Text:
      - Section titles above each logical group
      - Annotations near complex relationships
      - Version or metadata notes in a corner
    </substep>

    <substep>Build Order by Type:
      - Architecture: Services -> Databases -> Connections -> Labels -> Annotations
      - ERD: Entities -> Attributes -> Relationships -> Cardinality -> Annotations
      - UML Class: Classes -> Attributes -> Methods -> Relationships -> Notes
      - UML Sequence: Actors -> Lifelines -> Messages -> Returns -> Notes
      - UML Use Case: Actors -> Use Cases -> Relationships -> Notes
    </substep>

    <substep>Alignment:
      - Snap to 20px grid
      - Space: 40px between components, 60px between sections
      - Follow the selected visual pattern's layout rules from {{templates}}
    </substep>

    <check if="planned element count exceeds 30">
      <action>Switch to section-by-section generation per {{helpers}} Section-by-Section Generation Protocol</action>
      <action>Identify logical sections based on diagram type (e.g., by service layer for architecture)</action>
      <action>Generate each section separately, using namespaced seeds</action>
      <action>After all sections: merge, validate cross-section arrows, verify boundElements</action>
    </check>

    <check if="depth is Comprehensive/Technical">
      <action>Add evidence artifacts per {{helpers}} Evidence Artifacts guide</action>
      <action>Embed real code/JSON/API shapes next to relevant components (max 5)</action>
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
    <action>Once validation passes, confirm: "Diagram created at {{default_output_file}}. Open to view?"</action>
  </step>

  <step n="11" goal="Visual Preview (Optional)" optional="true">
    <action>Load {{render_validate}}</action>
    <action>Run: python3 {{render_script}} {{default_output_file}}</action>
    <check if="exit code is 2 (Playwright not installed)">
      <action>Skip: "Playwright not available — skipping PNG preview. Diagram is still valid."</action>
    </check>
    <check if="exit code is 0">
      <action>View the generated PNG file</action>
      <action>Audit against design vision: Does the diagram ARGUE its point?</action>
      <action>Check: overlapping text, misaligned arrows, spacing issues, color rendering</action>
      <check if="issues found">
        <action>Fix the JSON</action>
        <action>Re-render (max 3 cycles)</action>
      </check>
      <action>Ask: "Keep the PNG preview file? (yes/no)"</action>
    </check>
  </step>

  <step n="12" goal="Validate Content">
    <invoke-task>Validate against {{validation}} using {faos}/core/tasks/validate-workflow.xml</invoke-task>
  </step>

</workflow>
```


## Quality Checklist

# Create Diagram - Validation Checklist

## Design Quality

- [ ] Diagram makes a clear visual argument (not just displaying components)
- [ ] Isomorphism Test: Structure communicates without labels
- [ ] Education Test: Unfamiliar viewer learns something in 30 seconds
- [ ] Visual pattern applied consistently throughout the diagram
- [ ] Depth level matches user intent (Simple/Standard/Comprehensive)
- [ ] Free-floating text used for labels and annotations (target <30% containerized)
- [ ] Semantic colors encode element roles (primary/secondary/accent/success/warning/danger)
- [ ] All IDs are descriptive strings (e.g., "auth_service_rect" not "rect_1")

## Generation Quality

- [ ] If >30 elements: section-by-section generation used with namespaced seeds
- [ ] Descriptive string IDs follow `{component}_{role}_{type}` pattern
- [ ] Cross-section arrows have valid startBinding and endBinding
- [ ] Evidence artifacts present for Comprehensive/Technical depth (if applicable)
- [ ] Evidence artifacts use dark rectangle styling with monospace text
- [ ] Element count within limit (under 80)

## Element Structure

- [ ] All components with labels have matching `groupIds`
- [ ] All text elements have `containerId` pointing to parent component
- [ ] Text width calculated properly (no cutoff)
- [ ] Text alignment appropriate for diagram type

## Layout and Alignment

- [ ] All elements snapped to 20px grid
- [ ] Component spacing consistent (40px/60px)
- [ ] Hierarchical alignment maintained
- [ ] No overlapping elements

## Connections

- [ ] All arrows have `startBinding` and `endBinding`
- [ ] `boundElements` array updated on connected components
- [ ] Arrow routing avoids overlaps
- [ ] Relationship types clearly indicated

## Notation and Standards

- [ ] Follows specified notation standard (UML/ERD/etc)
- [ ] Symbols used correctly
- [ ] Cardinality/multiplicity shown where needed
- [ ] Labels and annotations clear

## Theme and Styling

- [ ] Theme colors applied consistently
- [ ] Component types visually distinguishable
- [ ] Text is readable
- [ ] Professional appearance

## Output Quality

- [ ] Element count under 80
- [ ] No elements with `isDeleted: true`
- [ ] JSON is valid
- [ ] File saved to correct location

