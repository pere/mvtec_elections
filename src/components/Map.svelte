
<script context="module"></script>
<script>
  import { onMount, onDestroy } from "svelte";
  import { getContext, createEventDispatcher } from "svelte";
  import { Map, NavigationControl, Popup, Marker, LngLat } from "maplibre-gl";
  import "maplibre-gl/dist/maplibre-gl.css";
  import * as d3 from "d3";
  import jQuery from 'jquery';

  import { munis } from "../data/municipis.js";
  import { observable_data } from "../data/observable_data.js";

  export let filtered_data;
  export let colorScale;

  export let _f;
  let hideTooltip = false;
	
	//const dispatch = createEventDispatcher();

  let map;
  let mapContainer;
  let svg;
  let featureElements;

  let projectPoint = function (lon, lat) {
    var point = map.project(new LngLat(lon, lat));
    this.stream.point(point.x, point.y);
  };
  let transform = d3.geoTransform({
    point: projectPoint,
  });
  let path = d3.geoPath().projection(transform);

  let first_update = true;
  
  $: if (filtered_data.length > 0 && featureElements) {
    console.warn(filtered_data);
    let arr = filtered_data.map((d) => d.municipality_code);
    
    featureElements
      .transition()
      .duration(1050)
      .delay(function (d, i) {
        return i * 5;
      })

      .attr("fill", function (d, i) {
        let codiine = String(d.properties.codiine);
        let pos=arr.indexOf(codiine);
        if (pos > -1) {
          console.warn(filtered_data[pos].voted_proportion,colorScale(filtered_data[pos].voted_proportion))
          return colorScale(filtered_data[pos].voted_proportion);
        } else {
          return "black";
        }
      });
  }
  onMount(() => {
    const data = [];
    console.warn(filtered_data);
    map = new Map({
      container: "map", // container id
      //style: 'mapbox://styles/mapbox/streets-v8',
      style: {
        version: 8,
        sources: {
          geoserver: {
            type: "vector",
            tiles: [
              "https://geospatial.jrc.ec.europa.eu/geoserver/gwc/service/wmts?REQUEST=GetTile&SERVICE=WMTS&VERSION=1.0.0&LAYER=hotspots:gaul_0_simplified&STYLE=&TILEMATRIX=EPSG:900913:{z}&TILEMATRIXSET=EPSG:900913&FORMAT=application/vnd.mapbox-vector-tile&TILECOL={x}&TILEROW={y}",
            ],
          },
        },
        layers: [
          {
            id: "gaul_0_simple",
            source: "geoserver",
            "source-layer": "gaul_0_simplified",
            type: "fill",
            // "minzoom": 4,
            // "maxzoom": 6,
            paint: {
              "fill-color": "#a2a7ab",
              "fill-outline-color": "#87989c",
              "fill-opacity": 1,
            },
          },
        ],
      },
      center: [1.05, 41.4],

      //center: [141.15448379999998, 39.702053　],
      zoom: 7.37,

      attributionControl: false,
    });

    map.addControl(new NavigationControl(), "top-right");
    function update() {
      featureElements = svg.selectAll("path.municipi");
      //featureElements.raise();
      featureElements.attr("d", path);
      if (first_update) {
        first_update = false;
        let tooltip = d3.select("#map")
            .append("div").attr("class", "tooltip")
            .style("position", "absolute")
            .style("z-index", "100000000")
            .style("color", "white")
        /*  featureElements
                            .transition()
                            .duration(1050)
                            .delay(function(d, i) {
                                return i * 5;
                            })

                            .attr('fill', function(d, i) {
                              return '#daa520'
                            }) */
      }
    }
    
    map.on("load", function () {
      let fs = munis.features;

      const municipis_geojson = {
        type: "FeatureCollection",
        features: fs,
      };
      map.addSource("municipis_source", {
        type: "geojson",
        data: municipis_geojson,
      });
      console.log(observable_data);
      console.warn(municipis_geojson);
      //we just add but hidden, maybe we can add it later
      map.addLayer({
        id: "municipis",
        type: "fill",
        source: "municipis_source",
        layout: {
          // Make the layer visible by default.
          visibility: "visible",
        },
        paint: {
          "fill-color": "#088",
          "fill-opacity": 0,
        },
      });
      console.info(map.getStyle().layers);

      var container = map.getCanvasContainer();

      svg = d3.select(container).append("svg");
      svg
        .style("position", "absolute")
        .style("width", "100%")
        .style("height", "100%")
        .style("top", "0")
        .style("left", "0")
        .style("z-index", "1000");

      //             var marker = new Marker()
      // .setLngLat([1.5,41])
      // .addTo(map);

      featureElements = svg
        .selectAll("path.municipi")
        .data(fs)
        .enter()
        .append("path")
        // .classed('region', true)
        .attr("class", function (d) {
          return "municipi code_" + d.properties.municipi;
        })
        .attr("fill", "black")
        .attr("stroke", "gray")

        /*
                .attrs(
                  {
                      "stroke": "#ffff",
                      "fill": "black",
                      "opacity": 0.4,
                      'stroke-width': 0.4
                  }
                ) 
                */
        .attr("code", function (d) {
          return d.properties.municipi;
        })
        .on('mouseenter', function(d) {
          console.warn(d)
         // d3.select(this).attr('fill', 'red')
           _f=d;
        
        })

      update();
      //alert(d3.selectAll('path').size())
      //alert(jQuery('.municipi').length)
    });
  

    map.on("viewreset", function () {
      /* this_app.to_update=true;
                this_app.first_update = false; */
      //svg.classed("hidden", true);
      svg.style("opacity", 0);
      update();
    });
    map.on("movestart", function () {
      svg.classed("hidden", true);
      //svg.classed("hidden", true);
      svg.style("opacity", 0);
    });
    map.on("rotate", function () {
      svg.classed("hidden", true);
      svg.style("opacity", 0);
    });
    map.on("moveend", function () {
      /* this_app.to_update=true;
                this_app.first_update = false; */
      svg.classed("hidden", true);
      svg.style("opacity", 0);
      update();
      svg.style("opacity", 1);
      svg.classed("hidden", false);
    });

    // change pointer on mouse over states
    map.on("mouseenter", "associated-munis", function () {
      map.getCanvas().style.cursor = "pointer";
    });
    map.on("mousemove", "associated-munis", function () {
      map.getCanvas().style.cursor = "";
    });
    map.on("mouseleave", "associated-munis", function () {
      map.getCanvas().style.cursor = "";
    });

    let municipi_popup = new Popup({
                closeButton: false,
                closeOnClick: true,
//className: 'municipi_map_popup',
                //offset: [20, -20]
                offset: {
                    'bottom': [0, -20],
                    'top': [0, 15],
                    'top-left': [0, 0], //[linearOffset, (markerHeight - markerRadius - linearOffset) * -1],
                    'top-right': [0, 0], //[-linearOffset, (markerHeight - markerRadius - linearOffset) * -1],

                }
            });
            municipi_popup.addTo(map);
    /* 
            var popup = new maplibregl.Popup({ closeOnClick: false })
.setLngLat([-96, 37.8])
.setHTML('<h1>Hello World!</h1>')
.addTo(map); */

    map.on("mouseover", function(e) {
console.warn(_f)
      if (!filtered_data || !_f) return false;
        var latlng = e.lngLat;
        var x = e.originalEvent.clientX
        var y = e.originalEvent.clientY


        console.warn(x, y)

        municipi_popup.setLngLat(latlng)
        
        let tooltip=d3.select("#map .tooltip")
        console.log(_f.target.__data__.properties)
         tooltip
                .html('<div class="rect_popup"></div>')
                /* .style("left", (x + 30) + "px")
                .style("top", (y - 180) + "px") */
                
                console.info(_f.target.__data__.properties.codiine)
                let binded_data=filtered_data.filter((d)=>
                {
                  
                 return String(d.municipality_code)==String(_f.target.__data__.properties.codiine)
                })[0]
                //[pos].voted_proportion
                if (binded_data)
                {
                  
                  municipi_popup.setHTML('<h3>'+_f.target.__data__.properties.nom_muni+'</h3><div>'+binded_data.voted_proportion+'</div>')
                  
                 // jQuery('.tooltip .rect_popup').empty().append('<h3>'+_f.target.__data__.properties.nom_muni+'</h3><div>'+binded_data.voted_proportion+'</div>').show()
                  //tooltip.style("opacity", 1); 
                }
   


    })


  });

  onDestroy(() => {
    map.remove();
  });
</script>

<div class="map-wrap">
  <div class="map" id="map" bind:this={mapContainer} />
</div>

<style>
  .maplibregl-popup 
  {
    z-index: 111111111;
  }
  svg,
  .maplibregl-canvas-container svg {
    position: absolute;
    width: 100%;
    height: 100%;
  }
  .map-wrap {
    position: relative;
    float: right;
    width: 70vw;
    height: calc(
      100vh - 77px
    ); /* calculate height of the screen minus the heading */
  }

  .map,
  .maplibregl-canvas {
    position: absolute !important;
    width: 100% !important;
    height: 100% !important;
  }

  .watermark {
    position: absolute;
    left: 10px;
    bottom: 10px;
    z-index: 999;
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


</style>