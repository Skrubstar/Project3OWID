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
</script>

<main>
    <h1>Hello Team!</h1>
    <p>Svelte Testing.</p>
</main>

<style>
    main {
        text-align: center;
    }
</style>
