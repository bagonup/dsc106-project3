<script>
//Imports
import { onMount } from 'svelte';
import * as d3 from 'd3';

// Load Data
let tempData = [];

onMount(async () => {

    const res = await fetch('owid-energy-data.csv'); 

    const csv = await res.text();

    tempData = d3.csvParse(csv, d3.autoType)

    console.log(tempData);

});
</script>
