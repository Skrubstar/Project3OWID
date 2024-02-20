<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let energyData = [];

    let dataset = [];
    let width = 1280;
    let height = 720;
        Promise.all([d3.csv("recent_energy_data.csv"),
        d3.json('geojson.json')])
            .then(([energyData, dataset]) => {
                for (var i = 0; i < energyData.length; i++) {
                    var iso_country_code = energyData[i].iso_country;
                    var index = energyData[i].index;
                    for (var j = 0; j < dataset.features.length; j++) {
                        var country_code_json = dataset.features[j].properties.iso_a3
                        if (country_code_json == iso_country_code) {
                            dataset.features[j].properties.corresponding_index = index;
                            break;
                            }
                        }
                    }
                console.log(dataset)
                var countries = g.selectAll("path")
                    .data(dataset.features)
                    .enter().append("path")
                    .attr("d", path)
                    .attr("fill", "lightgray")
                    .attr("stroke", "white");
            });

        let svg = d3.select("body")
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .style("display", "block")
            .style("margin", "auto")
        var g = svg.append('g');
        var projection = d3.geoNaturalEarth1()
            .scale(250)
            .center([-30, 35])
        var path = d3.geoPath().projection(projection);

</script>

<main>
    <h1>Hello Team!</h1>
    <p>Svelte Testing.</p>
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
