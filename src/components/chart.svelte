<script>
	import { afterUpdate } from 'svelte';
	import { onMount } from 'svelte';

	import * as d3 from 'd3';

	export let data;

	let svg;
	const margin = { top: 20, right: 30, bottom: 40, left: 50 };
	const width = 800 - margin.left - margin.right;
	const height = 300 - margin.top - margin.bottom;

	const legendData = [
		{ label: 'Ø Last 20 Tokens', color: 'steelblue' },
		{ label: 'Ø Last 5 Offers', color: 'red' },
		{ label: 'Ø Last 5 weighted Offers', color: 'green' }
	];

	function createSvg() {
		svg = d3
			.select('#chart')
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr('transform', `translate(${margin.left},${margin.top})`);
	}

	function updateChart() {
		if (!svg || !data) return;

		// Clear the existing chart
		svg.selectAll('*').remove();

		// Create scales
		const x = d3
			.scaleLinear()
			.domain([0, data.length - 1])
			.range([0, width]);
		const y = d3
			.scaleLinear()
			.domain([
				0,
				d3.max(data, (d) =>
					Math.max(
						d.averagePriceLastXTokens,
						d.averagePriceLastXOffers,
						d.weightedAveragePriceLastXOffers
					)
				)
			])
			.range([height, 0]);

		// Define lines for each attribute
		const lineGenerators = {
			averagePriceLastXTokens: d3
				.line()
				.x((d, i) => x(i))
				.y((d) => y(d.averagePriceLastXTokens)),
			averagePriceLastXOffers: d3
				.line()
				.x((d, i) => x(i))
				.y((d) => y(d.averagePriceLastXOffers)),
			weightedAveragePriceLastXOffers: d3
				.line()
				.x((d, i) => x(i))
				.y((d) => y(d.weightedAveragePriceLastXOffers))
		};

		// Append legend after drawing lines
		const legend = svg
			.append('g')
			.attr('font-family', 'sans-serif')
			.attr('font-size', 10)
			.attr('text-anchor', 'start')
			.selectAll('g')
			.data(legendData)
			.join('g')
			.attr('transform', (d, i) => `translate(-10,${i * 20})`);

		legend
			.append('rect')
			.attr('x', 0)
			.attr('width', 19)
			.attr('height', 19)
			.attr('fill', (d) => d.color);

		legend
			.append('text')
			.attr('x', 24)
			.attr('y', 9.5)
			.attr('dy', '0.32em')
			.text((d) => d.label);

		// Draw lines for each attribute
		for (const [key, lineGenerator] of Object.entries(lineGenerators)) {
			svg
				.append('path')
				.datum(data)
				.attr('fill', 'none')
				.attr('stroke', getStrokeColor(key))
				.attr('stroke-width', 1.5)
				.attr('d', lineGenerator);
		}
	}

	function getStrokeColor(attribute) {
		const colors = {
			averagePriceLastXTokens: 'steelblue',
			averagePriceLastXOffers: 'red',
			weightedAveragePriceLastXOffers: 'green'
		};
		return colors[attribute];
	}

	onMount(createSvg);

	afterUpdate(updateChart);
</script>

<div id="chart" />
