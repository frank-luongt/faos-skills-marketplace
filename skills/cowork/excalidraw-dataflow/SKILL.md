---
name: excalidraw-dataflow
description: "Create data flow diagrams (DFD) in Excalidraw format showing data movement between systems and processes."
tags: [excalidraw, diagrams, dataflow, visualization]
---

# Create Data Flow Diagram - Workflow Instructions

```xml
<critical>The workflow execution engine is governed by: {project_root}/.faos/core/tasks/workflow.xml</critical>
<critical>You MUST have already loaded and processed: {installed_path}/workflow.yaml</critical>
<critical>This workflow creates data flow diagrams (DFD) in Excalidraw format.</critical>

<workflow>

  <step n="0" goal="Contextual Analysis">
    <action>Review user's request and extract: DFD level, processes, data stores, external entities</action>
    <check if="ALL requirements clear"><action>Skip to Step 5</action></check>
  </step>

  <step n="1" goal="Assess Depth and Select Visual Pattern">
    <action>Based on the user's request, determine the DFD depth level per {{helpers}} Depth Assessment:</action>
    <action>Present assessment:
      - Simple/Conceptual (10-20 elements): Context diagram (Level 0), single process with boundaries
      - Standard (20-40 elements): Level 1 DFD, major processes and data flows
      - Comprehensive/Technical (40-80 elements): Level 2 DFD, detailed sub-processes
    </action>
    <action>State your assessment: "This looks like a [depth] DFD because [reason]"</action>
    <action>Select the best visual pattern from {{helpers}} Visual Pattern Library. For DFDs, common patterns are:
      - fan-out: For processes distributing data to multiple stores or entities
      - convergence: For data aggregation from multiple sources into a process
      - pipeline: For linear data transformations (ETL-style flows)
      - gap/break: For showing system boundaries in context diagrams
      - cluster: For grouping related processes at higher DFD levels
    </action>
    <action>State: "I recommend the [pattern] pattern because it argues that [specific argument]"</action>
    <action>Ask: "Does this depth and pattern match your intent? (yes/adjust)"</action>
    <action>WAIT for confirmation</action>
    <check if="user says adjust">
      <action>Revise depth and/or pattern based on feedback</action>
      <action>Confirm revised choices</action>
    </check>
  </step>

  <step n="2" goal="Identify DFD Level" elicit="true">
    <action>Ask: "What level of DFD do you need?"</action>
    <action>Present options:
      1. Context Diagram (Level 0) - Single process showing system boundaries
      2. Level 1 DFD - Major processes and data flows
      3. Level 2 DFD - Detailed sub-processes
      4. Custom - Specify your requirements
    </action>
    <action>WAIT for selection</action>
  </step>

  <step n="3" goal="Gather Requirements" elicit="true">
    <action>Ask: "Describe the processes, data stores, and external entities in your system"</action>
    <action>WAIT for user description</action>
    <action>Summarize what will be included and confirm with user</action>
  </step>

  <step n="4" goal="Theme Setup" elicit="true">
    <action>Check for existing theme.json, ask to use if exists</action>
    <check if="no existing theme">
      <action>Ask: "Choose a DFD color scheme:"</action>
      <action>Present numbered options:
        1. Standard DFD
           - Process: #e3f2fd (light blue)
           - Data Store: #e8f5e9 (light green)
           - External Entity: #f3e5f5 (light purple)
           - Border: #1976d2 (blue)

        2. Colorful DFD
           - Process: #fff9c4 (light yellow)
           - Data Store: #c5e1a5 (light lime)
           - External Entity: #ffccbc (light coral)
           - Border: #f57c00 (orange)

        3. Minimal DFD
           - Process: #f5f5f5 (light gray)
           - Data Store: #eeeeee (gray)
           - External Entity: #e0e0e0 (medium gray)
           - Border: #616161 (dark gray)

        4. Custom - Define your own colors
      </action>
      <action>WAIT for selection</action>
      <action>Create theme.json based on selection, including semantic color mapping per {{helpers}}</action>
    </check>
  </step>

  <step n="5" goal="Plan DFD Structure">
    <action>List all processes with numbers (1.0, 2.0, etc.)</action>
    <action>List all data stores (D1, D2, etc.)</action>
    <action>List all external entities</action>
    <action>Map all data flows with labels</action>
    <action>Apply the selected visual pattern to the layout plan</action>
    <action>Show planned structure, confirm with user</action>
  </step>

  <step n="6" goal="Load Resources">
    <action>Load {{templates}} and extract `dataflow` section</action>
    <action>Load visual_patterns from {{templates}} for the selected pattern</action>
    <action>Load {{library}}</action>
    <action>Load theme.json</action>
    <action>Load {{helpers}}</action>
  </step>

  <step n="7" goal="Build DFD Elements">
    <critical>Apply the design philosophy from {{helpers}}: This DFD must ARGUE a point about data flow, not just list processes. Follow the selected visual pattern for layout.</critical>
    <critical>Use descriptive string IDs per {{helpers}} ID conventions (e.g., "validate_order_ellipse" not "proc_1")</critical>
    <critical>Prefer free-floating text for data flow labels, boundary annotations, and DFD level indicators. Target less than 30% containerized text per {{helpers}} Text Strategy.</critical>

    <substep>Build Order:
      1. External entities (rectangles, bold border) with descriptive IDs
      2. Processes (circles/ellipses with numbers) with descriptive IDs
      3. Data stores (parallel lines or rectangles) with descriptive IDs
      4. Data flows (labeled arrows) with descriptive IDs
      5. Free-floating annotations and boundary labels
    </substep>

    <substep>DFD Rules:
      - Processes: Numbered (1.0, 2.0), verb phrases
      - Data stores: Named (D1, D2), noun phrases
      - External entities: Named, noun phrases
      - Data flows: Labeled with data names, arrows show direction
      - No direct flow between external entities
      - No direct flow between data stores
      - Apply semantic color based on element role per {{helpers}}
    </substep>

    <substep>Layout:
      - External entities at edges
      - Processes in center
      - Data stores between processes
      - Minimize crossing flows
      - Left-to-right or top-to-bottom flow
      - Follow the selected visual pattern's layout rules from {{templates}}
    </substep>

    <check if="planned element count exceeds 30">
      <action>Switch to section-by-section generation per {{helpers}} Section-by-Section Generation Protocol</action>
      <action>Identify logical sections (e.g., external entities, core processes, data stores)</action>
      <action>Generate each section separately, using namespaced seeds</action>
      <action>After all sections: merge, validate cross-section arrows, verify boundElements</action>
    </check>

    <check if="depth is Comprehensive/Technical">
      <action>Add evidence artifacts per {{helpers}} Evidence Artifacts guide</action>
      <action>Embed real data payload shapes or query examples next to relevant processes (max 5)</action>
    </check>
  </step>

  <step n="8" goal="Optimize and Save">
    <action>Verify DFD rules compliance</action>
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
    <action>Once validation passes, confirm with user</action>
  </step>

  <step n="10" goal="Visual Preview (Optional)" optional="true">
    <action>Load {{render_validate}}</action>
    <action>Run: python3 {{render_script}} {{default_output_file}}</action>
    <check if="exit code is 2 (Playwright not installed)">
      <action>Skip: "Playwright not available — skipping PNG preview. DFD is still valid."</action>
    </check>
    <check if="exit code is 0">
      <action>View the generated PNG file</action>
      <action>Audit against design vision: Does the DFD ARGUE its point about data flow?</action>
      <action>Check: overlapping text, misaligned arrows, spacing issues, color rendering</action>
      <check if="issues found">
        <action>Fix the JSON</action>
        <action>Re-render (max 3 cycles)</action>
      </check>
      <action>Ask: "Keep the PNG preview file? (yes/no)"</action>
    </check>
  </step>

  <step n="11" goal="Validate Content">
    <invoke-task>Validate against {{validation}} using {faos}/core/tasks/validate-workflow.xml</invoke-task>
  </step>

</workflow>
```


