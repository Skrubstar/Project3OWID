<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let energyData = [];
    let dataset = [];
    let countries = [];
    let width = 1280;
    let height = 720;
    let hoveredCountryId = null;
    let year = "2022";
    let yearData = []

    onMount(() => {
        Promise.all([
            d3.csv("owid-energy-data.csv"),
            d3.json('geojson.json')
        ]).then(([energyDataLoaded, geojsonData]) => {
            energyData = energyDataLoaded;
            dataset = geojsonData.features;
            updateCountries(year, energyData);
        });
    });

    const projection = d3.geoNaturalEarth1()
        .scale(250)
        .center([-30, 35]);
    const pathGenerator = d3.geoPath().projection(projection);

    function updateCountries(year, energyData) {
        const yearData = energyData.filter(d => d.year === year);
        for (let i = 0; i < yearData.length; i++) {
            const iso_country_code = yearData[i].iso_code;
            for (let j = 0; j < dataset.length; j++) {
                const country_code_json = dataset[j].properties.iso_a3;
                if (country_code_json === iso_country_code) {
                    dataset[j].properties.yearData = yearData[i];
                    break;
                }
            }
        }

        countries = dataset.map(feature => ({
            id: feature.properties.iso_a3,
            path: pathGenerator(feature),
            yearData: feature.properties.yearData
        }
        ));
    }

    function hoveredCountryData(){
        const country = countries.find(c => c.id == hoveredCountryId);
        console.log(country)
        return country ? country.yearData : null;
    }

    $: if (year >= 1950 && year <= 2022) {
        updateCountries(String(year), energyData);
    }
</script>
<input type="number" min="1950" max="2022" bind:value={year} />

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
