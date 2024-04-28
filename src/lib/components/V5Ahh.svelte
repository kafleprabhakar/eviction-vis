<!-- Input form for user to select demographic -->
<form id='demoform' class='inputForm'>
    <label>
      Select Racial Demographic:
      <select bind:value="{selectedDemographic}">
        <option value="" disabled>-- Select a demographic --</option>
        <option value="Asian">Asian</option>
        <option value="Black">Black</option>
        <option value="Hispanic">Hispanic</option>
        <option value="White">White</option>
      </select>
    </label>
    <button type="submit">Submit</button>
  </form>

  <!-- <form on:submit|preventDefault="{handleSubmitIncome}">
    <label>
      Select Income:
      <select bind:value="{selectedIncome}">
        <option value="" disabled>-- Select a demographic --</option>
        <option value="Asian">Asian</option>
        <option value="Black">Black</option>
        <option value="Hispanic">Hispanic</option>
        <option value="White">White</option>
      </select>
    </label>
    <button type="submit">Submit</button>
  </form> -->

  <div class='v5-container'>
    <div id="v5map" style="width: 90%; height: 600px;"></div>
    <div class="v5-map-overlay" id="legend">

        <h3>{ mode }</h3>
        <div id="v5-legend-color"></div>
        <div id="v5-legend-label">
            <div class="v5-legend-item" id="v5-low-legend"></div>
            <div class="v5-legend-item" id="v5-high-legend"></div>
        </div>

    </div>
</div>


