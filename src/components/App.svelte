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
        for (const d of csvData) {
            if (+d.year === selectedYear) {
                if (selectedColumn === "Population") {
                    data.set(d.iso_code, +d.population);
                } else if (selectedColumn === "GDP") {
                    data.set(d.iso_code, +d.gdp)
                } else if (selectedColumn === "Biofuel Consumption") {
                    data.set(d.iso_code, +d.biofuel_consumption)
                } else if (selectedColumn === "Coal Consumption") {
                    data.set(d.iso_code, +d.coal_consumption)
                } else if (selectedColumn === "Fossil Fuel Consumption") {
                    data.set(d.iso_code, +d.fossil_fuel_consumption)
                } else if (selectedColumn === "Gas Consumption") {
                    data.set(d.iso_code, +d.gas_consumption)
                } else if (selectedColumn === "Hydro Consumption") {
                    data.set(d.iso_code, +d.hydro_consumption)
                } else if (selectedColumn === "Nuclear Consumption") {
                    data.set(d.iso_code, +d.nuclear_consumption)
                } else if (selectedColumn === "Oil Consumption") {
                    data.set(d.iso_code, +d.oil_consumption)
                } else if (selectedColumn === "Renewables Consumption") {
                    data.set(d.iso_code, +d.renewables_consumption)
                } else if (selectedColumn === "Solar Consumption") {
                    data.set(d.iso_code, +d.solar_consumption)
                } else if (selectedColumn === "Wind Consumption") {
                    data.set(d.iso_code, +d.wind_consumption)
                } else if (selectedColumn === "Greenhouse Gas Emissions") {
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
        } else if (selectedColumn === "GDP") {
            colorScale = d3.scaleLinear()
            .domain([1000000000, 10000000000, 100000000000, 1000000000000, 1000000000000, 10000000000000])
            .range(d3.schemeGreens[7]);
        } else if (selectedColumn === "Biofuel Consumption") {
            colorScale = d3.scaleLinear()
            .domain([5, 10, 50, 100, 200, 400])
            .range(d3.schemePurples[7]);
        } else if (selectedColumn === "Coal Consumption") {
            colorScale = d3.scaleLinear()
            .domain([100, 500, 1000, 10000, 15000, 20000])
            .range(d3.schemeReds[7]);
        } else if (selectedColumn === "Fossil Fuel Consumption") {
            colorScale = d3.scaleLinear()
            .domain([500, 1000, 5000, 10000, 20000, 30000])
            .range(d3.schemeBlues[7]);
        } else if (selectedColumn === "Gas Consumption") {
            colorScale = d3.scaleLinear()
            .domain([100, 500, 1000, 4000, 6000, 8000])
            .range(d3.schemeGreens[7]);
        } else if (selectedColumn === "Hydro Consumption") {
            colorScale = d3.scaleLinear()
            .domain([100, 200, 500, 1000, 2000, 3000])
            .range(d3.schemePurples[7]);
        } else if (selectedColumn === "Nuclear Consumption") {
            colorScale = d3.scaleLinear()
            .domain([100, 300, 500, 1000, 1500, 2100])
            .range(d3.schemeReds[7]);
        } else if (selectedColumn === "Oil Consumption") {
            colorScale = d3.scaleLinear()
            .domain([100, 500, 1000, 4000, 7000, 10000])
            .range(d3.schemeBlues[7]);
        } else if (selectedColumn === "Renewables Consumption") {
            colorScale = d3.scaleLinear()
            .domain([100, 500, 1500, 3000, 4000, 5000])
            .range(d3.schemeGreens[7]);
        } else if (selectedColumn === "Solar Consumption") {
            colorScale = d3.scaleLinear()
            .domain([1, 10, 50, 100, 200, 500])
            .range(d3.schemePurples[7]);
        } else if (selectedColumn === "Wind Consumption") {
            colorScale = d3.scaleLinear()
            .domain([1, 10, 40, 100, 500, 1000])
            .range(d3.schemeReds[7]);
        } else if (selectedColumn === "Greenhouse Gas Emissions") {
            colorScale = d3.scaleLinear()
            .domain([10, 100, 500, 1000, 2000, 4000])
            .range(d3.schemeBlues[7]);
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
    <h3>Write-up <a href="https://docs.google.com/document/d/1lHdUQZcc8RADYanFeSNzh0y4YZ2MpvFf/edit?usp=sharing&ouid=116419232595005144189&rtpof=true&sd=true" target="_blank">here</a>.</h3>
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