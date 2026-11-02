<script>
	import { onMount } from 'svelte';
	import { csv } from 'd3';
	import scrollama from 'scrollama';
	import LineChart from '#lib/LineChart.svelte';

	let step = $state(0);
	let data = $state([]);
	let loading = $state(true);
	let error = $state('');
	let years = $derived(data.length ? `${data[0].year}–${data[data.length - 1].year}` : '');
	const scenes = [
		{
			// Establish the full fatalities chart.
			title: 'Exposition',
			measure: 'fatalities', xDomain: null, highlightedYear: null,
			narration: 'This chart shows the number of people killed on U.S. roads each year from 1975 to 2024. Annual deaths declined overall, but the path was uneven. The 2024 figure is preliminary.'
		},
		{
			// Change only the x-domain: zoom around 2011.
			title: 'Rising Action',
			measure: 'fatalities', xDomain: [2007, 2015], highlightedYear: null,
			narration: 'Looking more closely at 2011, we see a low point in the annual number of road deaths. The count was lower than in earlier decades.'
		},
		{
			// Change only the x-domain: shift toward 2022.
			title: 'Rising Action',
			measure: 'fatalities', xDomain: [2019, 2024], highlightedYear: null,
			narration: 'After 2011, the annual number of road deaths rose overall. By 2022, the count was much higher than at that earlier low point.'
		},
		{
			// Change only the x-domain: focus on 2022–2024.
			title: 'Rising Action',
			measure: 'fatalities', xDomain: [2022, 2024], highlightedYear: null,
			narration: 'The annual number of road deaths decreased from 2022 to 2024, although the 2024 figure is preliminary. That decline did not erase the earlier increase: the count remained above the 2011 low.'
		},
		{
			// Change only the x-domain: restore the full timeline.
			title: 'Climax',
			measure: 'fatalities', xDomain: null, highlightedYear: null,
			narration: 'Across the full timeline, the annual number of road deaths declined overall, with periods of increases and decreases along the way. The 2024 endpoint remains preliminary.'
		},
		{
			// Change only the measure: show deaths per 100 million vehicle-miles.
			title: 'Climax / Comparison',
			measure: 'rate_per_100m_vmt', xDomain: null, highlightedYear: null,
			narration: 'Now the chart shows deaths per 100 million miles driven instead of the total number of deaths each year. Accounting for how much driving took place gives us a different view of the long-term pattern. The 2024 rate is preliminary.'
		},
		{
			// Change only the annotation: compare the 2022 rate with the 2014 low.
			title: 'Resolution',
			measure: 'rate_per_100m_vmt', xDomain: null, highlightedYear: null,
			comparison: { year: 2022, baselineYear: 2014 },
			narration: 'In 2022, the rate was 1.34 deaths per 100 million miles driven, about 24% above the 2014 low of 1.08. Despite the long-term decline in deaths per mile, the rate remained above that earlier low.'
		}
	];
	let scene = $derived(scenes[step]);
	let measure = $derived(scene.measure);
	let xDomain = $derived(scene.xDomain);
	let highlightedYear = $derived(scene.highlightedYear);
	let annotation = $derived.by(() => {
		if (!scene.comparison) return null;
		const point = data.find((row) => row.year === scene.comparison.year);
		const baseline = data.find((row) => row.year === scene.comparison.baselineYear);
		if (!point || !baseline) return null;
		const increase = Math.round((point[measure] / baseline[measure] - 1) * 100);
		return {
			year: point.year,
			lines: [
				`${point.year}: ${increase}% above ${baseline.year}`,
				`${point[measure].toFixed(2)} vs ${baseline[measure].toFixed(2)} deaths`,
				'per 100 million miles'
			]
		};
	});

	onMount(() => {
		const controller = new AbortController();
		let disposed = false;
		const scroller = scrollama();
		scroller.setup({ step: '.step', offset: 0.6 })
			.onStepEnter((response) => {
				step = response.index;
			});
		const handleResize = () => scroller.resize();
		window.addEventListener('resize', handleResize);

		async function loadData() {
			try {
				const rows = await csv('/us_road_deaths_by_year.csv', { signal: controller.signal }, (row) => ({
					year: Number(row.year),
					fatalities: Number(row.fatalities),
					vmt_billions: Number(row.vmt_billions),
					rate_per_100m_vmt: Number(row.rate_per_100m_vmt),
					population: Number(row.population)
				}));
				if (!rows.length || rows.some((row) => Object.values(row).some((value) => !Number.isFinite(value)))) {
					throw new Error('Invalid annual data');
				}
				if (disposed) return;
				data = rows.sort((a, b) => a.year - b.year);
			} catch (cause) {
				if (disposed || cause.name === 'AbortError') return;
				error = 'We couldn’t load the road-deaths data. Please refresh to try again.';
			} finally {
				if (!disposed) loading = false;
			}
		}

		loadData();
		return () => {
			disposed = true;
			controller.abort();
			window.removeEventListener('resize', handleResize);
			scroller.destroy();
		};
	});
