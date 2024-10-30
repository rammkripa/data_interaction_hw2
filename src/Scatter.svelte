<script>
    import * as d3 from 'd3';

    export let data;
    export let fullData;
    export let variable1;
    export let variable2;
    export let update;

    let width = 400;
    let height = 400;
    let margin = { top: 10, right: 30, bottom: 30, left: 40 };
    let chartW = width - margin.left - margin.right - 5;
    let chartH = height - margin.top - margin.bottom - 5;

    let xAxis;
    let yAxis;

    export let selectedPoints = []; // To store the currently selected point

    function handlePointClick(point) {
        // Check if the point is already selected
        const index = selectedPoints.findIndex(p => p === point);

        if (index === -1) {
            // Add the point if not already selected
            selectedPoints = [...selectedPoints, point];
        } else {
            // Remove the point if already selected
            selectedPoints = selectedPoints.filter((_, i) => i !== index);
        }

        update();
    }

    // Define scales
    $: xScale = d3.scaleLinear()
        .range([0, chartW])
        .domain(d3.extent(fullData.map((d) => d.properties[variable1]))).nice();

    $: yScale = d3.scaleLinear()
        .range([chartH, 0])
        .domain(d3.extent(fullData.map((d) => d.properties[variable2]))).nice();

    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale).ticks(5));
        d3.select(yAxis).call(d3.axisLeft(yScale).ticks(5));
    }

    function data_contains(dataPoint){
        return data.includes(dataPoint);
    }
</script>

<main>
    <h2>Scatterplot of {variable2} vs. {variable1}</h2>
    <p>Click on a point to select it. This will then show the census tract associated with that point in the chloropleth map. Select as many points as you want.</p>

    <svg {width} {height}>
        <!-- Define clipping path to prevent points from spilling out -->
        <defs>
            <clipPath id="plotAreaClip">
                <rect width={chartW} height={chartH} />
            </clipPath>
        </defs>

        <g transform="translate({margin.left}, {margin.top})">
            <!-- X Axis -->
            <g transform="translate(0, {chartH})" bind:this={xAxis} />

            <!-- Y Axis -->
            <g bind:this={yAxis} />

            <!-- Points, with clipping path applied -->
            <g clip-path="url(#plotAreaClip)">
                {#each fullData as dataPoint}
                    <circle
                        cx={xScale(dataPoint.properties[variable1])}
                        cy={yScale(dataPoint.properties[variable2])}
                        r={5}
                        fill={data_contains(dataPoint) ? "steelblue" : "grey"}
                        stroke="white"
                        stroke-width="1px"
                        on:click={() => handlePointClick(dataPoint)}
                        on:keydown={(e) => e.key === 'Enter' && handlePointClick(dataPoint)}
                        class:selected={selectedPoints.includes(dataPoint)}
                        role="button"
                        tabindex="0"/>
                {/each}
            </g>
        </g>

        <!-- Axis labels -->
        <text x={width / 2} y={height - 5} text-anchor="middle" fill="white">{variable1}</text>
        <text x={-chartH / 2} y="15" transform="rotate(-90)" text-anchor="middle" fill="white">{variable2}</text>
    </svg>
</main>

<style>
    circle {
        cursor: pointer;
    }

    /* Style for selected point */
    .selected {
        fill: orange;
        stroke: red;
        stroke-width: 2px;
    }
</style>