## Quality Checklist

# Create Data Flow Diagram - Validation Checklist

## Design Quality

- [ ] DFD makes a clear visual argument about data flow (not just listing processes)
- [ ] Isomorphism Test: Data flow structure communicates without labels
- [ ] Education Test: Unfamiliar viewer understands data movement in 30 seconds
- [ ] Visual pattern applied consistently (fan-out/convergence/pipeline/gap/cluster)
- [ ] Depth level matches user intent (Context/Level 1/Level 2)
- [ ] Free-floating text used for flow labels and boundary annotations (target <30% containerized)
- [ ] Semantic colors encode element roles (primary for processes, secondary for stores)
- [ ] All IDs are descriptive strings (e.g., "validate_order_ellipse" not "proc_1")

## Generation Quality

- [ ] If >30 elements: section-by-section generation used with namespaced seeds
- [ ] Descriptive string IDs follow `{component}_{role}_{type}` pattern
- [ ] Cross-section arrows have valid startBinding and endBinding
- [ ] Evidence artifacts present for Comprehensive/Technical depth (if applicable)
- [ ] Evidence artifacts use dark rectangle styling with monospace text

## DFD Notation

- [ ] Processes shown as circles/ellipses
- [ ] Data stores shown as parallel lines or rectangles
- [ ] External entities shown as rectangles
- [ ] Data flows shown as labeled arrows
- [ ] Follows standard DFD notation

## Structure

- [ ] All processes numbered correctly
- [ ] All data flows labeled with data names
- [ ] All data stores named appropriately
- [ ] External entities clearly identified

## Completeness

- [ ] All inputs and outputs accounted for
- [ ] No orphaned processes (unconnected)
- [ ] Data conservation maintained
- [ ] Level appropriate (context/level 0/level 1)

## Layout

- [ ] Logical flow direction (left to right, top to bottom)
- [ ] No crossing data flows where avoidable
- [ ] Balanced layout
- [ ] Grid alignment maintained

## Technical Quality

- [ ] All elements properly grouped
- [ ] Arrows have proper bindings
- [ ] Text readable and properly sized
- [ ] No elements with `isDeleted: true`
- [ ] JSON is valid
- [ ] File saved to correct location

