<script>
  //Imports
  import { onMount } from "svelte";
  import * as d3 from "d3";

  // Load Data
  let tempData = [];

  onMount(async () => {
    const res = await fetch("data_energy.csv");

    const csv = await res.text();

    tempData = d3.csvParse(csv, d3.autoType);

    console.log(tempData);
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
