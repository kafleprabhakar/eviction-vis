<div class='v3-words'>
    <h3>Corporate vs Private Eviction Distribution</h3>
</div>
<div class="container">
    <div class="sidebar-info">
        <div class="sidebar-info-item" data-step="1">
            <p>In 2023, <span class="focus-text">2600</span> eviction cases were filed in Boston. Of these, the defendent did not show up in 487 instances (or 18.7% of all the cases). They lost the case by default. Of the rest, <span class="focus-text">35%</span> got their case dismissed.</p>
        </div>
        <div class="sidebar-info-item" data-step="2">
            <p>Corporate owners were responsible for <span class="focus-text">81%</span> of the evictions despite owning less than 25% of the properties. Moreover, people facing eviction from corporate owners are <u>less likely to show up to court</u> than non-corporate owners (19.5% vs 15.4%).</p>
        </div>
    </div>
    <div class="primary">
        <div class="chart" id="eviction-judgement">
            <div class="legend">
                {#each Object.entries(legends) as [legend, title]}
                    <div class="legend-item">
                        <svg height="30" width="24">
                            <use xlink:href="#personIcon" x="0" y="0" fill={colors[legend]}></use>
                        </svg>
                        <span>{title}</span>
                    </div>
                {/each}
            </div>
            <div id="corp-non-corp-legend">
                <div>By corporate owners</div>
                <div>By non-corporate owners</div>
            </div>
        </div>
        <div id="footnote">* 1 icon represents {scaleFactor} eviction filings.</div>
    </div>
</div>


<script lang="ts">
    import { onMount } from 'svelte';
    import scrollama from "scrollama";
    import * as d3 from 'd3';

    let svg, x, y;
    const numCols = 35;
    const scaleFactor = 7;
    let final_data;
    const colors = {
        'Default': 'steelblue',
        'Charged': 'red',
        'Dismissal': 'green',
        'Other': 'orange'
    };
    const legends = {
        'Default': 'Defendent did not show up',
        'Charged': 'Landlord won case',
        'Dismissal': 'Case dismissed in favor of defendent',
        'Other': 'Other'
    };

    let drawGraph = function(){

        let container = document.getElementById('eviction-judgement');
        const width = Math.min(container.offsetWidth, 800);
        const height = 600;
        console.log({window})

        //create an svg with width and height
        svg = d3.select('#eviction-judgement')
            .append('svg')
            .attr("width", width)
            .attr("height", height);

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

        // var svg = d3.select("#my_dataviz")

        d3.csv('/datasets/eviction_judgement_23.csv').then(data => {
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

            const getColor = function(d, i){
                if (d.hasOwnProperty('judge_cat')){
                    return colors[d.judge_cat];
                }
                return 'brown';
            }

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
            var totalIcons = 0;
            data.forEach(function(d){
                for (var i = 0; i < parseInt(d.count) / scaleFactor; i++){
                    final_data.push({
                        "judge_cat": d.judge_cat,
                        "def_att": d.def_att,
                        "is_ptf_corp": d.is_ptf_corp,
                        "corp_non_corp_idx": d.is_ptf_corp == 'True' ? corp_idx : non_corp_idx
                    });
                    if (d.is_ptf_corp == 'True'){
                        corp_idx++;
                    } else {
                        non_corp_idx++;
                    }
                    totalIcons++;
                }
            });

            var numRows = Math.ceil(totalIcons/numCols);

            //x and y axis scales
            y = d3.scaleBand()
                .range([0,height])
                .domain(d3.range(numRows + 4));

            x = d3.scaleBand()
                .range([0,width])
                .domain(d3.range(numCols + 1));

            // final_data.sort(function(a, b){
            //     return a.judge_cat - b.judge_cat;
            // });

            //container to hold the grid
            var container = svg.append("g")
                .attr("transform", "translate(0,0)");
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
                step: ".sidebar-info-item",
                progress: true,
            })
            .onStepEnter((response) => {
                // { element, index, direction }
                console.log(response);
                console.log(svg.selectAll("use"));
                if (final_data === undefined){
                    return;
                }
                // var corp_idx = 0;
                // var non_corp_idx = 0;
                // const non_corp_offset = 27;


                if (response.index === 1) {
                    var numColsCorp = 23;
                    var numColsNonCorp = 11;
                    const non_corp_offset = 25;
                    var numCols = 20;
                    svg.selectAll("use")
                        .data(final_data)
                        .join('use')
                        .transition()
                        .delay(function(d, i){return(i)})
                        .duration(1000)
                        .attr("x", function (d, i) {
                            var row;
                            if (d.is_ptf_corp == 'False'){
                                row = d.corp_non_corp_idx % numColsNonCorp + non_corp_offset;
                            } else {
                                row = d.corp_non_corp_idx % numColsCorp;
                            }
                            return x(row);
                        })
                        .attr("y", function (d, i) {
                            if (d.is_ptf_corp == 'False'){
                                return y(Math.floor(d.corp_non_corp_idx/numColsNonCorp));
                            } else {
                                return y(Math.floor(d.corp_non_corp_idx/numColsCorp));
                            }
                        });
                    document.getElementById('corp-non-corp-legend').style.opacity = 1;
                } else {
                    const numCols = 35;
                    console.log({numCols});
                    svg.selectAll("use")
                        .data(final_data)
                        .join('use')
                        .transition()
                        .delay(function(d, i){return(i)})
                        .duration(1000)
                        .attr('x', function(d, i){return x(i%numCols);})
                        .attr('y', function(d, i){return y(Math.floor(i/numCols));});
                    document.getElementById('corp-non-corp-legend').style.opacity = 0;
                }

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
                // console.log(response);
                if (final_data === undefined){
                    return;
                }
                // if (response.progress > 0.4) {
                //     var numColsCorp = 25;
                //     var numColsNonCorp = 13;
                //     const non_corp_offset = 27;
                //     var numCols = 20;
                //     svg.selectAll("use")
                //         .data(final_data)
                //         .join('use')
                //         .transition()
                //         .delay(function(d, i){return(i)})
                //         .duration(2000)
                //         .attr("x", function (d, i) {
                //             var row;
                //             if (d.is_ptf_corp == 'False'){
                //                 row = d.corp_non_corp_idx % numColsNonCorp + non_corp_offset;
                //             } else {
                //                 row = d.corp_non_corp_idx % numColsCorp;
                //             }
                //             return x(row);
                //         })
                //         .attr("y", function (d, i) {
                //             if (d.is_ptf_corp == 'False'){
                //                 return y(Math.floor(d.corp_non_corp_idx/numColsNonCorp));
                //             } else {
                //                 return y(Math.floor(d.corp_non_corp_idx/numColsCorp));
                //             }
                //         });
                // } else {
                //     var numCols = 40;
                //     svg.selectAll("use")
                //         .data(final_data)
                //         .join('use')
                //         .transition()
                //         .delay(function(d, i){return(i)})
                //         .duration(2000)
                //         .attr('x', function(d, i){return x(i%numCols);})
                //         .attr('y', function(d, i){return y(Math.floor(i/numCols));})
                // }
            })
            .onStepExit((response) => {
                // { element, index, direction }
                
            });
    });
