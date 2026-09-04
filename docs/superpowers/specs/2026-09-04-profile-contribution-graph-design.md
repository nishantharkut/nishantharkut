# Profile Contribution Graph Design

## Goal

Add a year-long contribution graph at the end of the GitHub profile README without introducing a runtime dependency on an external image service or adding unrelated statistics.

## Presentation

- Add a final `## Contributions` section after the Tools table.
- Show one full-width, year-long contribution calendar.
- Provide separate light and dark SVGs through an HTML `picture` element.
- Do not add streaks, ranks, counters, animations, badges, or explanatory text.

## Generation

- Generate the calendar with the stable Lowlighter Metrics GitHub Action and its full-year isocalendar plugin.
- Pin the action to a specific release commit.
- Run the workflow once per day and allow manual runs.
- Store the generated SVGs in `profile/` so the README always loads repository-owned assets.
- Preserve the last successful SVGs when generation fails.

## Verification

- Validate the workflow syntax and README formatting.
- Confirm both generated SVG paths exist and parse as valid XML.
- Render the README in light and dark modes at desktop and mobile widths.
- Confirm the graph is full-width, legible, and does not overlap adjacent content.

## Scope

Only the contribution section, its two generated assets, and its refresh workflow are included. Existing README content and unrelated untracked files remain unchanged.
