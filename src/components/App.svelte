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

    let referenceColumns = {
        "Oil" : "oil_electricity",
        "Coal": "coal_electricity",
        "Biofuel": "biofuel_electricity",
        "Nuclear": "nuclear_electricity",
        "Fossil Fuel": "fossil_electricity",
        "Gas": "gas_electricity",
        "Hydro": "hydro_electricity",
        "Solar": "solar_electricity",
        "Wind": "wind_electricity",
    }
    var colors=[];
    let newOption = 0;
    var domain=[];
    let referenceColors = {
        "Solar" : ['#DBF3CC', '#40A600'],
        "Coal" :['#e3c3a3', '#996003'],
        "Fossil Fuel" : ['#FFEEEE', '#FF0000'],
        "Hydro" : ['#DAD7FB', '#3725FA'],
        "Gas" : ['#FFEAF8', '#FF00AB'],
        "Nuclear" : ['#D5FFEA', '#10D874'],
        "Oil" : ['#FFCCE1', '#FF0069'],
        "Biofuel" : ['#D3FEDA', '#00CC20'],
        "Wind" : ['#C7FDFD', '#2470C8']
    }



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

    let selectedOption = 'Fossil Fuel';
    function handleSelectChange(event) {
        selectedOption = event.target.value;
        newOption=0;
        const svgAttr = document.getElementById("worldmap");
        const paths = svgAttr.querySelectorAll("path")
        let reference = referenceColumns[selectedOption]
        let fillvalues=[]
        for (let country in countries) {
                if (countries[country].yearData && countries[country].yearData[reference]) {
                    fillvalues.push(test(countries[country].yearData));
                } else {
                    fillvalues.push(test(undefined))
                }
        }
        for (let i = 0; i< paths.length; i++) {
            paths[i].setAttribute("fill", fillvalues[i])
        }
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
            { category: 'Fossil Fuel', value: +countryData.fossil_electricity},
            { category: 'Biofuel', value: +countryData.biofuel_electricity },
            { category: 'Coal', value: +countryData.coal_electricity },
            { category: 'Gas', value: +countryData.gas_electricity },
            { category: 'Hydro', value: +countryData.hydro_electricity },
            { category: 'Nuclear', value: +countryData.nuclear_electricity },
            { category: 'Oil', value: +countryData.oil_electricity },
            { category: 'Solar', value: +countryData.solar_electricity },
            { category: 'Wind', value: +countryData.wind_electricity }
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
            .range(["#FF0000", "#D3FEDA", "#996003", "#FF00AB", "#3725FA", "#10D874", "#FF0069", "#40A600", "#C7FDFD"]);

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

    function test(yearData) {
    let reference = referenceColumns[selectedOption];
    if (yearData === undefined || yearData[reference] === undefined || yearData[reference] == "") {
        return "grey";
    }
    if (newOption == 0) {
        scaleColors();
    }
    createLegend(); // Call the function to create the legend
    return colors(parseFloat(yearData[reference]));
}

function createLegend() {
    const legend = d3.select("#legend");
    legend.selectAll("*").remove(); // Clear any existing legend

    const legendData = [
        { name: 'High', color: referenceColors[selectedOption][1] },
        { name: 'Medium', color: d3.interpolate(referenceColors[selectedOption][0], referenceColors[selectedOption][1])(0.5) },
        { name: 'Low', color: referenceColors[selectedOption][0] },
        { name: 'No Data', color: 'grey' }
    ];

    // Adjust the position of the legend group
    const legendGroup = legend.append("g")
        .attr("transform", 'translate(0, 110)'); // Adjust these values as needed

    const legendItems = legendGroup.selectAll(".legend-item")
        .data(legendData)
        .enter()
        .append("g")
        .attr("class", "legend-item")
        .attr("transform", (d, i) => `translate(0, ${i * 20})`);

    legendItems.append("rect")
        .attr("width", 18)
        .attr("height", 18)
        .attr("fill", d => d.color);

    legendItems.append("text")
        .attr("x", 24)
        .attr("y", 9)
        .attr("dy", "0.35em")
        .text(d => d.name);

    // Add a border around the legend
    legendGroup.append("rect")
        .attr("x", -5)
        .attr("y", -5)
        .attr("width", 150) // Adjust the width as needed
        .attr("height", legendData.length * 20 + 10) // Adjust the height based on the number of items
        .attr("fill", "none")
        .attr("stroke", "black")
        .attr("stroke-width", 1);
}



    function scaleColors() {
        newOption=1;
        let countries_copy=[];
        let reference = referenceColumns[selectedOption]
        for (let country in countries) {
            if (countries[country].yearData) {
                if (countries[country].yearData[reference]) {
                    countries_copy.push(parseFloat(countries[country].yearData[reference]));
                }
            }
        }
        countries_copy = countries_copy.sort(function (a,b) {
              return a-b;  
            });
        countries_copy = countries_copy.slice(countries_copy.lastIndexOf(0))
        domain = [0, 
            (d3.quantile(countries_copy, 0.99))
            ]
        let range = referenceColors[selectedOption]
        colors = d3.scaleLinear()
            .domain(domain)
            .range(range)
            .clamp(true);
    }     
</script>
<input type="number" min="1950" max="2022" bind:value={year} />

<main>
    <label for="dropdown">Choose the energy type:</label>
    <select on:change={handleSelectChange}>
`       <option value="Fossil Fuel">Fossil Fuel</option>
        <option value="Biofuel">Biofuel</option>
        <option value="Coal">Coal</option>
        <option value="Gas">Gas</option>
        <option value="Hydro">Hydro</option>
        <option value="Nuclear">Nuclear</option>
        <option value="Oil">Oil</option>
        <option value="Solar">Solar</option>
        <option value="Wind">Wind</option>
    </select>

    {#if selectedOption !== ''}
        <p>The selected option is {selectedOption}. This is calculated in TWh.</p>
    {/if}

    <svg id="worldmap" width={width} height={height}>
        {#each countries as {id, path, yearData}}
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <path 
            d={path}
            fill={test(yearData)}
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
        stroke: darkgreen;
    }
</style>
