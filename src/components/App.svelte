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
    let yearData = [];
    let barChart = d3.select("#barChart");
    const barChartContainer = d3.select("#barChartContainer")

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

    let selectedOption = '';
    function handleSelectChange(event) {
        selectedOption = event.target.value;
        console.log(selectedOption);    
    }

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

    function showBarChart(countryData) {
        const barChartData = [
            { category: 'Bio-Fuel', value: +countryData.biofuel_consumption },
            { category: 'Coal', value: +countryData.coal_consumption },
            { category: 'Fossil Fuel', value: +countryData.fossil_fuel_consumption },
            { category: 'Gas', value: +countryData.gas_consumption },
            { category: 'Hydro', value: +countryData.hydro_consumption },
            { category: 'Nuclear', value: +countryData.nuclear_consumption },
            { category: 'Oil', value: +countryData.oil_consumption },
            { category: 'Solar', value: +countryData.solar_consumption },
            { category: 'Wind', value: +countryData.wind_consumption },
            { category: 'Other', value: +countryData.other_consumption }
        ];

        const allZeroOrNaN = barChartData.every(d => d.value === 0 || isNaN(d.value));
        if (allZeroOrNaN) {
            barChartContainer.style("display", "none");
            return; 
    }

        const margin = {top: 40, right: 0, bottom: 60, left: 50},
            chartWidth = 350 - margin.left - margin.right,
            chartHeight = 250 - margin.top - margin.bottom;

        const x = d3.scaleBand()
            .range([0, chartWidth])
            .padding(0.1);
        const y = d3.scaleLinear()
            .range([chartHeight, 0]);

        barChart.selectAll("*").remove(); 

        const svg = barChart
            .attr("width", chartWidth + margin.left + margin.right)
            .attr("height", chartHeight + margin.top + margin.bottom)
            .attr("viewBox", `0 0 ${chartWidth + margin.left + margin.right} ${chartHeight + margin.top + margin.bottom}`)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);
        svg.append("text")
            .attr("x", chartWidth / 2)
            .attr("y", -margin.top / 2)
            .attr("text-anchor", "middle")
            .style("font-size", "16px")
            .text(countryData.country);

        x.domain(barChartData.map(d => d.category));
        y.domain([0, d3.max(barChartData, d => d.value)]);

        const colorScale = d3.scaleOrdinal()
            .domain(barChartData.map(d => d.category))
            .range(["blue", "orange", "green", "red", "purple", "brown", "pink", "gray", "cyan", "yellow"]);

        svg.selectAll(".bar")
            .data(barChartData)
        .enter().append("rect")
            .attr("class", "bar")
            .attr("x", d => x(d.category))
            .attr("width", x.bandwidth())
            .attr("y", d => y(d.value))
            .attr("height", d => chartHeight - y(d.value))
            .attr("fill", d => colorScale(d.category)); // Use the color scale for the fill color

        // Add the x-axis
        svg.append("g")
            .attr("transform", `translate(0,${chartHeight})`)
            .call(d3.axisBottom(x))
            .selectAll("text")
            .style("text-anchor", "end")
            .attr("dx", "-.8em")
            .attr("dy", ".15em")
            .attr("transform", "rotate(-90)");


        svg.append("g")
            .call(d3.axisLeft(y));

        barChartContainer.style("display", "block");
        const offset = { x: 20, y: 0 }; // Adjust as needed
        barChartContainer.style("left", (event.pageX + offset.x) + "px")
        .style("top", (event.pageY + offset.y) + "px");        

}

    $: if (year >= 1950 && year <= 2022) {
        updateCountries(String(year), energyData);
    }
    $: if (hoveredCountryId && hoveredCountryData()) {
        showBarChart(hoveredCountryData());
    }
</script>
<input type="number" min="1950" max="2022" bind:value={year} />

<main>
    <label for="dropdown">Choose the energy type:</label>
    <select on:change={handleSelectChange}>
        <option value="">Select an option</option>
        <option value="Biofuel">Biofuel</option>
        <option value="Coal">Coal</option>
        <option value="Fossil Fuel">Fossil Fuel</option>
        <option value="Gas">Gas</option>
        <option value="Hydro">Hydro</option>
        <option value="Nuclear">Nuclear</option>
        <option value="Oil">Oil</option>
        <option value="Solar">Solar</option>
        <option value="Wind">Wind</option>
        <option value="Other Renewable">Other Renewable</option>
    </select>

    {#if selectedOption !== ''}
        <p>The selected option is {selectedOption}. This is calculated in TWh.</p>
    {/if}

    <svg width={width} height={height}>
        {#each countries as {id, path}}
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <path 
            d={path}
            fill="lightgray"
            stroke="white"
            on:mouseenter={() => hoveredCountryId = id}
            on:mouseleave={() => {
                hoveredCountryId = null;
                barChart.selectAll("*").remove();
                barChartContainer.style("display", "none");
            }}
        />
        {/each}
    </svg>

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
