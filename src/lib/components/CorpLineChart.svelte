<div class='v3-container'>
    <div class="chart"></div>
</div>

<style>
    .v3-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding-top: 1rem;
        padding-bottom: 8rem
    }
</style>

<script lang="ts">
    import { onMount } from 'svelte';
    import { select } from 'd3-selection';
    import { scaleLinear, scaleTime } from 'd3-scale';
    import { line } from 'd3-shape';
    import { axisBottom, axisLeft } from 'd3-axis';
    import { timeParse } from 'd3-time-format';
    import * as d3 from 'd3';

    const margin = { top: 20, right: 30, bottom: 30, left: 40 };
    const width = 1000 - margin.left - margin.right;
    const height = 600 - margin.top - margin.bottom;

    onMount(() => {
        d3.csv('/datasets/corp_rates.csv').then(data => {
            console.log(data);
            const svg = select('.chart')
                .append('svg')
                .attr('width', width + margin.left + margin.right)
                .attr('height', height + margin.top + margin.bottom)
                .append('g')
                .attr('transform', `translate(${margin.left},${margin.top})`);
            
            svg.append('rect')
                .attr('width', width).attr('height', height)
                .attr('fill', 'blue').attr('fill-opacity', 0.00);

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
                .domain([0, 100])
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

            const mouse_g = svg.append('g').classed('mouse', true).style('display', 'none');
            const markerLine = mouse_g
                .append('line')
                .attr('x1', 0)
                .attr('x2', 0)
                .attr('y1', 0)
                .attr('y2', height - margin.top - margin.bottom)
                .attr('stroke-width', 3)
                .attr('stroke', 'darkviolet')
                .attr('opacity', 1);
                
            const markerDot1 = mouse_g
                .append('circle')
                .attr('cx', 0)
                .attr('cy', 0)
                .attr('r', 5)
                .attr('fill', 'darkviolet')
                .attr('opacity', 1);

            const markerDot2 = mouse_g
                .append('circle')
                .attr('cx', 0)
                .attr('cy', 0)
                .attr('r', 5)
                .attr('fill', 'darkviolet')
                .attr('opacity', 1);

            const bisect = d3.bisector(d => d.year);
            console.log({svg});
            svg.on('mouseover', () => {
                mouse_g.style('display', null);
            });
            svg.on('mousemove', (e) => {
                const pointerCoords = d3.pointer(e);
                const [posX, posY] = pointerCoords;
                const date = x.invert(posX);
                const index = bisect.center(data, date);
                const d = data[index];
                
                const x_ = x(d.year);
                const y1_ = y(d.own_rate);
                const y2_ = y(d.eviction_share);
                
                markerLine
                    .attr('x1', x_)
                    .attr('x2', x_)
                    .attr('y1', y1_)
                    .attr('y2', y2_);
                
                markerDot1
                    .attr('cx', x_)
                    .attr('cy', y1_);
                markerDot2
                    .attr('cx', x_)
                    .attr('cy', y2_);
            });
            svg.on('mouseout', () => {
                mouse_g.style('display', 'none');
            });

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