<div id="vis-container">
    <div id="map-container">
        <div id="map" class="map"></div>
    </div>
    <div class="map-overlay" id="legend">
        <div id="type-selector">
            <input type="radio" id="evictions" name="vis-type" value="evictions" checked>
            <label for="evictions">Eviction Numbers</label>
            <input type="radio" id="percent" name="vis-type" value="percent">
            <label for="percent">Eviction Rate</label>
        </div>
        <h2>Number of evictions in { year }</h2>
        <div id="legend-color"></div>
        <div id="legend-label">
            <div class="legend-item" id="low-legend"></div>
            <div class="legend-item" id="high-legend"></div>
        </div>
        <div id="time-slider">
            <h2>Year: <span id="active-year">{ year }</span></h2>
            <input
                id="slider"
                class="row"
                type="range"
                min="2020"
                max="2023"
                step="1"
                value="{ year }"
            />
        </div>
        
        <div id="trend-graph">
            Hover over the map to see the numbers. Click the census tract to see the trend.
        </div>
    </div>
</div>

<script lang="ts">
    import { onMount, onDestroy } from "svelte";
	import * as mapbox from 'mapbox-gl';
    import * as d3 from 'd3';

    const lowColor = '#f7d654';
    const highColor = '#e64302';
    let year = 2020;
    let vis_type = 'evictions';

    function draw_graph(data) {
        console.log({data});
        // set the dimensions and margins of the graph
        var margin = {top: 10, right: 30, bottom: 30, left: 60},
            width = 200 - margin.left - margin.right,
            height = 200 - margin.top - margin.bottom;
        
        let container = document.getElementById('trend-graph');
        container.innerHTML = '';

        // append the svg object to the body of the page
        var svg = d3.select("#trend-graph")
        .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
        .append("g")
            .attr("transform",
                "translate(" + margin.left + "," + margin.top + ")");

        // Add X axis --> it is a date format
        var x = d3.scaleTime()
        .domain(d3.extent(data, function(d) { return d.year; }))
        .range([ 0, width ]);
        svg.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x));

        // Add Y axis
        var y = d3.scaleLinear()
        .domain([0, d3.max(data, function(d) { return +d.evictions; })])
        .range([ height, 0 ]);
        svg.append("g")
        .call(d3.axisLeft(y).ticks(5));

        var bisect = d3.bisector(function(d) { return d.year; }).left;

        var focus = svg
            .append('g')
            .append('circle')
            .style("fill", "none")
            .attr("stroke", highColor)
            .attr('r', 3)
            .style("opacity", 0);

        var focusText = svg
            .append('g')
            .append('text')
            .style("opacity", 0)
            .attr("text-anchor", "left")
            .attr("alignment-baseline", "middle")

        // Add the line
        svg.append("path")
            .datum(data)
            .attr("fill", "none")
            .attr("stroke", highColor)
            .attr("stroke-width", 1.5)
            .attr("d", d3.line()
                .x(function(d) { return x(d.year) })
                .y(function(d) { return y(d.evictions) })
            );
        
        // Create a rect on top of the svg area: this rectangle recovers mouse position
        svg
            .append('rect')
            .style("fill", "none")
            .style("pointer-events", "all")
            .attr('width', width)
            .attr('height', height)
            .on('mouseover', mouseover)
            .on('mousemove', mousemove)
            .on('mouseout', mouseout);


        // What happens when the mouse move -> show the annotations at the right positions.
        function mouseover() {
            focus.style("opacity", 1)
            focusText.style("opacity",1)
        }

        function mousemove(event) {
            // recover coordinate we need
            var x0 = x.invert(d3.pointer(event)[0]);
            console.log({x0});
            var i = bisect(data, x0, 1);
            let selectedData = data[i];
            console.log(selectedData);
            console.log({i});
            focus
            .attr("cx", x(selectedData.year))
            .attr("cy", y(selectedData.evictions))
            focusText
            .html(selectedData.evictions + " evictions")
            .attr("x", x(selectedData.year)+15)
            .attr("y", y(selectedData.evictions))
            }
        function mouseout() {
            focus.style("opacity", 0)
            focusText.style("opacity", 0)
        }
    }

    onMount(() => {
        // Initialize Mapbox map
        const map = new mapbox.Map({
            container: 'map',
            accessToken: 'pk.eyJ1IjoiamVzc3N4IiwiYSI6ImNsdW16ZnIwbzFpbmkya2xobXo1MDJmZ3oifQ.ogmulGwB0XVmVzqZO72KCA',
            style: 'mapbox://styles/mapbox/light-v10',
            zoom: 11,
            center: [-71.0596, 42.3101] // Center on Boston
        });

        // map.scrollZoom.disable();

        // Load GeoJSON data
        map.on('load', () => {
            // Add the GeoJSON data to the map
            d3.json('/datasets/Metro_Boston_Census_Tracts.geojson').then(geojsonData => {
                map.addSource('census-tracts', {
                    type: 'geojson',
                    data: geojsonData,
                    promoteId: 'geoid'
                });
                // Load CSV data
                d3.csv('/datasets/evictions.csv').then(csvData => {
                    // let max_evictions = d3.max(csvData, d => parseFloat(d['2020_eviction']));
                    // let min_evictions = d3.min(csvData, d => parseFloat(d['2020_eviction']));
                    let max_evictions = 0;
                    let min_evictions = 0;
                    let max_percent = 0;
                    let min_percent = 0;
                    // Update GeoJSON data with CSV data
                    map.getSource('census-tracts').setData({
                        type: 'FeatureCollection',
                        features: map.getSource('census-tracts')._data.features.map(function (feature) {
                            var censusTractID = feature.properties.geoid;

                            // Find corresponding data in CSV
                            var evictionData = csvData.find(row => row.GEOID === censusTractID);
                            if (evictionData) {
                                feature.properties.evictions_2020 = parseFloat(evictionData['2020_eviction']);
                                feature.properties.evictions_2021 = parseFloat(evictionData['2021_eviction']);
                                feature.properties.evictions_2022 = parseFloat(evictionData['2022_eviction']);
                                feature.properties.evictions_2023 = parseFloat(evictionData['2023_eviction']);
                                max_evictions = Math.max(
                                    max_evictions,
                                    feature.properties.evictions_2020,
                                    feature.properties.evictions_2021,
                                    feature.properties.evictions_2022,
                                    feature.properties.evictions_2023
                                );
                                min_evictions = Math.min(
                                    min_evictions,
                                    feature.properties.evictions_2020,
                                    feature.properties.evictions_2021,
                                    feature.properties.evictions_2022,
                                    feature.properties.evictions_2023
                                );

                                feature.properties.percent_2020 = parseFloat(evictionData['2020_eviction']) / parseFloat(evictionData['pop']) * 100;
                                feature.properties.percent_2021 = parseFloat(evictionData['2021_eviction']) / parseFloat(evictionData['pop']) * 100;
                                feature.properties.percent_2022 = parseFloat(evictionData['2022_eviction']) / parseFloat(evictionData['pop']) * 100;
                                feature.properties.percent_2023 = parseFloat(evictionData['2023_eviction']) / parseFloat(evictionData['pop']) * 100;

                                if (!isNaN(feature.properties.percent_2020)) {
                                    max_percent = Math.max(
                                        max_percent,
                                        feature.properties.percent_2020,
                                        feature.properties.percent_2021,
                                        feature.properties.percent_2022,
                                        feature.properties.percent_2023
                                    );
                                    min_percent = Math.min(
                                        min_percent,
                                        feature.properties.percent_2020,
                                        feature.properties.percent_2021,
                                        feature.properties.percent_2022,
                                        feature.properties.percent_2023
                                    );
                                }
                                feature.properties.description = `
                                    <b>${feature.properties.geoid}</b>
                                `;
                                feature.id = feature.properties.geoid;
                            }
                            return feature;
                        })
                    });

                    // Add the choropleth layer
                    map.addLayer({
                        id: 'census-tracts',
                        type: 'fill',
                        source: 'census-tracts',
                        paint: {
                            'fill-color': [
                                'interpolate',
                                ['linear'],
                                ['get', vis_type + '_' + year],
                                min_evictions, lowColor,
                                max_evictions, highColor,
                            ],
                            'fill-opacity': 0.9
                        },
                        filter: ['has', 'evictions_2020']
                    });

                    map.addLayer({
                        id: 'census-tracts-outline',
                        type: 'line',
                        source: 'census-tracts',
                        paint: {
                            'line-color': [
                                'case',
                                ['boolean', ['feature-state', 'hover'], false],
                                'rgba(0,0,0,1)', // Increase line width on hover
                                'rgba(0,0,0,0.3)' // Default line width
                            ],
                            'line-width': [
                                'case',
                                ['boolean', ['feature-state', 'hover'], false],
                                1, // Increase line width on hover
                                0.2 // Default line width
                            ]
                        },
                        filter: ['has', 'evictions_2020']
                    });



                    // Create a popup, but don't add it to the map yet.
                    const popup = new mapboxgl.Popup({
                        closeButton: false,
                        closeOnClick: false
                    });

                    map.on('mousemove', 'census-tracts', (e) => {
                        // Change the cursor style as a UI indicator.
                        map.getCanvas().style.cursor = 'pointer';

                        // Copy coordinates array.
                        const props = e.features[0].properties;
                        const coordinates = [props.intptlon, props.intptlat];
                        const description = e.features[0].properties.description;

                        // Ensure that if the map is zoomed out such that multiple
                        // copies of the feature are visible, the popup appears
                        // over the copy being pointed to.
                        while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
                        }

                        // Populate the popup and set its coordinates
                        // based on the feature found.
                        popup.setLngLat(e.lngLat).setHTML(description).addTo(map);

                        var features = map.queryRenderedFeatures({ layers: ['census-tracts-outline'] });
                        features.forEach(feature => {
                            map.setFeatureState({ source: 'census-tracts', id: feature.id }, { hover: false });
                        });

                        map.setFeatureState({ source: 'census-tracts', id: e.features[0].id }, { hover: true });
                    });

                    map.on('mouseleave', 'census-tracts', () => {
                        map.getCanvas().style.cursor = '';
                        popup.remove();
                        // map.removeFeatureState({ source: 'census-tracts', id: map.queryRenderedFeatures({ layers: ['census-tracts'] })[0].id }, 'hover');
                        var features = map.queryRenderedFeatures({ layers: ['census-tracts-outline'] });
                        features.forEach(feature => {
                            map.setFeatureState({ source: 'census-tracts', id: feature.id }, { hover: false });
                        });
                    });

                    map.on('click', 'census-tracts', (e) => {
                        const props = e.features[0].properties;
                        console.log(props);
                        draw_graph([
                            { year: 2020, evictions: props.evictions_2020 },
                            { year: 2021, evictions: props.evictions_2021 },
                            { year: 2022, evictions: props.evictions_2022 },
                            { year: 2023, evictions: props.evictions_2023 }
                        ]);
                    });

                    function updateMap() {
                        const paintProperty = map.getPaintProperty('census-tracts', 'fill-color');
                        paintProperty[2][1] = vis_type + "_" + year;
                        if (vis_type === 'percent') {
                            paintProperty[3] = min_percent;
                            paintProperty[5] = max_percent;
                        } else {
                            paintProperty[3] = min_evictions;
                            paintProperty[5] = max_evictions;
                        }
                        map.setPaintProperty('census-tracts', 'fill-color', paintProperty);
                    }

                    function updateLegend() {
                        const lowLegend = document.getElementById('low-legend');
                        const highLegend = document.getElementById('high-legend');
                        if (vis_type === 'percent') {
                            lowLegend!.textContent = min_percent.toFixed(2) + '%';
                            highLegend!.textContent = max_percent.toFixed(2) + '%';
                        } else {
                            lowLegend!.textContent = min_evictions.toLocaleString();
                            highLegend!.textContent = max_evictions.toLocaleString();
                        }

                        console.log(min_percent, max_percent, min_evictions, max_evictions);
                    }

                    const legend = document.getElementById('legend');
                    const legendColor = document.getElementById('legend-color');
                    // Set background of legend color to be a gradient between lowColor and highColor
                    legendColor!.style.background = `linear-gradient(to right, ${lowColor}, ${highColor})`;
                    // Set text of low and high legend
                    updateLegend();

                    // update hour filter when the slider is dragged
                    document.getElementById('slider')!.addEventListener('input', (event) => {
                        year = parseInt(event.target.value);
                        // update the map
                        updateMap();
                    });

                    const radios = document.querySelectorAll('input[name="vis-type"]');

                    // Loop through each radio button and add an event listener
                    radios.forEach(radio => {
                        radio.addEventListener('change', (event) => {
                            console.log(map.getSource('census-tracts')._data.features);
                            // When a radio button is selected, log its value to the console
                            console.log(event.target.value);
                            vis_type = event.target.value;
                            updateMap();
                            updateLegend();
                        });
                    });
                });
            });
        });
    });