</script>

<svelte:head>
	<title>Deaths per mile fell far more than total U.S. road deaths | Road Story</title>
	<meta name="description" content="Annual U.S. motor-vehicle traffic deaths, shown as counts and deaths per 100 million vehicle-miles." />
</svelte:head>

<main>
	<header>
		<p class="eyebrow">Road Story / United States</p>
		<h1>Deaths per mile fell far more than total U.S. road deaths</h1>
		<p class="intro">People killed in U.S. motor-vehicle traffic crashes each year.</p>
	</header>

	<section class="story" aria-label="U.S. road deaths in seven scenes">
		<div class="graphic">
			{#if loading}
				<p class="status" role="status">Loading annual road-deaths data…</p>
			{:else if error}
				<p class="status" role="alert">{error}</p>
			{:else}
				<div class="chart-card">
					<p class="eyebrow">U.S. traffic deaths / {years}</p>
					<h2>{measure === 'fatalities' ? 'The number of lives lost' : 'Deaths per 100 million vehicle-miles'}</h2>
					<LineChart {data} {measure} {xDomain} {highlightedYear} {annotation} />
					<p class="chart-note">2024 is a preliminary estimate. {measure === 'fatalities' ? 'This chart shows the annual count of deaths.' : 'This chart shows deaths per 100 million vehicle-miles traveled.'}</p>
				</div>
			{/if}
		</div>
		<div class="steps">
			{#each scenes as scene, index}
				<div class="step" aria-labelledby={`scene-${index + 1}`}>
					<div class="narration">
						<h2 id={`scene-${index + 1}`}>Scene {index + 1} — {scene.title}</h2>
						<p>{scene.narration}</p>
					</div>
				</div>
			{/each}
		</div>
	</section>

	<footer>
		<p>Sources: NHTSA Fatality Analysis Reporting System (deaths and rates); Federal Highway Administration (vehicle-miles traveled).</p>
		<p>These data describe deaths in fatal crashes, not total crashes or injuries. <a href="/road_deaths_README.md">Read the dataset notes</a> · <a href="/us_road_deaths_by_year.csv" download>Download the CSV</a></p>
	</footer>
</main>

<style>
	:global(body) { margin: 0; background: #f6f4ed; color: #222f30; font-family: system-ui, sans-serif; }
	:global(*) { box-sizing: border-box; }
	main { max-width: 1440px; margin: 0 auto; padding: 0 5vw; }
	header { max-width: 850px; padding: 80px 0 45px; }
	.eyebrow { color: #506362; font-size: 0.72rem; font-weight: 700; letter-spacing: 0.12em; text-transform: uppercase; }
	h1 { margin: 22px 0; font-family: Georgia, serif; font-size: clamp(2.8rem, 6vw, 5.7rem); font-weight: 400; line-height: 1.05; letter-spacing: -0.04em; }
	.intro { max-width: 620px; font-size: 1.15rem; line-height: 1.7; }
	.story { display: grid; grid-template-columns: minmax(0, 1.8fr) minmax(250px, 1fr); gap: 3vw; margin-bottom: 40px; }
	.graphic { position: sticky; top: 0; height: 100vh; align-self: start; display: flex; align-items: center; max-width: 760px; min-width: 0; }
	.chart-card { width: 100%; max-height: calc(100vh - 32px); overflow-y: auto; padding: 24px 20px 16px; background: #fffef9; border: 1px solid #d9ded5; border-radius: 6px; }
	.chart-card h2 { font-size: clamp(1.15rem, 2vw, 1.7rem); font-weight: 500; margin: 12px 0 20px; }
	.chart-note { font-size: 0.75rem; color: #506362; line-height: 1.5; }
	.steps { min-width: 0; }
	.step { min-height: 90vh; display: flex; align-items: center; padding: 24px 0; }
	.narration h2 { font-size: 1rem; font-weight: 600; }
	.narration p { font-size: 1.05rem; line-height: 1.7; }
	footer { border-top: 1px solid #cbd3cd; padding: 30px 0 50px; font-size: 0.8rem; line-height: 1.7; color: #506362; }
	a { color: #285c58; text-underline-offset: 3px; }
	.status { padding: 60px 0; }
	@media (max-width: 760px) {
		header { padding-top: 45px; }
		.story { display: block; }
		.graphic { align-items: flex-start; padding-top: 16px; }
		.chart-card { max-height: 55vh; padding: 12px; }
		.chart-card h2 { margin: 8px 0; }
		.chart-note { margin: 6px 0 0; font-size: 0.65rem; }
		.steps { position: relative; margin-top: -100vh; pointer-events: none; }
		.step { align-items: flex-end; padding: 58vh 0 4vh; }
		.narration { width: 100%; padding: 12px 16px; background: #fffef9; border: 1px solid #d9ded5; border-radius: 6px; pointer-events: auto; }
		.narration h2 { margin: 0 0 8px; }
		.narration p { margin: 0; font-size: 0.95rem; line-height: 1.6; }
	}
</style>
