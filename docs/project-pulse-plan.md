# Project Pulse dashboard implementation plan

## Goal and delivery boundary

Build a lightweight, static **Project Pulse** dashboard that lets contributors
quickly scan active work: project name, owner, current status, recent activity,
and priority/risk signal. The result should be a polished, responsive browser
view rather than a directory listing, with all visible project cards generated
from local JSON data.

The implementation deliverables are:

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`
- `.vscode/launch.json`

This repository has no application framework or package manifest. The dashboard
should therefore use plain semantic HTML, CSS, and a small inline browser script
in `index.html`; it must not introduce build tooling or dependencies. The
existing `.vscode/tasks.json` is unrelated to previewing the dashboard and
should remain unchanged.

## Proposed product and design specification

### Information architecture

The page should have the document and visible page title **Project Pulse** and
these regions, in order:

1. A `<header>`/hero with the title and a brief contributor-oriented statement
   of purpose.
2. A compact summary row, populated or derived from the same data, showing the
   total active projects and useful status/priority counts.
3. A clearly labelled project section containing status filter controls:
   **All**, plus one control for each available status.
4. A live count/summary of the selected projects and a card grid. Every project
   is represented by one visible `.project-card`.
5. A concise empty state when a filter has no matches, and a visible,
   understandable data-load error state if JSON cannot be retrieved or is
   malformed.

Each card should make its hierarchy easy to scan: project name first, owner as
supporting metadata, then a status badge, a priority/risk badge or label, and a
plain-language “Recent activity” field. Status and priority must never be
communicated solely by color.

### Visual direction and responsive behavior

Use a calm, polished dashboard treatment: a restrained neutral page background,
a high-contrast heading, one consistent accent color, and distinct semantic
status/priority colors. Cards should use comfortable whitespace, rounded
corners, subtle `box-shadow`, and a visible border or focus treatment. Use a
system font stack and readable line heights rather than external assets.

The main container should be centered with a sensible maximum width. The
`.dashboard` layout and card grid should be single-column on narrow viewports,
then use responsive `minmax` grid columns as space permits. Filter controls
must wrap instead of causing horizontal scrolling. At a small-screen breakpoint,
retain at least comfortably sized targets and avoid relying on hover for
essential information.

### User-facing behavior

On initial load, the page fetches `project-data.json`, derives the summary and
available filters, and renders all projects. Selecting a status filter updates
the selected state, visible card list, and result count without a reload.
Keyboard users can reach and activate each native button, and the current filter
is exposed with an appropriate selected/pressed state. Project text is inserted
as text, not HTML, so data remains display-only. If loading fails, the page
shows an error message that explains that project data could not be loaded,
rather than silently rendering an empty dashboard.

## Data contract

`app/project-data.json` must be strict JSON with this top-level shape:

```json
{
  "projects": [
    {
      "name": "Contributor Onboarding",
      "owner": "Mona",
      "status": "On track",
      "recentActivity": "Updated the contributor starter checklist.",
      "priority": "High"
    }
  ]
}
```

Requirements for the data set:

- `projects` is a non-empty array with multiple (at least three) realistic
  active projects.
- Every item has non-empty string values for exactly the required display
  fields: `name`, `owner`, `status`, `recentActivity`, and `priority`.
- Use a small, consistent vocabulary for status (for example, `On track`,
  `At risk`, and `Blocked`) and priority (for example, `High`, `Medium`, and
  `Low`) so the UI can apply predictable badge modifier classes and filters.
- The renderer should validate that the response has a `projects` array and
  gracefully report a load error for an invalid shape. It may skip or flag
  individual incomplete entries; it must not invent project cards independent
  of the JSON.

## File assignments and responsibilities

| File or responsibility | Owner | Required work | Depends on |
| --- | --- | --- | --- |
| Design direction, IA, accessibility review, and CSS-hook guidance | **Designer** | Supply the approved layout above; specify semantic regions, card field order, status/priority treatment, responsive rules, empty/error states, and stable hooks including `.dashboard` and `.project-card`. Review the implemented result. | Brief and this plan |
| `app/project-data.json` | **Coder** | Create the data fixture that follows the data contract and provides varied statuses/priorities for filtering and visual treatments. | Approved data contract |
| `app/index.html` | **Coder** | Create semantic page structure, load `styles.css`, fetch and validate `project-data.json`, render cards with the `project-card` class, render summary/filter/error/empty states, and implement the filter behavior. | Designer's hooks/IA and finalized JSON shape |
| `app/styles.css` | **Coder** | Implement the approved visual system, including `.dashboard`, `.project-card`, badge states, focus styles, loading/error/empty states, and responsive grid/layout behavior. | Designer's direction and HTML class/element hooks |
| `.vscode/launch.json` | **Coder** | Create the deterministic dashboard preview configuration described below. | None beyond the required `app/` path |
| Integration and acceptance review | **Orchestrator** | Check that independently produced files agree on paths, class hooks, data keys, and launch behavior; request corrections within the responsible owner’s scope. | All implementation files |

The Designer should not edit the implementation files in the parallel build
phase. Keeping its output as an explicit design handoff avoids overlapping
write scopes while preserving its ownership of UX and accessibility decisions.
The Coder owns all four implementation files and is responsible for applying
that handoff consistently.

## Launch configuration specification

`.vscode/launch.json` must be comment-free, strict JSON with `"version":
"0.2.0"` and one configuration named exactly **Run Project Pulse Dashboard**.
Use a terminal launch configuration that runs exactly `python3 -m http.server
5500`, with:

- `"cwd": "${workspaceFolder}/app"` so the server root is the application
  directory;
- `serverReadyAction` that recognizes the Python server port; and
- `"uriFormat": "http://localhost:%s/index.html"` (with an open-externally
  action) so the preview opens the dashboard file rather than the server root.

Serving over HTTP is required because the page fetches JSON; opening
`index.html` directly from the filesystem can prevent that fetch in browsers.

## Ordered execution plan, dependencies, and parallel decisions

### Phase 1 — agree the interface contract (sequential)

1. The Orchestrator gives the Designer the brief and this plan and requests the
   concise visual/accessibility handoff described in the table.
2. The Orchestrator confirms the handoff retains the required field names,
   `.dashboard`, and `.project-card`; it then gives the Coder the approved
   direction and data contract.

**Why sequential:** the markup and stylesheet need stable semantic hierarchy,
CSS hooks, and badge vocabulary. Starting implementation before this handoff
risks redesign and incompatible selectors.

### Phase 2 — independent foundations (parallel)

Once Phase 1 is approved, the Coder may create:

- `app/project-data.json`, using the specified schema and data vocabulary; and
- `.vscode/launch.json`, using the fixed command, working directory, and URL.

**Why parallel:** these files have non-overlapping scopes and neither consumes
the contents of the other. Their only shared facts—the `app/` directory,
filename, and fixed data schema—are already settled by Phase 1. If separate
Coder tasks are used, assign one file to each task to avoid write conflicts.

### Phase 3 — dashboard implementation (mostly sequential)

1. Finish and parse the JSON fixture first.
2. Implement `app/index.html` against that finalized shape, including data
   loading, safe rendering, filtering, state messages, and semantic controls.
3. Implement `app/styles.css` using the actual HTML hooks, then perform the
   Designer’s visual/accessibility review and make Coder-owned corrections.

**Why sequential:** HTML consumes the JSON keys and establishes the selectors
and component states that CSS must style. The CSS author can begin token
exploration in parallel after the design handoff, but merging the final
stylesheet before the markup hooks exist would create avoidable integration
work. The Designer review follows a rendered build so it can assess real
content, wrapping, focus, and contrast.

### Phase 4 — integration and validation (sequential)

The Orchestrator performs the validation below only after all implementation
files are present. Fixes return to the Coder in the relevant assigned file;
repeat the affected checks after every fix.

## Validation and acceptance expectations

### Functional rendering and interactions

- Start the HTTP preview, open `index.html`, and confirm the visible heading is
  exactly “Project Pulse,” not a directory listing.
- Confirm every JSON project creates exactly one visible `.project-card` and
  that each card displays its name, owner, status, recent activity, and
  priority.
- Confirm the stylesheet is linked and the document requests
  `project-data.json` rather than duplicating hard-coded cards.
- Exercise All and each status filter with mouse and keyboard. Confirm the
  card set, result count, and selected state update correctly; confirm the
  no-results message works if a no-match state is made available for testing.
- Test an unavailable or invalid JSON response during development and confirm a
  clear error state appears without uncaught console errors.

### Responsive and accessibility review

- Inspect a narrow phone-sized viewport and a desktop viewport: no horizontal
  overflow, readable text, wrapped controls, and a usable one-to-multiple
  column card layout.
- Navigate solely by keyboard; focus order follows the page, focus is clearly
  visible, filters work with Enter/Space, and no interaction requires a mouse.
- Verify semantic landmarks/headings, native buttons and labels, status/priority
  text in addition to color, and sufficient foreground/background contrast.
- Verify cards and controls retain readable spacing and target sizes at narrow
  widths and at browser zoom.

### JSON validity and data integration

- Run `python3 -m json.tool app/project-data.json` successfully.
- Verify the top-level `projects` key is an array and each item supplies
  non-empty `name`, `owner`, `status`, `recentActivity`, and `priority`
  strings.
- Confirm JSON status values match the filter/badge mapping and no project
  disappears because of a key-name or casing mismatch.

### VS Code launch configuration

- Run `python3 -m json.tool .vscode/launch.json` successfully; it must contain
  no comments or trailing-comma syntax.
- In VS Code Run and Debug, select **Run Project Pulse Dashboard** and start
  it. Confirm `python3 -m http.server 5500` runs with `app/` as its working
  directory.
- Confirm `serverReadyAction` opens
  `http://localhost:%s/index.html` as the resolved localhost URL and the
  browser displays the dashboard. Stop the preview server after testing.

## Risks and resolution rules

- **Fetch fails when opened as a file:** always use the launch configuration or
  an equivalent HTTP server for manual testing.
- **Unexpected data values:** derive filter choices from valid data or keep the
  data vocabulary and mapping in sync; show a load error for an invalid root
  shape.
- **Color-only status semantics:** retain status/priority text labels and use
  color only as a supplemental cue.
- **Scope conflicts:** Designer supplies decisions and reviews; Coder is the
  only implementation-file writer. The Orchestrator coordinates corrections
  rather than editing files.

No open product questions block implementation: the brief fixes the mandatory
fields, app paths, title, server command, launch name, and preview target.