</script>

<style>
    .container {
        display: flex;
        padding-top: 1rem;
        padding-bottom: 8rem;
        margin: 0rem 2rem;
    }
    .sidebar-info {
        width: 35%;
        min-width: 400px;
        max-width: 500px;
        padding: 4rem;
        text-align: center;
    }
    .sidebar-info-item {
        height: 100vh;
        display: flex;
        align-items: center;
    }
    .focus-text {
        font-size: 1.5em;
        color: #ED5701;
        font-weight: bold;
    }
    .primary {
        width:65%;
        position: sticky;
        top: 20px;
        left: 0;
        height: 90vh;
    }

    #footnote {
        text-align: right;
        font-size: 14px;
        font-style: italic;
        position: sticky;
        bottom: 20px;
        right: 20px;
        z-index: -1;
    }

    .v3-words {
        display: flex;
        justify-content: center;    
    }

    .legend {
        display: flex;
        justify-content: space-evenly;
        flex-wrap: wrap;
    }

    .legend-item {
        max-width: 25%;
        font-size: 15px;
        margin-bottom: 1rem;
        display: flex;
        align-items: center;
    }
    .legend-item span {
        margin-left: 0.5rem;
        width: calc(100% - 24px);
    }

    #corp-non-corp-legend {
        display: flex;
        justify-content: space-between;
        margin-top: 1rem;
        text-align: center;
        font-size: 16px;
        font-weight: bold;
        margin: 2rem 0 1rem;
        opacity: 0;
    }
    #corp-non-corp-legend div:first-child {
        width: 66%;
    }
    #corp-non-corp-legend div:last-child {
        width: 32%;
    }
</style>