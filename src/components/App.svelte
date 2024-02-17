<script>
    import * as d3 from 'd3';

    let energyData = [];

    onMount(async () => {

     const res = await fetch('owid-energy-data.csv'); 

     const csv = await res.text();

     energyData = d3.csvParse(csv, d3.autoType)

     console.log(energyData);

 });
</script>

<main>
    <!-- Add HTML elements here -->
</main>

<style>
    main {
        // Add CSS styling here, scoped to only work on App.svelte
    }
</style>
