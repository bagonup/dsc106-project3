<script>
  //Imports
  import { onMount } from "svelte";
  import * as d3 from "d3";

  // Load tempData
  let tempData = [];
  let svg;
  let fixdata;
  let groupedData;
  let summedData;

  onMount(async () => {
    const res = await fetch("data_energy.csv");

    const csv = await res.text();

    tempData = d3.csvParse(csv, d3.autoType);

    console.log(tempData);

    fixdata = tempData.filter(function(d) { return d.Continent != null && d.total_consumption != null;});
    console.log(fixdata);

    groupedData = d3.group(fixdata, d => d.Continent);

    summedData = Array.from(d3.rollup(fixdata, v => d3.sum(v, d => d.total_consumption), d => d.Continent));
    let dataWithNamedProperties = summedData.map(([Continent, total_consumption]) => ({Continent, total_consumption}));

    console.log(dataWithNamedProperties);

    // Assuming data is an array of objects with 'country' and 'consumption' properties

var width = 1000;
var height = 1000;
var radius = Math.min(width, height) / 2;

var svg = d3.select('body')
  .append('svg')
  .attr('width', width)
  .attr('height', height)
  .append('g')
  .attr('transform', 'translate(' + width / 2 + ',' + height / 2 + ')');

var color = d3.scaleOrdinal(d3.schemeCategory10);

var pie = d3.pie()
  .value(function(d) { return d.total_consumption; })
  .sort(null);

var path = d3.arc()
  .outerRadius(radius - 10)
  .innerRadius(0);

var label = d3.arc()
  .outerRadius(radius - 40)
  .innerRadius(radius - 40);

var g = svg.selectAll('.arc')
  .data(pie(dataWithNamedProperties))
  .enter().append('g')
  .attr('class', 'arc');

g.append('path')
  .attr('d', path)
  .style('fill', function(d) { return color(d.data.Continent); });

g.append('text')
  .attr('transform', function(d) { return 'translate(' + label.centroid(d) + ')'; })
  .attr('dy', '0.35em')
  .text(function(d) { return d.data.Continent; });


  });
</script>

<!--Website Code-->

<main>
  <h1>The World in Energy</h1>
  <p>
    Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation
  </p>
</main>

<!--Styling for Website. Fonts and other things-->
<style>
  h1 {
    text-align: center;
  }
</style>
