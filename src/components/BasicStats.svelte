<script context="module">
  import * as d3 from "d3";
  import { onMount } from "svelte";

  export function create_scale2(scaleSvgEl, colorScale, filtered_data) {
    let f = filtered_data
      .filter((d) => d.voted_proportion > 0)
      .map((d) => d.voted_proportion);
    let stats = {
      min: d3.min(f),
      max: d3.max(f),
      avg: d3.mean(f),
    };
    //it shoudl be dynamic
    let svg_width = 300;
    let svg = d3.select(scaleSvgEl);
    svg.style("width", svg_width + "px");
    svg.style("height", "90px");
    svg.select("defs").remove();
    var defs = svg.append("defs");
    var gradient = defs
      .append("linearGradient")
      .attr("id", "svgGradient")
      .attr("x1", "0%")
      .attr("y1", "0%")
      .attr("x2", "100%")
      .attr("y2", "0%");

    gradient
      .selectAll("stop")
      .data(d3.range(0, 100))
      .enter()
      .append("stop")
      .attr("offset", function (d, i) {
        return i + "%";
      })
      .attr("stop-color", function (d, i) {
        if (i < 10) console.warn(i, colorScale(i));
        return colorScale(i);
      });
    var rect = svg
      .append("rect")
      .attr("class", "rect_legend")
      .attr("width", svg_width)
      .attr("height", 10)
      .attr("fill", "url(#svgGradient)");

    var x_pos = [];
    //var domain=[0, 25, 50, 75, 100];

    var domain = [
      stats.min,
      (stats.max - stats.min) / 4,
      (stats.max - stats.min) / 2,
      (stats.max - stats.min) / 1.5,
      stats.max,
    ];
    var range = domain.map((d, i) => {
      if (i < domain.length - 1) return (d * svg_width) / stats.max;
      else return (d * svg_width) / stats.max - 20;
    });

    let xValues = domain; //.map((d)=>String(d));
    console.warn(xValues);
    console.log(range);
    var xScale = d3
      .scaleLinear()

      //domain ==>the values
      .domain(domain)

      .range(range);

    var axisLeg = d3.axisBottom(xScale).tickValues(xValues);
    let legendHeight = 45;
    var legendsvg = svg
      .append("g")
      .attr("class", "legendWrapper")
      .style("width", svg_width)

      .style("height", legendHeight);

    legendsvg
      .append("g")
      .style("width", svg_width)

      .style("height", 45)
      .attr("class", "axis") //Assign "axis" class
      .attr("transform", "translate(0, 20)")
      .call(axisLeg)
      .selectAll("text")
      .style("text-anchor", "start")
      .attr("fill", "white")
      .attr("font-size", "0.7rem")
      //.attr("dx", "2em")
      // .attr("dy", ".29em")
      .attr("transform", "rotate(-65)");
  }
</script>

<script>
  export let filtered_data;
  export let colorScale;
  export let stats;

  let scaleSvgEl;

  // run 2 onwards (reactive updates)
  $: if (scaleSvgEl) create_scale2(scaleSvgEl, colorScale, filtered_data);

  onMount(() => {
    // first run only (when component is mounted)
    create_scale2(scaleSvgEl, colorScale, filtered_data);
  });
</script>

<section>
  <h3>Basic stats of voted_proportion</h3>
  <ul>
    <li>Min: {stats.min}</li>
    <li>Max: {stats.max}</li>
    <li>Avg: {stats.avg}</li>
  </ul>
  <div class="scale_container">
    <h2>Scale</h2>
    <svg bind:this={scaleSvgEl} />
  </div>
</section>

<style>
  path.domain,
  line {
    opacity: 0;
  }
  section {
    padding: 10px;
    margin: 10px;
    color: white;
  }
</style>
