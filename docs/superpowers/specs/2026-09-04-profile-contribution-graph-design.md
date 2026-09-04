# Profile Contribution Graph Design

## Goal

Add a year-long contribution graph at the end of the GitHub profile README without introducing a runtime dependency on an external image service or adding unrelated statistics.

## Presentation

- Rename the existing `Projects` section to `Selected Projects` without changing its entries.
- Add a final `## Contributions` section after the Tools table.
- Show one full-width, flat year-long contribution calendar.
- Provide separate light and dark SVGs through an HTML `picture` element.
- Do not add streaks, ranks, counters, animations, badges, or explanatory text.

## Generation

- Fetch a rolling 365-day SVG from the established GitHub Chart renderer during the scheduled workflow.
- Validate its size, day count, XML structure, and absence of executable SVG content before accepting it.
- Derive the dark variant locally from GitHub's light graph using GitHub's dark contribution colors.
- Run the workflow once per day and allow manual runs.
- Store the generated SVGs in `profile/` so the README always loads repository-owned assets.
- Replace the stored SVGs only after both variants pass validation, preserving the last successful files when fetching or validation fails.

## Verification

- Validate the workflow syntax and README formatting.
- Confirm both generated SVG paths exist and parse as valid XML.
- Render the README in light and dark modes at desktop and mobile widths.
- Confirm the graph is full-width, legible, and does not overlap adjacent content.

## Scope

Only the `Selected Projects` heading change, contribution section, two generated assets, and refresh workflow are included. Existing project entries, other README content, and unrelated untracked files remain unchanged.
