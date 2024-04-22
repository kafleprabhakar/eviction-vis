<div class="chart"></div>

<script lang="ts">
    import { onMount } from 'svelte';
    import { select } from 'd3-selection';
    import { scaleLinear, scaleTime } from 'd3-scale';
    import { line } from 'd3-shape';
    import { axisBottom, axisLeft } from 'd3-axis';
    import { timeParse } from 'd3-time-format';
    import * as d3 from 'd3';

    const margin = { top: 20, right: 30, bottom: 30, left: 40 };
    const width = 600 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    onMount(() => {
        d3.csv('/datasets/corp_rates.csv').then(data => {
            console.log(data);
            const svg = select('.chart')
                .append('svg')
                .attr('width', width + margin.left + margin.right)
                .attr('height', height + margin.top + margin.bottom)
                .append('g')
                .attr('transform', `translate(${margin.left},${margin.top})`);

                data.forEach(d => {
                    // Convert the "year" column to JavaScript Date objects
                    d.year = new Date(+d.year, 0); // Assuming "year" is stored as integer
                });
            
            const x = scaleTime()
                // .domain(data.map(d => d.year))
                .domain(d3.extent(data, function(d) { return d.year; }))
                .range([0, width]);
            console.log(x.range());

            // const y1 = scaleLinear()
            //     .domain([0, 100])
            //     .range([height, 0]);

            const y = scaleLinear()
                .domain([0, d3.max(data, d => Math.max(d.eviction_share, d.own_rate))])
                .range([height, 0]);

            const xAxis = axisBottom(x);
            const yAxisLeft = axisLeft(y).ticks(5);
            // const yAxisRight = axisLeft(y2).ticks(5);

            svg.append('g')
                .attr('transform', `translate(0,${height})`)
                .call(xAxis);

            svg.append('g')
                .call(yAxisLeft);

            // svg.append('g')
            //     .attr('transform', `translate(${width}, 0)`)
            //     .call(yAxisRight);

            const line1 = line()
                .x(d => x(d.year))
                .y(d => y(d.own_rate));

            const line2 = line()
                .x(d => x(d.year))
                .y(d => y(d.eviction_share));

            svg.append('path')
                .datum(data)
                .attr('fill', 'none')
                .attr('stroke', 'steelblue')
                .attr('stroke-width', 1.5)
                .attr('d', line1);

            svg.append('path')
                .datum(data)
                .attr('fill', 'none')
                .attr('stroke', 'green')
                .attr('stroke-width', 1.5)
                .attr('d', line2);

            // Legend
            const legend = svg.append('g')
            .attr('transform', `translate(${0},${0})`);

            legend.append('path')
            .attr('d', `M 10 10 L 30 10`)
            .attr('stroke', 'steelblue')
            .attr('stroke-width', 2);

            legend.append('text')
            .attr('x', 40)
            .attr('y', 15)
            .text('Share of houses owned by corporations')
            .attr('font-size', '0.9rem');

            legend.append('path')
            .attr('d', `M 10 30 L 30 30`)
            .attr('stroke', 'green')
            .attr('stroke-width', 2);

            legend.append('text')
            .attr('x', 40)
            .attr('y', 35)
            .text('Share of evictions filed by corporations')
            .attr('font-size', '0.9rem');
            
        });
    });
</script>