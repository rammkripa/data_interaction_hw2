<script>
    import * as d3 from 'd3';
    import {legendColor} from 'd3-svg-legend';
    export let data = [];
    export let fullData = [];
    let width = 500;
    let height = 500;
    let proj = d3.geoMercator()
    .scale(40000)
    .center([-87.39, 41.52])
    .translate([width, height]);

    let path = d3.geoPath().projection(proj);
    // reactive functionality
    $: scale = d3.scaleSequential(d3.interpolateGreens)
        .domain(d3.extent(data.map((d) => (+d.properties.population))))

    let legend;
    $: {
        const colorLegend = legendColor()
        .shape('rect')
        .cells(9)
        .scale(scale);

    d3.select(legend)
        .call(colorLegend);

    }
</script>

<main>

    <svg {width} {height}>
        {#each data as d}
          <path class="geo" d={path(d)} style="fill: {scale(+d.properties.population)}"></path>
        {/each}
        <g transform="translate({width-100}, 50)" bind:this={legend}></g>
    </svg>

</main>

<style>
    .geo {
    stroke: gray;
    stroke-width: 1px;
  }
  svg {
    color: maroon;
  }
  
</style>