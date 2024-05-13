<!-- Input form for user to select demographic -->
<form id='demoform' class='inputForm'>
    <label>
      Racial Demographic:
      <select bind:value="{selectedDemographic}">
        <option value="" disabled>-- Select a demographic --</option>
        <option value="Asian">Asian</option>
        <option value="Black">Black</option>
        <option value="Hispanic">Hispanic</option>
        <option value="White">White</option>
        <option value="Other">Other</option>
      </select>
    </label>
    <button type="submit">Submit</button>
  </form>

  <form id='incomeform' class='inputForm'>
    <label>
      Income:
      <select bind:value="{selectedIncome}">
        <option value="" disabled>-- Select Income Range --</option>
        <option value="lower">Less than $30,000</option>
        <option value="lower-middle">$30,001 - $58,020</option>
        <option value="middle">$58,021 - $94,000</option>
        <option value="upper-middle">$94,001 - $153,000</option>
        <option value="upper">Greater than $153,000</option>
      </select>
    </label>
    <button type="submit">Submit</button>
  </form>


  <div class='v5-container'>
    <div id="v5map" style="width: 100%; height: 680px;"></div>
    <div class="v5-map-overlay" id="legend">

        <div class='slider-container'>
            <div class="s s--multi">
                <div role='radiogroup'
                             class="group-container"
                             aria-labelledby={`label-${uniqueID}`}
                             style="font-size:1.1rem" 
                             id={`group-${uniqueID}`}>
                <div class='slider-legend' id={`label-${uniqueID}`} style='text-align:center'>Select a Map</div>
                    {#each options as option}
                        <input type="radio" class='slider-input' id={`${option}-${uniqueID}`} value={option} bind:group={sliderValue} on:change={handleClickSlider}>
                        <label for={`${option}-${uniqueID}`}>
                            {option}
                        </label> 
                    {/each}
                </div>
            </div>
        </div>

        <h3 class='map5-title'>{ mode }</h3>
        <div id="v5-legend-color"></div>
        <div id="v5-legend-label">
            <div class="v5-legend-item" id="v5-low-legend"></div>
            <div class="v5-legend-item" id="v5-high-legend"></div>
        </div>

        <div class="v5-words">
            {#if sliderValue === 'Eviction Risk'}
                <p>Select a demographic to visualize the eviction risk for that group. Hover over the map to see the specific percentage for each neighborhood.</p>
                <p>While the data for eviction risk can provide insight into the relative trends for different neighborhoods and demographics, it is <b>important to note households are evicted for different reasons beyond race</b>. The correlations do not necessarily imply causation, but can provide insight on how historical systemic inequalities impact groups today.</p>
            {:else}
                <p>Select an income range to visualize the rent burden for that group. Hover over the map to see the percentage and more details for each neighborhood.</p>
                <p>Percentage of households with rent burden refers to the percent of renter households paying more than 30% of their income on housing costs. <b>More than half of Bostonians have to spend more than 30%</b> of their income just to keep themselves housed, meaning families have to choose between paying rent, eating, accessing medicines, or other necessities. Consequences of not paying rent include foreclosure, eviction, and all physical and mental health consquences of homelessness.</p>
                 
            {/if}
        </div>

    </div>
</div>


<script lang="ts">
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import mapboxgl from 'mapbox-gl';
	
    let options = ['Eviction Risk', 'Income Burden'];
	let sliderValue = 'Eviction Risk';
    const uniqueID = Math.floor(Math.random() * 100);

    let lowColor = '#f7d654';
    let highColor = '#e64302';

    let mode = "Eviction Risk";

    let v5map: mapboxgl.Map;
    let selectedDemographic: string = ''; // Selected demographic option
    let selectedIncome: string = ''; // Selected demographic option
    let csvData: any; 
    let overallMin = 0.03; 
    let overallMax = 14.12; 
    let corporateData: any; 
    
    const geoJsonDataRates = { type: 'FeatureCollection', features: {}}; // GeoJSON data for Boston neighborhoods
    const availableNeighborhoods = ['Roslindale', 'Jamaica Plain', 
        'Roxbury', 'South End', 'Back Bay', 'East Boston', 'Charlestown',
        'Beacon Hill', 'Fenway', 'Brighton', 'West Roxbury', 'Hyde Park', 
        'Mattapan', 'Dorchester', 'South Boston', 'Allston'];

    const popup = new mapboxgl.Popup({
        closeButton: false,
        closeOnClick: false
    });

    function handleClickSlider(event: Event){
        if (sliderValue === 'Eviction Risk') {
            mode = "Eviction Risk";
            overallMin = 0.03; 
            overallMax = 14.12; 
            lowColor = '#f7d654';
            highColor = '#e64302';

            document.getElementById('demoform')!.style.display = 'block';
            document.getElementById('incomeform')!.style.display = 'none';
        } else {
            mode = "Income Burden";
            overallMin = 17.84; 
            overallMax = 173.08; 
            lowColor = '#B2E6FF'; 
            highColor = '#1636A9';

            document.getElementById('demoform')!.style.display = 'none';
            document.getElementById('incomeform')!.style.display = 'block';

        }
        updateLegend();
    }

    function handlePopup() {
        v5map.on('mouseenter', 'risk', (e) => {
                // Change the cursor style as a UI indicator.
                v5map.getCanvas().style.cursor = 'pointer';

                // Copy coordinates array.
                const props = e.features[0].properties;
                const coordinates = [props.intptlon, props.intptlat];
                const description = e.features[0].properties.description;

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

    function updateLegend() {
        const lowLegend = document.getElementById('v5-low-legend');
        const highLegend = document.getElementById('v5-high-legend');
        lowLegend!.textContent = overallMin.toFixed(2) + "%";
        highLegend!.textContent = overallMax.toFixed(2) + "%";

        const legendColor = document.getElementById('v5-legend-color');
        legendColor!.style.background = `linear-gradient(to right, ${lowColor}, ${highColor})`;
    }

    function getCorporate(neighborhood: string): string{
        let ownership = 0;
        corporateData.forEach(row => {
            if (row["Neighborhood"] === neighborhood && row['Year'] === '2024') {
                ownership = parseFloat(row["corp_own_rate"]) * 100;
            }
        });
        return `${ownership}`;
    }
  
    onMount(async () => {
        // console.log("mount", sliderValue);
        csvData = await d3.csv('/datasets/Eviction_Filings_Boston_Neighborhood_Data.csv');
        corporateData = await d3.csv('/datasets/Change_Corporate.csv');

        await d3.json('/datasets/Boston_Neighborhoods.geojson').then(data => {

            const filteredFeatures = data.features.filter(feature => {
                return availableNeighborhoods.includes(feature.properties.Name);
            });

            geoJsonDataRates.features = filteredFeatures;

            geoJsonDataRates.features.forEach(feature => {
                feature.properties['calc_rate'] = 0;
                feature.properties['calc_burden'] = 0;
                feature.properties['description'] =  `<p>Neighborhood: ${feature.properties.Name}</p>`;
                feature.properties["avg_rent"] = 0;
                feature.properties["median_income"] = 0;
                feature.properties["percent_burden"] = 0;
            });
        });

        // Initialize MapBox map
        mapboxgl.accessToken = 'pk.eyJ1IjoiamVzc3N4IiwiYSI6ImNsdW16ZnIwbzFpbmkya2xobXo1MDJmZ3oifQ.ogmulGwB0XVmVzqZO72KCA';
        v5map = new mapboxgl.Map({
            container: 'v5map',
            style: 'mapbox://styles/mapbox/light-v11',
            center: [-71.0596, 42.3101], // Boston coordinates
            zoom: 10.7,
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
                    'fill-opacity': .75 // Adjust opacity if needed
                }
            });

            v5map.addLayer({
                id: 'neighborhoods-outline',
                type: 'line',
                source: 'neighborhoods',
                paint: {
                    'line-color': '#000',
                    'line-width': 0.75
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

            handlePopup();            
            updateLegend();

            document.getElementById('demoform')!.addEventListener('submit', (event) => {
                updateMap(event);
                updateLegend();
            });

            document.getElementById('incomeform')!.addEventListener('submit', (event) => {
                updateMapIncome(event);
                updateLegend();
            });

            // map.on('click', 'city-outlines', (e) => {
            //     const neighborhood = e.features[0].properties.Name;
            //     console.log(`You clicked on ${neighborhood}`);
            // });

        });
    });

    function fetchDemographicData(selectedDemographic: string): void {
        try {

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
                    case 'Other':
                        demographicPercent = parseFloat(row["% Other"]);
                        break;
                    default:
                        demographicPercent = 0;
                        break;
                }

                const risk = (evictionFilingRate / 100) * (demographicPercent / 100) * 100;

                geoJsonDataRates.features.forEach(feature => {
                    if (feature.properties.Name === currNeighborhood) {
                        feature.properties.calc_rate = risk;
                    }
                });
            });

            filteredData.forEach(row => {
                geoJsonDataRates.features.forEach(feature => {
                    const calcRate = feature.properties.calc_rate;
                    const corporateOwnership = getCorporate(feature.properties.Name);
                    feature.properties.description = `<p>Neighborhood: ${feature.properties.Name}</p><p>Eviction Risk: ${calcRate.toFixed(2)}%</p><p>Corporate Ownership Rate: ${corporateOwnership}%</p>`;

                });
            });

        } catch (error) {
            console.error("Error fetching or processing data:", error);
            throw error;
        }
    }

    function handleSubmitDemographic(event: Event) {
        event.preventDefault();

        try {
            fetchDemographicData(selectedDemographic);
            v5map.setPaintProperty(
                'risk', 
                'fill-color', 
                ['interpolate', ['linear'], ['get', 'calc_rate'],
                    overallMin, lowColor, 
                    overallMax, highColor
                ]
            );


            } catch (error) {
                console.error('Error handling demographic data:', error);
            }
    }

    function updateMap(event: Event) {
        handleSubmitDemographic(event);
        v5map.getSource('neighborhoods').setData(geoJsonDataRates);
        const paintProperty = v5map.getPaintProperty('risk', 'fill-color');
        v5map.setPaintProperty('risk', 'fill-color', paintProperty);
    }

    function fetchIncomeData(selectedIncome: string): void {
        try {
            // let maxVal = 0;
            // let minVal = 10000;

            const filteredData = csvData.filter(row => availableNeighborhoods.includes(row["Neighborhood"]));

            filteredData.forEach(row => {
                const currNeighborhood = row["Neighborhood"];
                const medianIncome = parseFloat(row["Median Household Income"]);
                const percentBurden = parseFloat(row["% Households With Rent Burden"]);
                const avgRent = parseFloat(row["Average Rent"]);
                let income = 0;

                switch (selectedIncome) {
                    case 'lower':
                        income = 30000;
                        break;
                    case 'lower-middle':
                        income = (30001+58020)/2;
                        break;
                    case 'middle':
                        income = (58021+94000)/2;
                        break;
                    case 'upper-middle':
                        income = (94001+153000)/2;
                        break;
                    case 'upper':
                        income = 153000;
                        break;
                    default:
                        income = 0;
                        break;
                }
                const burden = avgRent * 12 / income * 100;
                // console.log(currNeighborhood, medianIncome, percentBurden, income, burden); 
                // const risk = (evictionFilingRate / 100) * (demographicPercent / 100) * 100;
                // if (burden > maxVal) {
                //     maxVal = burden;
                // }
                // if (burden < minVal) {
                //     minVal = burden;
                // }

                geoJsonDataRates.features.forEach(feature => {
                    if (feature.properties.Name === currNeighborhood) {
                        feature.properties.calc_burden = burden;
                        feature.properties.avg_rent = avgRent;
                        feature.properties.median_income = medianIncome;
                        feature.properties.percent_burden = percentBurden;
                    }
                });
            });

            filteredData.forEach(row => {
                geoJsonDataRates.features.forEach(feature => {
                    const burden = feature.properties.calc_burden;
                    const avgRent = feature.properties.avg_rent;
                    const medianIncome = feature.properties.median_income;
                    const percentBurden = feature.properties.percent_burden;
                    const corporateOwnership = getCorporate(feature.properties.Name);

        
                    feature.properties.description = `<p>Neighborhood: ${feature.properties.Name}</p><p>Selected Income Burden: ${burden.toFixed(2)}%</p><p>Average Rent: $${avgRent}</p><p>Median Income: $${medianIncome}</p><p>Overall Percent Households with Rent Burden: ${percentBurden.toFixed(2)}%</p><p>Corporate Ownership Rate: ${corporateOwnership}%</p>`;

                });
            });

            // console.log(geoJsonDataRates.features);
            // console.log("minmax", minVal, maxVal);
            // return [minVal, maxVal];
        } catch (error) {
            console.error("Error fetching or processing data:", error);
            throw error;
        }
    }

    async function handleSubmitIncome(event: Event) {
        event.preventDefault();

        try {
            fetchIncomeData(selectedIncome);
            v5map.setPaintProperty(
                'risk', 
                'fill-color', 
                ['interpolate', ['linear'], ['get', 'calc_burden'],
                    overallMin, lowColor, 
                    overallMax, highColor
                ]
            );

            } catch (error) {
                console.error('Error handling demographic data:', error);
            }
    }

    function updateMapIncome(event: Event) {
        handleSubmitIncome(event);
        v5map.getSource('neighborhoods').setData(geoJsonDataRates);
        const paintProperty = v5map.getPaintProperty('risk', 'fill-color');
        v5map.setPaintProperty('risk', 'fill-color', paintProperty);
    }

        
</script>

<style>
    #v5map {
        cursor: auto;
    }

    #incomeform {
        display: none;
    }

    .map5-title{
        margin-top: 1rem;
    }

    .slider-container {
        display: flex;
        justify-content: center;
        margin-top: 0.5rem;
    }

    .v5-words {
        font-size: 1.1rem;
        margin-top: 1rem;
        transition: all 0.3s ease;
    }
    
    .inputForm{
        margin-bottom: 1.5rem;
        align-self: center;
        margin-top: 0px;
    }
    
    .v5-map-overlay {
        /* position: absolute; */
        width: 45%;
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
        margin-bottom: 5rem;
    }

    .s--multi .group-container {
        border: none;
        padding: 0;
        white-space: nowrap;
    }

    /* .s--multi legend {
    font-size: 2px;
    opacity: 0;
    position: absolute;
    } */

    .s--multi label {
        display: inline-block;
        line-height: 1.6;
        position: relative;
        z-index: 2;
    }

    .s--multi input {
        opacity: 0;
        position: absolute;
    }

    .slider-input:hover {
        cursor: pointer;
    }

    .s--multi label:first-of-type {
        padding-right: 5em;
    }

    .s--multi label:last-child {
        margin-left: -5em;
        padding-left: 5em;
    }

    /* .s--multi:focus-within label:first-of-type:after {
        box-shadow: 0 0px 8px var(--accent-color);
        border-radius: 1.5em;
    } */

    /* making the switch UI.  */
    .s--multi label:first-of-type:before,
    .s--multi label:first-of-type:after {
        content: "";
        height: 1.25em;
        overflow: hidden;
        pointer-events: none;
        position: absolute;
        vertical-align: middle;
    }

    .s--multi label:first-of-type:before {
        border-radius: 100%;
        z-index: 2;
        position: absolute;
        width: 1.2em;
        height: 1.2em;
        background: #fff;
        top: 0.2em;
        right: 1.2em;
        transition: transform 0.3s;
    }

    .s--multi label:first-of-type:after {
        background: var(--accent-color);
        border-radius: 1em;
        margin: 0 1em;
        transition: background .2s ease-in-out;
        width: 3em;
        height: 1.6em;
    }

    .s--multi input:first-of-type:checked ~ label:first-of-type:after {
        background: var(--gray);
    }

    .s--multi input:first-of-type:checked ~ label:first-of-type:before {
        transform: translateX(-1.4em);
    }

    .s--multi input:last-of-type:checked ~ label:last-of-type {
        z-index: 1;
    }

    /* .s--multi input:focus {
        box-shadow: 0 0px 8px var(--accent-color);
        border-radius: 1.5em;
    } */

    :root {
		--accent-color: rgb(91 125 233);
		--gray: #ffae45;
	}

    select {
    /* width: 100%; */
    padding: 0.5rem; /* Adjust padding as needed */
    font-size: 1.2rem; /* Adjust font size as needed */
    border: none; /* Add a border */
    outline: none; /* Remove default outline */
    border-bottom: 2px solid #403E3A;
    color: #403E3A;
    background: none;
    margin-right: 1rem;
  }

  /* Style the select element when focused */
  select:hover {
    cursor: pointer;
  }

  /* Style the input button */
  button[type="submit"] {
    background-color: #072B69;
    color: #fff;
    font-size: 1rem; /* Adjust font size as needed */
    padding: 0.5rem 2rem; /* Adjust padding as needed */
    border: none;
    border-radius: 10rem;
    cursor: pointer;
    transition: background-color 0.3s; /* Add transition for background color */
  }

  /* Style the input button on hover */
  button[type="submit"]:hover {
    background-color: #feb64a; /* Change background color on hover */
  }
    
</style>