</script>

  <style>
    #vis-container {
        position: relative;
        width: 100%;
        height: 85vh;
        display: flex;
    }
    #map-container {
        position: relative;
        height: 100%;
        flex-grow: 1;
    }
    .map {
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
    }

    .map-overlay {
        /* position: absolute; */
        min-width: 200px;
        max-width: 320px;
        background-color: white;
        padding: 8px 16px;
        border-radius: 4px;
    }

    /* .map-overlay#legend {
        top: 16px;
        right: 16px;
    } */

    #legend-color {
        height: 20px;
        margin-bottom: 8px;
    }

    #legend-label {
        display: flex;
        justify-content: space-between;
        font-size:1.25rem;
    }

    #time-slider input {
        width: 100%;
    }
    
    /* .map-legend {
      position: absolute;
      bottom: 40px;
      right: 40px;
      background-color: white;
      padding: 16px;
      border-radius: 4px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    } */

    .legend-item {
      display: flex;
      align-items: center;
      margin-bottom: 8px;
      font-size:1.25rem;
    }

    /* .legend-color {
      width: 20px;
      height: 20px;
      margin-right: 8px;
    }

    .legend-label {
      font-size: 14px;
    } */
    #trend-graph {
        font-size:1.2rem;
    }
    label {
        font-size:1.2rem;
    }
    h2{
        font-size:2rem;
    }
  </style>