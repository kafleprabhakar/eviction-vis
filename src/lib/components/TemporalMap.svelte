
<div id="map" class="map"></div>

<script lang="ts">
    import { onMount, onDestroy } from "svelte";
	import * as mapbox from 'mapbox-gl';
    import * as d3 from 'd3';
    // import type { TemporalMap } from "./temporalMap.type";

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
                    map.getSource('census-tracts').setData({
                        type: 'FeatureCollection',
                        features: map.getSource('census-tracts')._data.features.map(function (feature) {
                            var censusTractID = feature.properties.geoid;

                            // Find corresponding data in CSV
                            var evictionData = csvData.find(row => row.GEOID === censusTractID);
                            if (evictionData) {
                                feature.properties.evictions = parseFloat(evictionData['2020_eviction']); // Assuming evictions are in the second column
                                feature.properties.description = `
                                    <b>${feature.properties.geoid}</b>
                                    <p>${feature.properties.evictions.toLocaleString()} evictions</p>
                                `;
                                feature.id = feature.properties.geoid;
                            }

                            return feature;
                        })
                    });
                    // Create a lookup object for the CSV data
                    // csvData.forEach(row => {
                    //     var feature = map.getSource('census-tracts')._data.features.find(feature => feature.properties.geoid === row.GEOID);
                    //     if (feature) {
                    //         feature.properties.evictions = Number(row["2020_eviction"]);
                    //         console.log({feature});
                    //     }
                    // });

                    console.log({csvData});


                    // Add the choropleth layer
                    map.addLayer({
                        id: 'census-tracts',
                        type: 'fill',
                        source: 'census-tracts',
                        paint: {
                            'fill-color': [
                                'interpolate',
                                ['linear'],
                                ['get', 'evictions'],
                                0, '#FFFFFF',
                                50, '#000000',
                            ],
                            'fill-opacity': 0.7
                        },
                        filter: ['has', 'evictions']
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
                        filter: ['has', 'evictions']
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

                    // Add a legend
                    // const legend = document.createElement('div');
                    // legend.classList.add('map-legend');
                    // const legendTitle = document.createElement('h3');
                    // legendTitle.textContent = 'Number of Evictions';
                    // legend.appendChild(legendTitle);

                    // const legendColors = ['#FFEDA0', '#FED976', '#FEB24C', '#FD8D3C', '#FC4E2A', '#E31A1C', '#BD0026'];
                    // const legendStops = [0, 100, 500, 1000, 5000, 10000, 50000];

                    // legendColors.forEach((color, i) => {
                    //     const legendItem = document.createElement('div');
                    //     legendItem.classList.add('legend-item');
                    //     const legendColor = document.createElement('div');
                    //     legendColor.classList.add('legend-color');
                    //     legendColor.style.backgroundColor = color;
                    //     const legendLabel = document.createElement('div');
                    //     legendLabel.classList.add('legend-label');
                    //     legendLabel.textContent = `${legendStops[i].toLocaleString()} +`;
                    //     legendItem.appendChild(legendColor);
                    //     legendItem.appendChild(legendLabel);
                    //     legend.appendChild(legendItem);
                    // });

                    // map.getContainer().appendChild(legend);
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