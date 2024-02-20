<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let energyData = [];
    let dataset = [];
    let countries = [];
    let width = 1280;
    let height = 720;
    let hoveredCountryId = null;

    onMount(() => {
        Promise.all([
            d3.csv("recent_energy_data.csv"),
            d3.json('geojson.json')
        ]).then(([energyData, geojsonData]) => {
            for (let i = 0; i < energyData.length; i++) {
                const iso_country_code = energyData[i].iso_code;
                for (let j = 0; j < geojsonData.features.length; j++) {
                    const country_code_json = geojsonData.features[j].properties.iso_a3;
                    if (country_code_json === iso_country_code) {
                        geojsonData.features[j].properties.energyData = energyData[i];
                        break;
                    }
                }
            }
            dataset = geojsonData.features;
            updateCountries();
        });
    });

    const projection = d3.geoNaturalEarth1()
        .scale(250)
        .center([-30, 35]);
    const pathGenerator = d3.geoPath().projection(projection);

    function updateCountries() {
        countries = dataset.map(feature => ({
            id: feature.properties.iso_a3,
            path: pathGenerator(feature),
            energyData: feature.properties.energyData
        }));
    }

    function hoveredCountryData(){
        const country = countries.find(c => c.id === hoveredCountryId);
        console.log(country)
        return country ? country.energyData : null;
    }
</script>

<main>
    <svg width={width} height={height}>
        {#each countries as {id, path}}
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <path 
            d={path}
            fill="lightgray"
            stroke="white"
            on:mouseenter={() => hoveredCountryId = id}
            on:mouseleave={() => hoveredCountryId = null}
        />
        {/each}
    </svg>
    {#if hoveredCountryId}
    <div>
        Hovered Country ID: {hoveredCountryId}
        {#if hoveredCountryData()}
        <div>
            Energy Consumption: {hoveredCountryData().energyConsumption}
        </div>
        {/if}
    </div>
    {/if}
</main>

<style>
    * {
        text-align: center;
    }
    path {
        fill: lightgray;
        stroke: darkgreen;
    }
    path:hover {
        fill: darkgreen;
    }
</style>
