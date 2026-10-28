# Repository Guidelines

## Project Structure & Module Organization

This repository is a JavaScript SvelteKit application for a road-fatality scrollytelling project, with D3 and Scrollama installed. The current page is still the starter scaffold.

- `src/routes/+page.svelte`: main page and story entry point.
- `src/routes/+layout.svelte`: shared layout and favicon setup.
- `src/lib/`: reusable components, visualization helpers, and imported assets; use the configured `#lib` import alias.
- `src/app.html`: HTML document shell.
- `static/`: publicly served CSV datasets, dataset documentation, and `robots.txt`. For example, load the annual dataset from `/us_road_deaths_by_year.csv`.
- `vite.config.js`: SvelteKit plugin, compiler, and adapter configuration.

Keep generated `.svelte-kit/` files and installed `node_modules/` out of source changes.

## Build, Test, and Development Commands

- `npm ci`: install dependencies from `package-lock.json`.
- `npm run dev`: start the local Vite development server at `http://localhost:5173`; add `-- --open` to open a browser.
- `npm run build`: create the production build and check compilation.
- `npm run preview`: serve the production build locally after building.

## Coding Style & Naming Conventions

Follow existing files: tab indentation, single-quoted JavaScript strings, and semicolons. Use ES modules and Svelte 5 runes (`$state`, `$derived`, `$props`), not legacy `$:`; the Vite configuration forces runes mode for project code. Preserve SvelteKit route filenames such as `+page.svelte` and `+layout.svelte`. Use descriptive camelCase names for JavaScript variables and functions, and PascalCase for reusable Svelte components. No formatter or linter is currently configured.

Use D3 for scales, axes, CSV loading, and path calculations. Svelte renders data marks in markup, using `{#each}` for repeated marks. D3 may draw axes in dedicated `<g>` elements.

Scrollama is browser-only because it uses Intersection Observer. Call `scrollama()` inside `onMount`, never at module top level, to avoid SSR crashes with `window is not defined`.

## Scrollytelling Architecture & Pacing

- Keep one shared scene state in `+page.svelte`: `let step = $state(0)`. Scrollama's `onStepEnter` sets it to the active step index. Keep one owner; in the core version, only `onStepEnter` updates it.
- Derive the measure, x-domain/zoom, and highlighted year from `step` in the page. Pass them to `LineChart` as `measure`, `xDomain`, and `highlightedYear` props alongside `data`. The chart renders supplied settings and never updates itself imperatively.
- Optional progress animation may depend on a separate progress value. Navigation controls scroll to a step rather than setting `step` directly.
- Use a `.graphic` div with `position: sticky; top: 0; height: 100vh` and a `.steps` column of `<div class="step">` blocks, approximately `90vh` each, one per scene.
- The first step establishes the graphic. Each later step changes one thing: add a line, change a measure, zoom, or highlight a year. Split independent changes into separate steps.

## Testing Guidelines

There is no automated test framework, test script, or coverage requirement yet. Run `npm run build` before submitting changes. For visualization changes, also check rendering, scrolling transitions, responsive layouts, and browser-console errors locally. If adding automated tests, document the runner, naming convention, and execution command.

## Data Integrity

Read `static/road_deaths_README.md` before changing charts or narrative claims. Load CSV files from `static/` with `d3.csv` inside `onMount`:

- `us_road_deaths_by_year.csv`: `year`, `fatalities`, `vmt_billions`, `rate_per_100m_vmt`, `population`. Starts in 1975; check the file for its ending year. Convert all five columns to numbers.
- `fatal_crashes_2022_when.csv` (stretch): `dimension` (`day_of_week`, `hour`, or `month`), `bucket`, `sort_key`, `deaths`. Convert `sort_key` and `deaths` to numbers.

Every scene must identify whether it shows `fatalities` (a count) or `rate_per_100m_vmt` (risk per mile); these can move in opposite directions. Check every trend against the relevant years. A column's presence does not establish a cause: this dataset supports descriptions and comparisons, not causal claims. The files summarize deaths in fatal crashes; do not infer total crashes or injuries. Retain the preliminary status of 2024 data, and avoid inferring driving risk from within-year counts without exposure data.

Data accuracy is required. A documented, sentence-by-sentence review using Claude is an optional stretch goal; record a correction or verified example if completing this review.

## Commit & Pull Request Guidelines

The single existing commit uses `scaffold: <description>`; follow that concise, action-oriented pattern with an appropriate prefix. Pull requests should describe the change, record validation performed, link relevant issues, and include screenshots for visible UI or chart changes. Explain any dataset or interpretation changes explicitly.

After checking a working feature, remind the user to run `git add -A && git commit`. If a change breaks a scene, inspect the diff before deciding whether to restore it.

## Definition of Done

- Run `npm run build` and check the app with `npm run dev`; there must be no console errors, including while scrolling.
- Provide a sticky graphic and 5–7 core steps, one per storyboard scene, comparing count and rate. Each transition changes one thing; optional scenes may extend the story.
- Check rendering, scrolling transitions, and responsive layouts. Support factual claims with the CSV and remove unsupported or overstated wording.
- Optional stretch: review every sentence against the CSV and document a correction or verified example.
