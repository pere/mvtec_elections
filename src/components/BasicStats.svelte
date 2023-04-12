

<script>
    
    
    
    import { onMount } from 'svelte';
  import App from '../App.svelte';
    export let filtered_data;
    export let selected_schema;
    

let f=filtered_data.filter(d=>d.voted_proportion>0).map(d=>d.voted_proportion);
let stats={
    min:d3.min(f),
    max:d3.max(f),
    avg:d3.mean(f)
};

console.log(stats)

                        
onMount(() => 
  {
    create_scale2(selected_schema,filtered_data);     
    console.log(stats)
    
  });
// }
</script>


<script context="module">
	
    import * as d3 from 'd3';
   
	export function create_scale2(selected_schema,filtered_data) {

        
        console.warn(filtered_data)
        let f=filtered_data.filter(d=>d.voted_proportion>0).map(d=>d.voted_proportion);
        let stats={
            min:d3.min(f),
            max:d3.max(f),
            avg:d3.mean(f)
        };
		 //it shoudl be dynamic
             let svg_width=300;
            let svg=d3.selectAll('.scale_container svg');
            svg.style('width', svg_width + 'px');
                    svg.style('height', '40px');
                    svg.select('defs').remove();
                    var defs = svg.append("defs");
                    var gradient = defs.append("linearGradient")
                        .attr("id", "svgGradient")
                        .attr("x1", "0%")
                        .attr("y1", "0%")
                        .attr("x2", "100%")
                        .attr("y2", "0%");
                        
                    var sel_colorScale_filter = d3.scaleSequential(d3[selected_schema])
                        .domain([0, 100]);   

                    gradient.selectAll("stop")
                                    .data(d3.range(0,100))
                                    .enter().append("stop")
                                    .attr("offset", function(d, i) {

                                        return i + '%';
                                    })
                                    .attr("stop-color", function(d, i) {
                                        if (i<10)
                                        console.warn(i,sel_colorScale_filter(i))
                                        return sel_colorScale_filter(i)
                                    });
                        var rect = svg.append("rect")
                        .attr('class', 'rect_legend')
                        .attr('width', svg_width)
                        .attr('height', 10)
                        .attr("fill", "url(#svgGradient)");    
                                    

                var x_pos = [];
                //var domain=[0, 25, 50, 75, 100];

                var domain=[stats.min, (stats.max-stats.min)/4, (stats.max-stats.min)/2, (stats.max-stats.min)/1.5, stats.max];
                var range=domain.map((d,i)=>
                {
                    if (i<domain.length-1)
                    return (d*svg_width)/stats.max
                    else
                    return ((d*svg_width)/stats.max)-20

                });
           
                let xValues=domain;//.map((d)=>String(d));
                console.warn(xValues)
                console.log(range)
                var xScale = d3.scaleLinear()
                    
                    //domain ==>the values
                    .domain(domain)
                    
                    .range(range);

                var axisLeg = d3.axisBottom(xScale)
                        .tickValues(xValues);
                let legendHeight=45;
                var legendsvg = svg.append("g")
                    .attr("class", "legendWrapper")
                    .style("width", svg_width)
                    
                    .style("height", legendHeight)


                legendsvg.append("g")
                    .style("width", svg_width)
                    
                    .style("height", 45)
                    .attr("class", "axis") //Assign "axis" class
                    .attr("transform", "translate(0, 20)")
                    .call(axisLeg)
                    .selectAll("text")
                    .style("text-anchor", "start")
                    .attr("fill", 'white')
                    .attr('font-size', '0.7rem')
                    //.attr("dx", "2em")
                    // .attr("dy", ".29em")
                    .attr("transform", "rotate(-65)")

                 /*    .attr("transform", function(d, i) {
                        if (i == 0) {
                            return "translate(0,0)"
                        } else {
                            if (i == domain.length - 1) {
                                return "translate(-8,0)"
                            } else {
                                return "translate(-8,0)"
                            }
                        }


                    }); */

            }  
	
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
    <svg>
    
    </svg>
    </div>
</section>
<style>
    line 
  {
    opacity: 0;
  }
    path.domain,line 
    {
        opacity: 0;
    }
    section{
        
        padding:10px;
        margin:10px;
        color:white;
    }
   
</style>