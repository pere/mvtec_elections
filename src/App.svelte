<script>
  import Navbar from "./components/Navbar.svelte";
  import Map from "./components/Map.svelte";
  import LateralMenu from "./components/LateralMenu.svelte";
  import * as d3 from "d3";
  import BasicStats from "./components/BasicStats.svelte";
  import ScaleForm from "./components/ScaleForm.svelte";
  import { create_scale2 } from "./components/BasicStats.svelte";
  import { observable_data } from "./data/observable_data.js";

  import { onMount } from "svelte";

  let years = [];
  let main_parties = [];
  $: selected_schema = "interpolateWarm";
  let selectedYear = ""; //// menu built from data
  let selectedParty = ""; //// menu built from data

  const createColorScale = (selected_schema, min, max) => {
    return d3.scaleSequential(d3[selected_schema]).domain([min, max]);
  };

  const getYears = () => {
    /*    console.info(
      observable_data.map(d=>d.year)
    ) */
    years = [...new Set(observable_data.map((d) => d.year))];
    years = years.sort((a, b) => a - b);
    //return years;
  };

  const getMainParties = () => {
    /*    console.info(
      observable_data.map(d=>d.year)
    ) */
    main_parties = [...new Set(observable_data.map((d) => d.main_party))];
    main_parties = main_parties.sort((a, b) => a - b);
  };

  $: filtered_data = observable_data.filter(
    (d) => d.year == selectedYear && d.main_party == selectedParty
  );

  $: f = filtered_data
    .filter((d) => d.voted_proportion > 0)
    .map((d) => d.voted_proportion);

  $: stats = {
    min: d3.min(f),
    max: d3.max(f),
    avg: d3.mean(f),
  };
  $: colorScale = createColorScale(selected_schema, stats.min, stats.max);

  $: console.log(filtered_data);

  onMount(() => {
    getYears();
    getMainParties();

    console.log(main_parties);
  });
</script>

<svelte:head>
  <!-- <script src="./js/topojson.v1.min.js"></script> -->
</svelte:head>
<div class="app">
  <!-- <Navbar/> -->
  <section class="lateralMenu">
    <LateralMenu bind:selectedYear {years} />
    {#if selectedYear}
      <LateralMenu bind:selectedParty {main_parties} />
    {/if}

    {#if selectedParty && selectedYear && selected_schema}
      <BasicStats
        {filtered_data}
        sel_schema="selected_schema"
        {colorScale}
        {stats}
      />
      {#if filtered_data.length > 0}
        <ScaleForm
          bind:selected_schema
          sel_schema="selected_schema"
          on:change={create_scale2(selected_schema, filtered_data)}
        />
      {/if}
    {/if}
  </section>
  <!-- on:mousemove={get_data()} -->
  <Map {filtered_data} {colorScale}  />
</div>

<style>
    .maplibregl-popup 
  {
    z-index: 111111111!important;
  }
   .tooltip{
    z-index: 333333300!important;
    font-size: 1rem;
    background-color: #3f4142;
    border: 1px solid #d7caca;
    padding: 0px;
    opacity: 1;
    width: auto;
    height: auto;
    z-index: 300;
    position: absolute;
    background-color: #dcdfdf;
    color: white;
}
  path.domain,
  line {
    opacity: 0;
  }
  .lateralMenu {
    width: 30vw;

    height: calc(100vh - 77px);
    background-color: black;
  }

  .app {
    display: flex;
    flex-direction: row;
    height: 100vh;
  }

  svg.hidden {
    opacity: 0;
    visibility: hidden;
    display: none;
  }
  .app > div {
    vertical-align: top;
  }
</style>