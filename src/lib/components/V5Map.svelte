<!-- Input form for user to select demographic -->
<form on:submit|preventDefault="{handleSubmitDemographic}">
    <label>
      Select Racial Demographic:
      <select bind:value="{selectedDemographic}">
        <option value="">-- Select a demographic --</option>
        <option value="Asian">Asian</option>
        <option value="Black">Black</option>
        <option value="Hispanic">Hispanic</option>
        <option value="White">White</option>
      </select>
    </label>
    <button type="submit">Submit</button>
  </form>

  <form on:submit|preventDefault="{handleSubmitIncome}">
    <label>
      Select Income:
      <select bind:value="{selectedIncome}">
        <option value="">-- Select a demographic --</option>
        <option value="Asian">Asian</option>
        <option value="Black">Black</option>
        <option value="Hispanic">Hispanic</option>
        <option value="White">White</option>
      </select>
    </label>
    <button type="submit">Submit</button>
  </form>

  <div id="v5map" style="width: 90%; height: 600px;"></div>


<script lang="ts">
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import mapboxgl from 'mapbox-gl';
  
    const lowColor = '#f7d654';
    const highColor = '#e64302';

    let v5map: mapboxgl.Map;
    let selectedDemographic: string = ''; // Selected demographic option
    let selectedIncome: string = ''; // Selected demographic option
    let demographicData: any; // Demographic data for selected area
    let neighborhoodDataMap: any; // Neighborhood data for Boston
    const geoJsonDataRates = { type: 'FeatureCollection', features: {}}; // GeoJSON data for Boston neighborhoods

  
    // Function to fetch demographic data based on user input
    async function fetchDemographicData(demographic: string) {
      // Fetch data based on user input
      // Process the data to match area codes/neighborhoods with demographic values
      // Return formatted data
    //   const csvData = await d3.csv('/datasets/Eviction_Filings_Boston_Neighborhood_Data.csv')
        let maxVal = 0; 
        let minVal = 100; 

      d3.csv('/datasets/Eviction_Filings_Boston_Neighborhood_Data.csv').then(csvData => {
        // console.log(csvData);
        // var evictionData = csvData.find(row => row.GEOID === censusTractID);

        // Filter the data based on a condition
        const filteredData = csvData.filter(function(row) {
            // Replace "condition" with your specific filtering condition
            return row["Neighborhood"] !== 'Boston (Total)';
        });


        filteredData.forEach(row => {
            // console.log(row);
            const currNeighborhood = row["Neighborhood"];
            const evictionFilingRate = parseFloat(row["% of Total Eviction Filings, as Fraction of Boston Total"]);
            let demographicPercent = 0; 

            if (selectedDemographic === 'Asian') {
                demographicPercent = parseFloat(row["% Asian"]);
            } else if (selectedDemographic === 'Black') {
                demographicPercent = parseFloat(row["% Black"]);
            } else if (selectedDemographic === 'Hispanic') {
                demographicPercent = parseFloat(row["% Hispanic/Latinx"]);
            } else if (selectedDemographic === 'White') {
                demographicPercent = parseFloat(row["% White"]);
            }
            // console.log(row, evictionFilingRate, demographicPercent);
            // console.log((evictionFilingRate/100) * (demographicPercent/100) * 100);
            const risk = (evictionFilingRate/100) * (demographicPercent/100) * 100;
            if (risk > maxVal) {
                maxVal = risk; 
            }
            if (risk < minVal) {
                minVal = risk; 
            }

            geoJsonDataRates.features.forEach(feature => {
                if (feature.properties.Name === currNeighborhood) {
                    // Update the property of the feature
                    feature.properties.calc_rate = risk;
                }
            });

        // map.setPaintProperty('risk', 'fill-color', {
        //     property: 'calc_risk', // Property containing eviction risk data
        //     stops: [
        //         [0, 'green'],   // Low risk
        //         [0.5, 'yellow'], // Medium risk
        //         [1, 'red']      // High risk
        //     ]
        // });
            
        });
        console.log(geoJsonDataRates);
        console.log(minVal, maxVal);

        v5map.setPaintProperty('risk', 'fill-color', [
            'interpolate',
            ['linear'],
            ['get', 'calc_rate'],
            minVal, lowColor, 
            maxVal, highColor
        ]);

      });

      
    }
  
    onMount(async () => {
        // geoJsonDataRates = d3.json('/datasets/Boston_Neighborhoods.geojson');
        await d3.json('/datasets/Boston_Neighborhoods.geojson').then(data => {
            geoJsonDataRates.features = data.features;

            geoJsonDataRates.features.forEach(feature => {
                feature.properties['calc_rate'] = 0;
            });
        });

        // console.log(geoJsonDataRates);
        // console.log(type(geoJsonDataRates.features);


        // Initialize MapBox map
        mapboxgl.accessToken = 'pk.eyJ1IjoiamVzc3N4IiwiYSI6ImNsdW16ZnIwbzFpbmkya2xobXo1MDJmZ3oifQ.ogmulGwB0XVmVzqZO72KCA';
        v5map = new mapboxgl.Map({
            container: 'v5map',
            style: 'mapbox://styles/mapbox/light-v11',
            center: [-71.0596, 42.3101], // Boston coordinates
            zoom: 10.5
            // scrollZoom: false, 
            // dragPan: false
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

            v5map.addLayer({
                id: 'risk',
                type: 'fill',
                source: 'neighborhoods',
                paint: {
                    'fill-color': 'grey',
                    'fill-opacity': 0.4
                }
            });



        // map.on('click', 'city-outlines', (e) => {
        //     const neighborhood = e.features[0].properties.Name;
        //     console.log(`You clicked on ${neighborhood}`);
        // });

        });
    });
    
  
    // Function to handle form submission
    async function handleSubmitDemographic(event: Event) {
        event.preventDefault();
        demographicData = await fetchDemographicData(selectedDemographic); // Get selected demographic from form
        console.log('seleced', selectedDemographic);
//         const obj = { lst1: 300, lst2: 381, lst3: 4, lst4: 4, lst5: 49 }
// const values = Object.values(obj)
// const lowest = Math.min.apply(null, values)



        // map.setPaintProperty('neighborhoods', 'fill-color', {
        //     property: 'eviction_risk', // Property containing eviction risk data
        //     stops: [
        //         [0, 'green'],   // Low risk
        //         [0.5, 'yellow'], // Medium risk
        //         [1, 'red']      // High risk
        //     ]
        // });

      // Update map visualization with new demographic data
    }

    async function handleSubmitIncome(event: Event) {
        event.preventDefault();
        // const demographic = await fetchDemographicData(selectedDemographic); // Get selected demographic from form
        console.log('selected', selectedIncome);
    }
        
  </script>
