<script>
	import { onMount } from 'svelte';
	import { csv } from 'd3';
	import LineChart from '#lib/LineChart.svelte';

	let data = $state([]);
	let loading = $state(true);
	let error = $state('');
	let years = $derived(data.length ? `${data[0].year}–${data[data.length - 1].year}` : '');

	onMount(() => {
		const controller = new AbortController();
		let disposed = false;

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
		};
	});
</script>

<svelte:head>
	<title>Road Story | U.S. traffic deaths</title>
	<meta name="description" content="Annual U.S. motor-vehicle traffic deaths, shown as the number of people killed each year." />
</svelte:head>

<main>
	<header>
		<p class="eyebrow">Road Story / United States</p>
		<h1>U.S. traffic deaths</h1>
		<p class="intro">People killed in U.S. motor-vehicle traffic crashes each year.</p>
	</header>

	{#if loading}
		<p class="status" role="status">Loading annual road-deaths data…</p>
	{:else if error}
		<p class="status" role="alert">{error}</p>
	{:else}
		<section class="graphic" aria-label="Annual U.S. road deaths">
			<div class="chart-card">
				<p class="eyebrow">U.S. traffic deaths / {years}</p>
				<h2>The number of lives lost</h2>
				<LineChart {data} measure="fatalities" />
				<p class="chart-note">2024 is a preliminary estimate. This chart shows the annual count of deaths.</p>
			</div>
		</section>
	{/if}

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
	.graphic { max-width: 760px; margin-bottom: 40px; min-width: 0; }
	.chart-card { width: 100%; padding: 24px 20px 16px; background: #fffef9; border: 1px solid #d9ded5; border-radius: 6px; }
	.chart-card h2 { font-size: clamp(1.15rem, 2vw, 1.7rem); font-weight: 500; margin: 12px 0 20px; }
	.chart-note { font-size: 0.75rem; color: #506362; line-height: 1.5; }
	footer { border-top: 1px solid #cbd3cd; padding: 30px 0 50px; font-size: 0.8rem; line-height: 1.7; color: #506362; }
	a { color: #285c58; text-underline-offset: 3px; }
	.status { padding: 60px 0; }
	@media (max-width: 760px) {
		header { padding-top: 45px; }
		.chart-card { padding: 12px; }
		.chart-card h2 { margin: 8px 0; }
		.chart-note { margin: 6px 0 0; font-size: 0.65rem; }
	}
</style>
