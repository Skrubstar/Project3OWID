<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let energyData = [];

    onMount(async () => {

     const res = await fetch('owid-energy-data.csv'); 

     const csv = await res.text();

     energyData = d3.csvParse(csv, d3.autoType)

     console.log(energyData);

 });
    let width = 1920;
    let height = 1080;
    var svg = d3.select("body")
    .append("svg")
    .attr("width", width)
    .attr("height", height);
    var projection = d3.geoMercator()
        .scale(175)
        .center([-11, 85]);
    var path = d3.geoPath().projection(projection);

    let dataset = [];
    d3.json("geojson.json"
    ).then((data)  => {
    dataset  =  data.features;
    });


</script>

<main>
    <h1>Hello Team!</h1>
    <p>Svelte Testing.</p>
    <svg width={height} height={height}>
        {#each  dataset  as data}
            <path  d={path(data)} />
        {/each}
    </svg>
</main>

<style>
    * {
        text-align:center;
    }
    path {
        fill: none;
        stroke: darkgreen;
    }
</style>
