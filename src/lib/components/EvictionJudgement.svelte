<div class="chart"></div>

<script lang="ts">
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    var drawGraph = function(){

        //number of icons to color in to visualize percent
        var percentNumber = 92;

        //variables for the font family, and some colors
        var fontFamily = "helvetica";
        var twitterFill = "#4D908E";
        var twitterFillActive = "#adf7b6";
        var svgBackgroundColor = '#264653';

        const width = 500;
        const height = 500;

        //create an svg with width and height
        var svg = d3.select('.chart')
            .append('svg')
            .attr("width", width)
            .attr("height", height)
            .style('background-color', svgBackgroundColor);

        /* 
            this is the twitter icon path definition that we are using instead of circles.
            the path data can be copied and pasted from an SVG icon file.
        */

        var person = svg.append("defs")
            .append("g")
            .attr("id","personIcon");

        person
            .append("path")
            .attr("d", "m31.25 35.016v19.906c0 4.4688 3.625 8.1094 8.0938 8.1094h0.3125l1.4531 32.344c0.03125 0.84375 0.71875 1.5 1.5625 1.5h14.656c0.84375 0 1.5312-0.65625 1.5625-1.5l1.4531-32.344h0.3125c4.4688 0 8.0938-3.6406 8.0938-8.1094v-19.906c0-4.4688-3.625-8.0938-8.0938-8.0938h-21.312c-4.4688 0-8.0938 3.625-8.0938 8.0938z")
            .attr("transform", "translate(0,0) scale(.3)");
        person
            .append("path")
            .attr("d", "m60.859 13.984c0 5.9961-4.8633 10.859-10.859 10.859s-10.859-4.8633-10.859-10.859 4.8633-10.859 10.859-10.859 10.859 4.8633 10.859 10.859")
            .attr("transform", "translate(0,0) scale(.3)");

        d3.csv('/datasets/eviction_judgement.csv').then(data => {
            console.log({data});

            const grouped = d3.rollup(
                data,
                v => d3.sum(v, d => parseInt(d.count)), // Summing up values for each group
                d => d.judge_cat
            );
            console.log({grouped});
            console.log({grouped: Array.from(grouped)});

            let cumulativeFrequency = 0;
            let cumulitiveData = {};
            Array.from(grouped).forEach(d => {
                console.log({d});
                cumulativeFrequency += d[1];
                cumulitiveData[d[0]] = cumulativeFrequency;
                // cumulativeFrequency += d.count;
                // d.cumulativeFrequency = cumulativeFrequency;
            });

            const colors = {
                '0': 'blue',
                '1': 'red',
                '2': 'green',
                '3': 'yellow'
            };
            const getColor = function(d){
                // loop over cumulitiveData
                for (let key in cumulitiveData){
                    if (d < cumulitiveData[key]){
                        return colors[key];
                    }
                }
                return 'brown';
            }

            //10 rows and 10 columns 
            var numCols = 20;
            var numRows = Math.ceil(cumulativeFrequency/numCols);

            //x and y axis scales
            var y = d3.scaleBand()
                .range([0,350])
                .domain(d3.range(numRows));

            var x = d3.scaleBand()
                .range([0, 200])
                .domain(d3.range(numCols));

            //the data is just an array of numbers for each cell in the grid
            var nums = d3.range(cumulativeFrequency);

            //container to hold the grid
            var container = svg.append("g")
                .attr("transform", "translate(120,120)");

            container.selectAll("use")
                    .data(nums)
                    .enter().append("use")
                    .attr("xlink:href", "#personIcon")
                    .attr("id", function(d){return "id"+d;})
                    .attr('x', function(d){return x(d%numCols);})
                    .attr('y', function(d){return y(Math.floor(d/numCols));})
                    .attr('fill', getColor);
                    // .style('stroke', 'black');
        });

    }

    onMount(() => {
        drawGraph();
    });
</script>