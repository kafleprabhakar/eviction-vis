
<div id="map" class="map"></div>
<div class="map-overlay" id="legend">
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
            value="2020"
		/>
	</div>
</div>

<script lang="ts">
    import { onMount, onDestroy } from "svelte";
	import * as mapbox from 'mapbox-gl';
    import * as d3 from 'd3';
    // import type { TemporalMap } from "./temporalMap.type";

    const lowColor = '#F8C80B';
    const highColor = '#ED5701';
    let year = 2020;

    onMount(() => {
        // Initialize Mapbox map
        const map = new mapbox.Map({
            container: 'map',
            accessToken: 'pk.eyJ1IjoiamVzc3N4IiwiYSI6ImNsdW16ZnIwbzFpbmkya2xobXo1MDJmZ3oifQ.ogmulGwB0XVmVzqZO72KCA',
            style: 'mapbox://styles/mapbox/light-v10',
            zoom: 11,
            center: [-71.0596, 42.3101] // Center on Boston
        });

        // Load GeoJSON data
        map.on('load', () => {
            // Add the GeoJSON data to the map
            d3.json('src/datasets/Metro_Boston_Census_Tracts.geojson').then(geojsonData => {
                map.addSource('census-tracts', {
                    type: 'geojson',
                    data: geojsonData,
                    promoteId: 'geoid'
                });
                // Load CSV data
                d3.csv('src/datasets/evictions.csv').then(csvData => {
                    // let maxEvictions = d3.max(csvData, d => parseFloat(d['2020_eviction']));
                    // let minEvictions = d3.min(csvData, d => parseFloat(d['2020_eviction']));
                    let maxEvictions = 0;
                    let minEvictions = 0;
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
                                maxEvictions = Math.max(
                                    maxEvictions,
                                    feature.properties.evictions_2020,
                                    feature.properties.evictions_2021,
                                    feature.properties.evictions_2022,
                                    feature.properties.evictions_2023
                                );
                                minEvictions = Math.min(
                                    minEvictions,
                                    feature.properties.evictions_2020,
                                    feature.properties.evictions_2021,
                                    feature.properties.evictions_2022,
                                    feature.properties.evictions_2023
                                );
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
                                ['get', 'evictions_2020'],
                                minEvictions, lowColor,
                                maxEvictions, highColor,
                            ],
                            'fill-opacity': 0.7
                        },
                        filter: ['has', 'evictions_2020']
                    });

                    map.addLayer({
                        id: 'census-tracts-outline',
                        type: 'line',
                        source: 'census-tracts',
                        paint: {
                            'line-width': [
                                'case',
                                ['boolean', ['feature-state', 'hover'], false],
                                1, // Increase line width on hover
                                0 // Default line width
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

                    const legend = document.getElementById('legend');
                    const lowLegend = document.getElementById('low-legend');
                    const highLegend = document.getElementById('high-legend');
                    const legendColor = document.getElementById('legend-color');
                    // Set background of legend color to be a gradient between lowColor and highColor
                    legendColor!.style.background = `linear-gradient(to right, ${lowColor}, ${highColor})`;
                    // Set text of low and high legend
                    lowLegend!.textContent = minEvictions.toLocaleString();
                    highLegend!.textContent = maxEvictions.toLocaleString();

                    // update hour filter when the slider is dragged
                    document.getElementById('slider')!.addEventListener('input', (event) => {
                        year = parseInt(event.target.value);
                        // update the map
                        const paintProperty = map.getPaintProperty('census-tracts', 'fill-color');
                        paintProperty[2][1] = "evictions_" + year;
                        map.setPaintProperty('census-tracts', 'fill-color', paintProperty);
                    });
                });
            });
        });
    });
</script>

  <style>
    .map {
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
    }

    .map-overlay {
        position: absolute;
        background-color: white;
        padding: 8px 16px;
        border-radius: 4px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    .map-overlay#legend {
        bottom: 16px;
        right: 16px;
    }

    #legend-color {
        height: 20px;
        margin-bottom: 8px;
    }

    #legend-label {
        display: flex;
        justify-content: space-between;
    }

    #time-slider input {
        width: 100%;
    }
    
    .map-legend {
      position: absolute;
      bottom: 40px;
      right: 40px;
      background-color: white;
      padding: 16px;
      border-radius: 4px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    .legend-item {
      display: flex;
      align-items: center;
      margin-bottom: 8px;
    }

    .legend-color {
      width: 20px;
      height: 20px;
      margin-right: 8px;
    }

    .legend-label {
      font-size: 14px;
    }
  </style>