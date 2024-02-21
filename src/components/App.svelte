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

    fixdata = tempData.filter(function (d) {
      return d.Continent != null && d.total_consumption != null;
    });
    console.log(fixdata);

    groupedData = d3.group(fixdata, (d) => d.Continent);

    summedData = Array.from(
      d3.rollup(
        fixdata,
        (v) => d3.sum(v, (d) => d.total_consumption),
        (d) => d.Continent
      )
    );
    let dataWithNamedProperties = summedData.map(
      ([Continent, total_consumption]) => ({ Continent, total_consumption })
    );

    console.log(dataWithNamedProperties);

    // Assuming data is an array of objects with 'country' and 'consumption' properties

    var width = 1800;
    var height = 1000;
    var radius = Math.min(width, height) / 2;

    var svg = d3
      .select("body")
      .append("svg")
      .attr("width", width)
      .attr("height", height)
      .append("g")
      .attr("transform", "translate(" + width / 2 + "," + height / 2 + ")");

    var color = d3.scaleOrdinal(d3.schemeCategory10);
    const tooltip = d3
      .select("body")
      .append("div")
      .attr("class", "tooltip")
      .style("opacity", 0);

    var pie = d3
      .pie()
      .value(function (d) {
        return d.total_consumption;
      })
      .sort(null);

    var path = d3
      .arc()
      .outerRadius(radius - 10)
      .innerRadius(0);

    var label = d3
      .arc()
      .outerRadius(radius - 40)
      .innerRadius(radius - 40);

    var g = svg
      .selectAll(".arc")
      .data(pie(dataWithNamedProperties))
      .enter()
      .append("g")
      .attr("class", "arc");

    g.append("path")
      .attr("d", path)
      .style("fill", function (d) {
        return color(d.data.Continent);
      });

    g.append("text")
      .attr("transform", function (d) {
        const centroid = label.centroid(d);
        const angle = Math.atan2(centroid[1], centroid[0]) * (180 / Math.PI);
        return "translate(" + centroid + ") rotate(" + angle + ")";
      })
      .attr("dy", "0.35em")
      .text(function (d) {
        return d.data.Continent;
      });

    function handleMouseOver(event, d) {
      const cdata = `
    Country: ${d.data.country}<br>
    Biofuel Consumption: ${d.data.biofuel_consumption}<br>
    Coal Consumption: ${d.data.coal_consumption}<br>
    Gas Consumption: ${d.data.gas_consumption}<br>
    Hydro Consumption: ${d.data.hydro_consumption}<br>
    Nuclear Consumption: ${d.data.nuclear_consumption}<br>
    Oil Consumption: ${d.data.oil_consumption}<br>
    Other Renewable Consumption: ${d.data.other_renewable_consumption}<br>
    Solar Consumption: ${d.data.solar_consumption}<br>
    Wind Consumption: ${d.data.wind_consumption}
  `;
      /**const biofuel = d.data.biofuel_consumption;
      const coal = d.data.coal_consumption;
      const gas = d.data.gas_consumption;
      const hydro = d.data.hydro_consumption;
      const nuclear = d.data.nuclear_consumption;
      const oil = d.data.oil_consumption;
      const other = d.data.other_renewable_consumption;
      const solar = d.data.solar_consumption;
      const wind = d.data.wind_consumption;*/

      tooltip.transition().duration(300).style("opacity", 0.95);
      tooltip
        .html(cdata)
        .style("left", event.pageX + 10 + "px")
        .style("top", event.pageY - 28 + "px")
        .style("position", "absolute")
        .style("background-color", "white")
        .style("border", "1px solid #ccc")
        .style("padding", "10px");

      d3.select("body").on("mousemove", (event) => {
        tooltip
          .style("left", event.pageX + 10 + "px")
          .style("top", event.pageY - 28 + "px");
      });
    }
    function handleMouseOut() {
      tooltip.transition().duration(300).style("opacity", 0);
      console.log("mouse is not");
    }
    function constructPieChart(data) {
      d3.select("#newpie svg").remove();
      const newSvg = d3
        .select("#newpie")
        .append("svg")
        .attr("width", width)
        .attr("height", height)
        .append("g")
        .attr("transform", "translate(" + width / 2 + "," + height / 2 + ")");

      const newColor = d3.scaleOrdinal(d3.schemeCategory10);

      const newPie = d3
        .pie()
        .value(function (d) {
          return d.total_consumption;
        })
        .sort(null);

      const newPath = d3
        .arc()
        .outerRadius(radius - 10)
        .innerRadius(0);

      const newLabel = d3
        .arc()
        .outerRadius(radius - 40)
        .innerRadius(radius - 40);

      const newG = newSvg
        .selectAll(".arc")
        .data(newPie(data))
        .enter()
        .append("g")
        .attr("class", "arc");

      newG
        .append("path")
        .attr("d", newPath)
        .style("fill", function (d) {
          return newColor(d.data.country);
        })
        .transition()
        .duration(700)
        .attrTween("d", function (d) {
          const interpolate = d3.interpolate({ startAngle: 0, endAngle: 0 }, d);
          return function (t) {
            return newPath(interpolate(t));
          };
        });

      newG
        .append("text")
        .attr("transform", function (d) {
          const centroid = label.centroid(d);
          const angle = Math.atan2(centroid[1], centroid[0]) * (180 / Math.PI);
          const newAngle = angle - 90;
          const newX = centroid[0] * 0.8;
          const newY = centroid[1] * 0.8;

          if (centroid[1] > 0) {
            return "translate(" + [newX, newY] + ") rotate(" + newAngle + ")";
          } else {
            return "translate(" + [newX, newY] + ")";
          }
        })
        .attr("dy", "0.35em")
        .style("text-anchor", "middle")
        .text(function (d) {
          return d.data.country;
        });

      newG
        .on("mouseover", (event, d) => handleMouseOver(event, d))
        .on("mouseout", () => handleMouseOut());
    }

    g.on("click", (event, d) => handleClick(event, d));

    function handleClick(event, d) {
      const clickedContinent = d.data.Continent;
      const countries = fixdata.filter(
        (country) => country.Continent === clickedContinent
      );
      constructPieChart(countries);
      console.log("constructing new pie map with: " + clickedContinent);
    }
  });
</script>

<!--Website Code-->

<main>
  <h1>The World in Energy</h1>
  <p>
    Visit <a href="https://kit.svelte.dev">z</a> to read the documentation
  </p>
  <div id="newpie"></div>
</main>

<!--Styling for Website. Fonts and other things-->
<style>
  h1 {
    text-align: center;
  }
  div {
    text-align: center;
  }
  bosdy {
    text-align: center;
  }
</style>
