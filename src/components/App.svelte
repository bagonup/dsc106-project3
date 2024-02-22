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

    var width = window.innerWidth - 100;
    var height = window.innerHeight;
    var radius = Math.min(width, height) / 2;

    var svg = d3
      .select("#continentPie")
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

    // Mapping for Continent Names
    const continentMapping = {
      NA: "North America",
      AS: "Asia",
      EU: "Europe",
      AF: "Africa",
      SA: "South America",
      OC: "Oceania",
    };

    g.append("text")
      .attr("transform", function (d) {
        const centroid = label.centroid(d);
        const angle = ((d.startAngle + d.endAngle) / 2) * (180 / Math.PI) - 90;
        const rotate = angle > 90 && angle < 270 ? angle + 180 : angle;
        const newX = centroid[0] * 0.9;
        const newY = centroid[1] * 0.9;

        return "translate(" + [newX, newY] + ") rotate(" + rotate + ")";
      })
      .attr("dy", "0.35em")
      .style("text-anchor", "middle")
      .text(function (d) {
        return continentMapping[d.data.Continent];
      });

    const backButton = document.getElementById("backButton");
    backButton.addEventListener("click", handleBackButtonClick);
    function handleBackButtonClick() {
      d3.select("#newpie svg").remove();
      backButton.style.display = "none";

      var svg = d3
        .select("#continentPie")
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

      // Mapping for Continent Names
      const continentMapping = {
        NA: "North America",
        AS: "Asia",
        EU: "Europe",
        AF: "Africa",
        SA: "South America",
        OC: "Oceania",
      };

      g.append("text")
        .attr("transform", function (d) {
          const centroid = label.centroid(d);
          const angle =
            ((d.startAngle + d.endAngle) / 2) * (180 / Math.PI) - 90;
          const rotate = angle > 90 && angle < 270 ? angle + 180 : angle;
          const newX = centroid[0] * 0.9;
          const newY = centroid[1] * 0.9;

          return "translate(" + [newX, newY] + ") rotate(" + rotate + ")";
        })
        .attr("dy", "0.35em")
        .style("text-anchor", "middle")
        .text(function (d) {
          return continentMapping[d.data.Continent];
        });
      g.on("click", (event, d) => handleClick(event, d));
    }

    function constructInitialPieChart() {}

    function handleMouseOver(event, d) {
      const cdata = `
    Country: ${d.data.country}<br>
    Total Consumption: ${d.data.total_consumption}<br>
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
      backButton.style.display = "1";
      d3.select("svg").remove();
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
          const angle =
            ((d.startAngle + d.endAngle) / 2) * (180 / Math.PI) - 90;
          const rotate = angle > 90 && angle < 270 ? angle + 180 : angle;
          const newX = centroid[0] * 0.8;
          const newY = centroid[1] * 0.8;

          return "translate(" + [newX, newY] + ") rotate(" + rotate + ")";
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
      backButton.style.display = "block";
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
  <h1>Energy consumption throughout the WORLD</h1>
  <h3>Using an interactive pie graph</h3>
  <h5>Consumption is measured in terawatt-hours.</h5>
  <h2>
    <button id="backButton" style="display: none; margin: auto;"
      >Back to Continent</button
    >
  </h2>
  <body>
    <div id="continentPie"></div>
    <div id="newpie"></div>
  </body>
  <h2>Writeup</h2>
  <p>
    The data we are utilizing for our visualization boils down to two things,
    location and energy consumption for the year 2022. Because our data is very
    specific and does not vary across time, we chose to use an arc mark or pie
    chart to display this information. The two other considerations for marks
    were the bar chart and the sunburst chart. The bar chart or more
    specifically, the stacked bar chart, would have achieved what we wanted to
    show, which was the differing total consumptions across location and type of
    energy through the length of each bar category and would have shown the
    total consumption value or percentage of the total value on the axis.
    However, due to the specific data we are using, a bar chart would be too
    complex. For example, if we chose to create a stacked bar chart, we would
    only have one bar since we are not comparing global total consumption
    against other global total consumptions, which would make the one bar
    visually unappealing and seem like the visualization was missing data. And
    if we were to split the stacked bar chart to where the different locations
    were side by side, we would lose the main focus of the visualization which
    is to show how each location contributes to the global consumption of
    energy. For these reasons, we didn’t choose the bar chart. The sunburst
    chart, however, was the original visualization chosen for this specific
    project. But, the main problem didn’t come from the visualization, but
    rather the data. For the sunburst chart, nested data or data with hierarchy
    is what makes the sunburst chart work, but our data is not nested. With
    enough time, we could have arranged our data in a way to satisfy this
    hierarchy structure to utilize the sunburst, but we couldn’t get to it. Thus
    we settled on a chart that matches the simplicity of our data. Besides
    marks, the simplicity of a pie chart means there are few visual encodings
    used. As shown, a piece of a pie is represented by location, whether that be
    the continent or country, and the width of that piece is represented by the
    total consumption of energy by that category. Color is used to indicate that
    the categories are different, but they do not indicate anything else beyond
    that. We also made sure to add a tooltip for the country pie; which is
    accessed through clicking on a continent which brings that continent's
    respective countries, and you're able to hover your mouse over the country
    that you want information for, and tooltip will appear giving all the
    different enery and the consumption.This tooltip allows you to easily read
    the data without having to wait for loading; and it instantly changes the
    data shown if you hover over a different country making it easy to compare
    between what you want. For the interaction technique, we chose to use
    zooming. We felt that zooming would best fit our visualization because we
    wanted to show our data from the large scale to the small scale and zooming
    helped achieve that as by zooming into each piece of the pie, we could show
    more information about our data. … Development for this project first
    started with filtering out our data to only the relevant columns necessary
    for our visualization. This was done to avoid loading in an extremely large
    dataset into our webpage along with keeping the visualization concise with
    our vision and not intentionally or accidentally adding in new or irrelevant
    variables into our visualization; along with the tooltip being a helpful
    tool. Along with that we also added a back button; so in the scenerio that a
    reader wants to look a different continent's information they can press the
    back button (which only shows up after you clicked a continent as it
    wouldn't make sense to have a button when you're already on the continent
    chart); making it so you don't have to reload the page to access new
    information. Both Bryan and David helped filter out the data, and this step
    took about thirty minutes. The next step was creating the necessary parts of
    our visualization in D3/Svetle, and this was the longest part of the
    development process. This took about 4 days, with a working time of about 4
    hours per day. This was due to both of us being new to D3/Svelte and trying
    to learn along the way how to implement certain attributes and features
    necessary to our visualization. Finally, the last part of development was
    the styling of the webpage, such as fonts and positioning. This took about
    two hours to do as most of it was trying to match the style of the webpage
    to our visualization.
  </p>
</main>

<!--Styling for Website. Fonts and other things-->
<style>
  @import url("https://fonts.googleapis.com/css2?family=Open+Sans&display=swap");

  h1 {
    font-family: "Open Sans", sans-serif;
    text-align: center;
    margin-top: 0px;
    margin-bottom: 10px;
    font-size: 4rem;
    background: linear-gradient(
      to right,
      #ef5350,
      #f48fb1,
      #7e57c2,
      #2196f3,
      #26c6da,
      #43a047,
      #f9a825,
      #ff5722
    );
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }
  h3 {
    font-family: "Open Sans", sans-serif;
    text-align: center;
    margin-top: 0px;
    margin-bottom: 10px;
    font-size: 30px;
    background: linear-gradient(
      to right,
      #f48fb1,
      #7e57c2,
      #2196f3,
      #26c6da,
      #43a047,
      #f9a825,
      #ff5722
    );
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }
  div {
    text-align: center;
  }
  body {
    font-family: "Open Sans", sans-serif;
    text-align: center;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0;
  }
  p {
    text-align: left;
  }
</style>