<script lang="ts">
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';
    import mapboxgl from 'mapbox-gl';
  
    const lowColor = '#f7d654';
    const highColor = '#e64302';

    let mode = "Eviction Risk";

    let v5map: mapboxgl.Map;
    let selectedDemographic: string = ''; // Selected demographic option
    let selectedIncome: string = ''; // Selected demographic option
    let csvData: any; 
    let overallMin = 0; 
    let overallMax = 0; 
    
    const geoJsonDataRates = { type: 'FeatureCollection', features: {}}; // GeoJSON data for Boston neighborhoods
    const availableNeighborhoods = ['Roslindale', 'Jamaica Plain', 
        'Roxbury', 'South End', 'Back Bay', 'East Boston', 'Charlestown',
        'Beacon Hill', 'Fenway', 'Brighton', 'West Roxbury', 'Hyde Park', 
        'Mattapan', 'Dorchester', 'South Boston', 'Allston'];

    const popup = new mapboxgl.Popup({
                closeButton: false,
                closeOnClick: false
    });

    function fetchDemographicData(selectedDemographic: string): [number, number] {
        try {
            let maxVal = 0;
            let minVal = 100;

            const filteredData = csvData.filter(row => availableNeighborhoods.includes(row["Neighborhood"]));

            filteredData.forEach(row => {
                const currNeighborhood = row["Neighborhood"];
                const evictionFilingRate = parseFloat(row["% of Total Eviction Filings, as Fraction of Boston Total"]);
                let demographicPercent = 0;

                switch (selectedDemographic) {
                    case 'Asian':
                        demographicPercent = parseFloat(row["% Asian"]);
                        break;
                    case 'Black':
                        demographicPercent = parseFloat(row["% Black"]);
                        break;
                    case 'Hispanic':
                        demographicPercent = parseFloat(row["% Hispanic/Latinx"]);
                        break;
                    case 'White':
                        demographicPercent = parseFloat(row["% White"]);
                        break;
                    default:
                        demographicPercent = 0;
                        break;
                }

                const risk = (evictionFilingRate / 100) * (demographicPercent / 100) * 100;
                if (risk > maxVal) {
                    maxVal = risk;
                }
                if (risk < minVal) {
                    minVal = risk;
                }

                geoJsonDataRates.features.forEach(feature => {
                    if (feature.properties.Name === currNeighborhood) {
                        feature.properties.calc_rate = risk;
                    }
                });
            });

            filteredData.forEach(row => {
                geoJsonDataRates.features.forEach(feature => {
                    const calcRate = feature.properties.calc_rate;
        
                    feature.properties.description = `<p>Neighborhood: ${feature.properties.Name}</p><p>Eviction Risk: ${calcRate.toFixed(2)}%</p>`;

                });
            });

            console.log(geoJsonDataRates.features);
            // console.log(minVal, maxVal);
            overallMin = minVal;
            overallMax = maxVal;

            return [minVal, maxVal];
        } catch (error) {
            console.error("Error fetching or processing data:", error);
            throw error;
        }
    }

    function updateMap(event: Event) {
        
        handleSubmitDemographic(event);
        v5map.getSource('neighborhoods').setData(geoJsonDataRates);
        const paintProperty = v5map.getPaintProperty('risk', 'fill-color');
        
        v5map.setPaintProperty('risk', 'fill-color', paintProperty);
    }


  
    onMount(async () => {
        csvData = await d3.csv('/datasets/Eviction_Filings_Boston_Neighborhood_Data.csv');

        await d3.json('/datasets/Boston_Neighborhoods.geojson').then(data => {

            const filteredFeatures = data.features.filter(feature => {
                return availableNeighborhoods.includes(feature.properties.Name);
            });

            geoJsonDataRates.features = filteredFeatures;

            geoJsonDataRates.features.forEach(feature => {
                feature.properties['calc_rate'] = 0;
                feature.properties['calc_color'] = '#999999';
                feature.properties['description'] =  
                    `<p>Neighborhood: ${feature.properties.Name}</p>`;
            });
        });

        // Initialize MapBox map
        mapboxgl.accessToken = 'pk.eyJ1IjoiamVzc3N4IiwiYSI6ImNsdW16ZnIwbzFpbmkya2xobXo1MDJmZ3oifQ.ogmulGwB0XVmVzqZO72KCA';
        v5map = new mapboxgl.Map({
            container: 'v5map',
            style: 'mapbox://styles/mapbox/light-v11',
            center: [-71.0596, 42.3101], // Boston coordinates
            zoom: 10.5,
            scrollZoom: false, 
            dragPan: false,
            
        });

        // Add GeoJSON layer for city outlines
        v5map.on('load', () => {
            v5map.addSource('neighborhoods', {
                type: 'geojson',
                data: geoJsonDataRates,
                promoteId: 'Name'
            });

            v5map.addLayer({
                id: 'neighborhoods-outline',
                type: 'line',
                source: 'neighborhoods',
                paint: {
                    'line-color': '#000',
                    'line-width': 1
                }
            });

            v5map.addLayer({
                id: 'neighborhood-labels',
                type: 'symbol',
                source: 'neighborhoods', // Specify the data source
                layout: {
                    'text-field': ['get', 'Name'], // Use the 'neighborhood_name' property for text labels
                    'text-size': 12,
                    'text-anchor': 'top'
                },
                paint: {
                    'text-color': 'black'
                }
            });

            // v5map.addLayer({
            //     'id': 'risk',
            //     'type': 'fill',
            //     'source': 'neighborhoods',
            //     'paint': {
            //         'fill-color': ['get', 'calc_color'],
            //         'fill-opacity': .6 // Adjust opacity if needed
            //     }
            // });

            v5map.addLayer({
                'id': 'risk',
                'type': 'fill',
                'source': 'neighborhoods',
                'paint': {
                    'fill-color': [
                        'interpolate',
                        ['linear'],
                        ['get', 'calc_rate'],
                        0, 'grey',
                        1, 'grey'
                    ],
                    'fill-opacity': .6 // Adjust opacity if needed
                }
            });

            handlePopup();

            function updateLegend() {
                const lowLegend = document.getElementById('v5-low-legend');
                const highLegend = document.getElementById('v5-high-legend');
                lowLegend!.textContent = overallMin.toFixed(2) + "%";
                highLegend!.textContent = overallMax.toFixed(2) + "%";

                // console.log(min_percent, max_percent, min_evictions, max_evictions);
            }

            const legend = document.getElementById('v5-legend');
            const legendColor = document.getElementById('v5-legend-color');
            // Set background of legend color to be a gradient between lowColor and highColor
            legendColor!.style.background = `linear-gradient(to right, ${lowColor}, ${highColor})`;
            // Set text of low and high legend
            updateLegend();

            document.getElementById('demoform')!.addEventListener('submit', (event) => {
                        // year = parseInt(event.target.value);
                        // update the map
                updateMap(event);
                updateLegend();
            });

            // map.on('click', 'city-outlines', (e) => {
            //     const neighborhood = e.features[0].properties.Name;
            //     console.log(`You clicked on ${neighborhood}`);
            // });

        });
    });

    function handlePopup() {
        v5map.on('mouseenter', 'risk', (e) => {
                // Change the cursor style as a UI indicator.
                v5map.getCanvas().style.cursor = 'pointer';

                // Copy coordinates array.
                const props = e.features[0].properties;
                const coordinates = [props.intptlon, props.intptlat];
                const description = e.features[0].properties.description;
                console.log("hi", description);

                // Ensure that if the map is zoomed out such that multiple
                // copies of the feature are visible, the popup appears
                // over the copy being pointed to.
                while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                    coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
                }

                // Populate the popup and set its coordinates
                // based on the feature found.
                popup.setLngLat(e.lngLat).setHTML(description).addTo(v5map);

                });

                v5map.on('mouseleave', 'risk', () => {
                    v5map.getCanvas().style.cursor = 'auto';
                    popup.remove();
                });

    }
 
    
    function handleSubmitDemographic(event: Event) {
        event.preventDefault();
        mode = "Eviction Risk"

        try {
            console.log('selected', selectedDemographic);

            const [minVal, maxVal] = fetchDemographicData(selectedDemographic);
            console.log('minVal:', minVal, 'maxVal:', maxVal);

            v5map.setPaintProperty(
                'risk', 
                'fill-color', 
                ['interpolate', ['linear'], ['get', 'calc_rate'],
                    minVal, lowColor, 
                    maxVal, highColor
                ]
            );


            } catch (error) {
                console.error('Error handling demographic data:', error);
                // Handle errors gracefully, e.g., display an error message to the user
            }
    }

    async function handleSubmitIncome(event: Event) {
        event.preventDefault();
        // const demographic = await fetchDemographicData(selectedDemographic); // Get selected demographic from form
        mode = "Income Burden"
        console.log('selected', selectedIncome);
    }

        
</script>

<style>
    #v5map {
        cursor: auto;
    }

    .inputForm{
        margin-bottom: 3rem;
        align-self: center;
    }
    
    .v5-map-overlay {
        /* position: absolute; */
        min-width: 200px;
        max-width: 320px;
        background-color: white;
        padding: 8px 16px;
        border-radius: 4px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    #v5-legend-color {
        height: 20px;
        margin-bottom: 8px;
    }

    #v5-legend-label {
        display: flex;
        justify-content: space-between;
        font-size:1.25rem;
    }

    .v5-legend-item {
      display: flex;
      align-items: center;
      margin-bottom: 8px;
      font-size:1.25rem;
    }

    .v5-container {
        /* position: relative; */
        /* width: 100%;
        height: 85vh; */
        margin: -8px; /* Offset the 8px margin added by default in body */
        display: flex;
    }
</style>