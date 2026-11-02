<script>
	import { extent, format, line, max, scaleLinear } from 'd3';

	const chartId = $props.id();
	let { data = [], measure = 'fatalities', xDomain = null, highlightedYear = null, annotation = null } = $props();
	let containerWidth = $state(640);
	let width = $derived(Math.max(containerWidth, 280));
	let height = $derived(width < 500 ? 230 : 360);
	const margin = { top: 42, right: 18, bottom: 44, left: 58 };
	let isRate = $derived(measure === 'rate_per_100m_vmt');
	let label = $derived(isRate ? 'Deaths per 100 million vehicle-miles' : 'Deaths');
	let valueFormat = $derived(format(isRate ? '.2f' : ',.0f'));
	let xScale = $derived(scaleLinear()
		.domain(xDomain ?? (data.length ? extent(data, (d) => d.year) : [1975, 2024]))
		.range([margin.left, width - margin.right]));
	let yScale = $derived(scaleLinear()
		.domain([0, max(data, (d) => d[measure]) || 1]).nice()
		.range([height - margin.bottom, margin.top]));
	let xTicks = $derived(xScale.ticks(width < 500 ? 4 : 6).filter(Number.isInteger));
	let yTicks = $derived(yScale.ticks(4));
	let path = $derived(line()
		.defined((d) => Number.isFinite(d.year) && Number.isFinite(d[measure]))
		.x((d) => xScale(d.year))
		.y((d) => yScale(d[measure]))(data));
	let highlighted = $derived(data.filter((d) => d.year === highlightedYear
		&& d.year >= xScale.domain()[0] && d.year <= xScale.domain()[1]));
	let annotationPoint = $derived(annotation ? data.find((d) => d.year === annotation.year
		&& d.year >= xScale.domain()[0] && d.year <= xScale.domain()[1]) : null);
	let annotationX = $derived(annotationPoint ? Math.max(margin.left + 8, xScale(annotationPoint.year) - 240) : 0);
	let annotationY = $derived(annotationPoint ? Math.max(margin.top + 18, yScale(annotationPoint[measure]) - 85) : 0);
</script>

<div class="chart" bind:clientWidth={containerWidth}>
	<svg viewBox={`0 0 ${width} ${height}`} role="img" aria-label={`${label} by year in the United States`}>
		<title>{label} by year</title>
		<desc>A line chart of annual U.S. traffic {isRate ? 'death rates per 100 million vehicle-miles traveled' : 'deaths'}. The vertical axis starts at zero. The 2024 figures are preliminary. {annotationPoint ? annotation.lines.join('. ') : ''}</desc>
		<defs>
			<clipPath id={`${chartId}-plot`}>
				<rect x={margin.left} y={margin.top} width={width - margin.left - margin.right} height={height - margin.top - margin.bottom} />
			</clipPath>
			{#if annotationPoint}
				<marker id={`${chartId}-arrow`} viewBox="0 0 8 8" refX="7" refY="4" markerWidth="8" markerHeight="8" orient="auto" markerUnits="userSpaceOnUse">
					<path class="arrowhead" d="M0,0 L8,4 L0,8 Z" />
				</marker>
			{/if}
		</defs>
		<text class="axis-label" x={margin.left} y="18">{label}</text>
		{#each yTicks as value (value)}
			<g transform={`translate(0, ${yScale(value)})`}>
				<line class="grid" x1={margin.left} x2={width - margin.right} y1="0" y2="0" />
				<text x={margin.left - 10} dy="0.32em" text-anchor="end">{isRate ? format('.1f')(value) : format(',d')(value)}</text>
			</g>
		{/each}
		<line class="axis" x1={margin.left} x2={margin.left} y1={margin.top} y2={height - margin.bottom} />
		<line class="axis" x1={margin.left} x2={width - margin.right} y1={height - margin.bottom} y2={height - margin.bottom} />
		{#each xTicks as year (year)}
			<g transform={`translate(${xScale(year)}, ${height - margin.bottom})`}>
				<line class="axis" y2="5" />
				<text y="20" text-anchor="middle">{year}</text>
			</g>
		{/each}
		<text class="axis-label" x={(margin.left + width - margin.right) / 2} y={height - 3} text-anchor="middle">Year</text>
		<path class="data-line" d={path ?? ''} clip-path={`url(#${chartId}-plot)`} />
		{#each highlighted as point (point.year)}
			<g transform={`translate(${xScale(point.year)}, ${yScale(point[measure])})`}>
				<circle r="5" />
				<text class="point-label" x={point.year > xScale.domain()[0] + (xScale.domain()[1] - xScale.domain()[0]) / 2 ? -10 : 10} y="-13" text-anchor={point.year > xScale.domain()[0] + (xScale.domain()[1] - xScale.domain()[0]) / 2 ? 'end' : 'start'}>{point.year}: {valueFormat(point[measure])}</text>
			</g>
		{/each}
		{#if annotationPoint}
			<g class="comparison-annotation">
				<path class="annotation-arrow" d={`M${annotationX + 100},${annotationY + 42} L${xScale(annotationPoint.year) - 5},${yScale(annotationPoint[measure]) - 5}`} marker-end={`url(#${chartId}-arrow)`} />
				<text class="annotation-label" x={annotationX} y={annotationY}>
					{#each annotation.lines as text, index}
						<tspan x={annotationX} dy={index === 0 ? 0 : 16}>{text}</tspan>
					{/each}
				</text>
			</g>
		{/if}
	</svg>
	<details>
		<summary>View annual values</summary>
		<div class="table-scroll">
			<table>
				<caption>{label} by year (2024 preliminary)</caption>
				<thead><tr><th scope="col">Year</th><th scope="col">{label}</th></tr></thead>
				<tbody>
					{#each data as row (row.year)}
						<tr><th scope="row">{row.year}</th><td>{valueFormat(row[measure])}</td></tr>
					{/each}
				</tbody>
			</table>
		</div>
	</details>
</div>

<style>
	.chart { width: 100%; min-width: 0; }
	svg { display: block; width: 100%; overflow: visible; }
	text { fill: #506362; font-family: system-ui, sans-serif; font-size: 11px; }
	.axis-label { fill: #222f30; font-size: 11px; font-weight: 600; }
	.grid { stroke: #e2e6de; }
	.axis { stroke: #a9b7af; }
	.data-line { fill: none; stroke: #285c58; stroke-width: 2.5; stroke-linejoin: round; stroke-linecap: round; }
	circle { fill: #b34425; stroke: #fffef9; stroke-width: 2; }
	.point-label { fill: #95391f; font-weight: 700; paint-order: stroke; stroke: #fffef9; stroke-width: 4px; stroke-linejoin: round; }
	.annotation-label { fill: #95391f; font-weight: 600; paint-order: stroke; stroke: #fffef9; stroke-width: 4px; stroke-linejoin: round; }
	.annotation-arrow { fill: none; stroke: #95391f; stroke-width: 1.5; }
	.arrowhead { fill: #95391f; }
	details { margin-top: 8px; font-size: 0.75rem; color: #506362; }
	summary { cursor: pointer; width: fit-content; }
	.table-scroll { max-height: 180px; overflow: auto; margin-top: 10px; }
	table { width: 100%; border-collapse: collapse; font-variant-numeric: tabular-nums; }
	caption { text-align: left; padding: 8px; }
	th, td { padding: 6px 8px; border-bottom: 1px solid #e2e6de; text-align: right; }
	th:first-child { text-align: left; }
</style>
