<script>
    import { onMount, afterUpdate, tick } from 'svelte';
    import * as d3 from 'd3';

    let geoJsonPath = 'world.geojson';
    let dataPath = 'owid-energy-data.csv';
    let csvData;
    let geojsonData;
    let draw_choropleth;
    let ready = false;

    async function loadData() {
        const response = await fetch(dataPath); // Path to your CSV file
        const data = await response.text(); // Get the text content of the response
        csvData = d3.csvParse(data);
        const json_response = await fetch(geoJsonPath);
        geojsonData = await json_response.json();
    }

    let svg, tooltip;
    let width;
    let height;
    let columns = [
        "Population",
        "GDP",
        "Biofuel Consumption",
        "Coal Consumption",
        "Fossil Fuel Consumption",
        "Gas Consumption",
        "Hydro Consumption",
        "Nuclear Consumption",
        "Oil Consumption",
        "Renewables Consumption",
        "Solar Consumption",
        "Wind Consumption",
        "Greenhouse Gas Emissions"
    ];
    let selectedYear = 1950;
    let selectedColumn = "Population";

    function choropleth() {
        svg = d3.select("#my_dataviz"),
        width = +svg.attr("width"),
        height = +svg.attr("height");
        tooltip = d3.select("#tooltip");

        // Map and projection
        const path = d3.geoPath();
        const projection = d3.geoMercator()
        .scale(130)
        .center([-10,40])
        .translate([width / 2, height / 2]);

        // Data and color scale
        let data = new Map()
        let colorScale;
        let maxValue = -Number.MAX_VALUE;
        for (const d of csvData) {
            if (+d.year === selectedYear) {
                if (selectedColumn === "Population") {
                    data.set(d.iso_code, +d.population);
                } else if (selectedColumn === "GDP") {
                    if (+d.gdp > maxValue) {
                        maxValue = +d.gdp;
                    }
                    data.set(d.iso_code, +d.gdp)
                } else if (selectedColumn === "Biofuel Consumption") {
                    if (+d.biofuel_consumption > maxValue) {
                        maxValue = +d.biofuel_consumption;
                    }
                    data.set(d.iso_code, +d.biofuel_consumption)
                } else if (selectedColumn === "Coal Consumption") {
                    if (+d.coal_consumption > maxValue) {
                        maxValue = +d.coal_consumption;
                    }
                    data.set(d.iso_code, +d.coal_consumption)
                } else if (selectedColumn === "Fossil Fuel Consumption") {
                    if (+d.fossil_fuel_consumption > maxValue) {
                        maxValue = +d.fossil_fuel_consumption;
                    }
                    data.set(d.iso_code, +d.fossil_fuel_consumption)
                } else if (selectedColumn === "Gas Consumption") {
                    if (+d.gas_consumption > maxValue) {
                        maxValue = +d.gas_consumption;
                    }
                    data.set(d.iso_code, +d.gas_consumption)
                } else if (selectedColumn === "Hydro Consumption") {
                    if (+d.hydro_consumption > maxValue) {
                        maxValue = +d.hydro_consumption;
                    }
                    data.set(d.iso_code, +d.hydro_consumption)
                } else if (selectedColumn === "Nuclear Consumption") {
                    if (+d.nuclear_consumption > maxValue) {
                        maxValue = +d.nuclear_consumption;
                    }
                    data.set(d.iso_code, +d.nuclear_consumption)
                } else if (selectedColumn === "Oil Consumption") {
                    if (+d.oil_consumption > maxValue) {
                        maxValue = +d.oil_consumption;
                    }
                    data.set(d.iso_code, +d.oil_consumption)
                } else if (selectedColumn === "Renewables Consumption") {
                    if (+d.renewables_consumption > maxValue) {
                        maxValue = +d.renewables_consumption;
                    }
                    data.set(d.iso_code, +d.renewables_consumption)
                } else if (selectedColumn === "Solar Consumption") {
                    if (+d.solar_consumption > maxValue) {
                        maxValue = +d.solar_consumption;
                    }
                    data.set(d.iso_code, +d.solar_consumption)
                } else if (selectedColumn === "Wind Consumption") {
                    if (+d.wind_consumption > maxValue) {
                        maxValue = +d.wind_consumption;
                    }
                    data.set(d.iso_code, +d.wind_consumption)
                } else if (selectedColumn === "Greenhouse Gas Emissions") {
                    if (+d.greenhouse_gas_emissions > maxValue) {
                        maxValue = +d.greenhouse_gas_emissions;
                    }
                    data.set(d.iso_code, +d.greenhouse_gas_emissions)
                }
            }
        }
        let topo = geojsonData;
        
        d3.select("svg").selectAll("*").remove();
        if (selectedColumn === "Population") {
            colorScale = d3.scaleThreshold()
            .domain([100000, 1000000, 10000000, 30000000, 100000000, 500000000])
            .range(d3.schemeBlues[7]);
        } else {
            colorScale = d3.scaleLinear()
            .domain([-maxValue, maxValue])
            .range(["white", "green"]);
        }
        svg.append("g")
        .selectAll("path")
        .data(topo.features)
        .join("path")
        .attr("d", d3.geoPath().projection(projection))
        .attr("fill", function (d) {
            d.total = data.get(d.id) || 0;
            return colorScale(d.total);
        })
        .on("mouseover", (event, d) => {
            tooltip.style("display", "block")
            .html(`Country: ${d.properties.name}<br>${selectedColumn}: ${data.get(d.id) || "No Data"}`);
        })
        .on("mousemove", (event) => {
            tooltip.style("left", (event.pageX + 10) + "px")
            .style("top", (event.pageY + 10) + "px");
        })
        .on("mouseout", () => {
            tooltip.style("display", "none");
        });
    }

    onMount(() => {
        Promise.all([
            loadData()
        ]).then(function(data) {
            draw_choropleth = choropleth;
            
            draw_choropleth();
            ready = true;
        });
    });

    afterUpdate(() => {
        if (ready) {
            tick().then(() => {
                draw_choropleth();
            });
        }
    });
</script>

<div style="text-align:center;">
    <h2 class="title">{selectedColumn} in {selectedYear}</h2>
    <div class="controls">
        <input type="range" min="1901" max="2018" step="1" bind:value={selectedYear} class="slider" />
        <span class="year-display">{selectedYear}</span>

        <select bind:value={selectedColumn} class="dropdown">
            {#each columns as column}
                <option value={column}>{column}</option>
            {/each}
        </select>
    </div>

    <svg id="my_dataviz" width="1500" height="700"></svg>
    <div id="tooltip" style="position: absolute; display: none; padding: 10px; background-color: white; border: 1px solid #ccc; pointer-events: none;"></div>
</div>

<style>
    #my_dataviz {
        display: block;
        margin: auto;
        background-color: #f4f4f4;
    }
    #tooltip {
        position: absolute;
        text-align: left;
        padding: 8px;
        font: 12px sans-serif;
        background: white;
        border: 1px solid #ccc;
        border-radius: 5px;
        pointer-events: none;
        opacity: 0.9;
        color: black;
    }
    .title {
        font-size: 24px;
        margin-bottom: 20px;
    }
    .controls {
        margin-bottom: 20px;
    }
    .slider {
        width: 50%;
        margin-right: 10px;
    }
    .year-display {
        font-size: 18px;
        font-weight: bold;
        color: #333;
    }
    .dropdown {
        width: 150px;
        padding: 5px;
        font-size: 16px;
        border-radius: 5px;
        border: 1px solid #ccc;
        background-color: #fff;
        color: #333;
    }
</style>