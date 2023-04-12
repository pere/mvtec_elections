<script>
    export let selected_schema;
    
    
    let schemas_arr = ['interpolateGreens','interpolateRainbow', 'interpolateSinebow', 'interpolateRdYlBu', 'interpolateInferno', 'interpolateRdPu', 'interpolateSpectral', 'interpolateViridis', 'interpolateReds', 'interpolateYlOrRd', 'interpolatePlasma', 'interpolateWarm', 'interpolateRdYlGn', 'interpolatePiYG'];
</script>
<section class="menu-cont">
	<select class="menu" 
					name="menuSchema" 
					id="menuSchema" 
					bind:value={selected_schema}
                    on:change
                    >
		<option disabled selected value="">Select a color schema</option>
		
		{#each schemas_arr as schema}
			<option value={schema}>{schema}</option>
		{/each}
	</select>
</section>