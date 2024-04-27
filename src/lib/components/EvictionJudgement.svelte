<div class='v3-words'>
    <h3>Corporate vs Private Eviction Distribution</h3>
</div>

<div class="chart-container step">
    <div class="chart" id="eviction-judgement"></div>
</div>

<script lang="ts">
    import { onMount } from 'svelte';
    import scrollama from "scrollama";
    import * as d3 from 'd3';

    let svg, x, y;
    const numCols = 40;
    let final_data;

    let drawGraph = function(){

        //number of icons to color in to visualize percent
        var percentNumber = 92;

        //variables for the font family, and some colors
        var fontFamily = "helvetica";
        var twitterFill = "#4D908E";
        var twitterFillActive = "#adf7b6";
        var svgBackgroundColor = '#264653';

        const width = 800;
        const height = 900;

        //create an svg with width and height
        svg = d3.select('#eviction-judgement')
            .append('svg')
            .attr("width", width)
            .attr("height", height);
            // .style('background-color', svgBackgroundColor);

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
                '0': 'steelblue',
                '1': 'red',
                '2': 'green',
                '3': 'orange'
            };
            const getColor = function(d, i){
                if (d.hasOwnProperty('judge_cat')){
                    return colors[d.judge_cat];
                }

                // // loop over cumulitiveData
                // for (let key in cumulitiveData){
                //     if (d < cumulitiveData[key]){
                //         return colors[key];
                //     }
                // }
                return 'brown';
            }

            //10 rows and 10 columns 
            var numRows = Math.ceil(cumulativeFrequency/numCols);

            //x and y axis scales
            y = d3.scaleBand()
                .range([0,height - 150])
                .domain(d3.range(numRows * 2));

            x = d3.scaleBand()
                .range([0, width - 150])
                .domain(d3.range(numCols + 1));

            //the data is just an array of numbers for each cell in the grid
            final_data = [];
            var corp_idx = 0;
            var non_corp_idx = 0;
            data.sort(function(a, b){
                return a.is_ptf_corp.localeCompare(b.is_ptf_corp); // First sort by corp ownership status
            });
            data.sort(function(a, b){
                return a.judge_cat.localeCompare(b.judge_cat); // Final sort by judgement type
            });
            data.forEach(function(d){
                for (var i = 0; i < parseInt(d.count); i++){
                    final_data.push({
                        "judge_cat": parseInt(d.judge_cat),
                        "def_att": d.def_att,
                        "is_ptf_corp": d.is_ptf_corp,
                        "corp_non_corp_idx": d.is_ptf_corp == 'True' ? corp_idx : non_corp_idx
                    });
                    if (d.is_ptf_corp == 'True'){
                        corp_idx++;
                    } else {
                        non_corp_idx++;
                    }
                }
            });

            // final_data.sort(function(a, b){
            //     return a.judge_cat - b.judge_cat;
            // });

            //container to hold the grid
            var container = svg.append("g")
                .attr("transform", "translate(120,120)");
            console.log({final_data});

            container.selectAll("use")
                    .data(final_data)
                    .enter().append("use")
                    .attr("xlink:href", "#personIcon")
                    .attr("id", function(d, i){return "id"+i;})
                    .attr('x', function(d, i){return x(i%numCols);})
                    .attr('y', function(d, i){return y(Math.floor(i/numCols));})
                    .attr('fill', getColor);
                    // .style('stroke', 'black');
        });

    }

    onMount(() => {
        drawGraph();
        const scroller = scrollama();
        scroller
            .setup({
                step: ".step",
                progress: true,
            })
            .onStepEnter((response) => {
                // { element, index, direction }
                console.log({response});
                console.log(svg.selectAll("use"));
                if (final_data === undefined){
                    return;
                }
                var corp_idx = 0;
                var non_corp_idx = 0;
                const non_corp_offset = 21;
                var numCols = 20;
                // svg.selectAll("use")
                //     .data(final_data)
                //     .join('use')
                //     .transition()
                //     .delay(function(d, i){return(i*3)})
                //     .duration(2000)
                //     .attr("x", function (d, i) {
                //         var row = d.corp_non_corp_idx % numCols;
                //         if (d.is_ptf_corp == 'False'){
                //             row = non_corp_offset + row;
                //         }
                //         return x(row);
                //     })
                //     .attr("y", function (d, i) {
                //         return y(Math.floor(d.corp_non_corp_idx/numCols));
                //     });
            })
            .onStepProgress((response) => {
                // { element, index, progress }
                console.log(response);
                if (final_data === undefined){
                    return;
                }
                if (response.progress > 0.3) {
                    const non_corp_offset = 21;
                    var numCols = 20;
                    svg.selectAll("use")
                        .data(final_data)
                        .join('use')
                        .transition()
                        .delay(function(d, i){return(i*3)})
                        .duration(2000)
                        .attr("x", function (d, i) {
                            var row = d.corp_non_corp_idx % numCols;
                            if (d.is_ptf_corp == 'False'){
                                row = non_corp_offset + row;
                            }
                            return x(row);
                        })
                        .attr("y", function (d, i) {
                            return y(Math.floor(d.corp_non_corp_idx/numCols));
                        });
                } else {
                    var numCols = 40;
                    svg.selectAll("use")
                        .data(final_data)
                        .join('use')
                        .transition()
                        .delay(function(d, i){return(i*3)})
                        .duration(2000)
                        .attr('x', function(d, i){return x(i%numCols);})
                        .attr('y', function(d, i){return y(Math.floor(i/numCols));})
                }
            })
            .onStepExit((response) => {
                // { element, index, direction }
                
            });
    });
</script>

<style>
    .chart-container {
        display: flex;
        /* height: 1600px; */
        position: relative;
        justify-content: center;
    }

    .v3-words {
        display: flex;
        justify-content: center;    
    }

    #eviction-judgement {
        position: sticky;
        top: 0;
        left: 0;
    }
</style>