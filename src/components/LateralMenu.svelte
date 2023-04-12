
<script>
  import { utcYears } from "d3";

export let years;
export let selectedYear;
export let main_parties;
export let selectedParty;
</script>

    
    {#if years && years.length>0}
    <div>
    <select class="menu" 
					name="menu" 
					id="menu" 
                    autocomplete="off"
					bind:value={selectedYear}>
                    <option disabled selected value="">Select a year</option>
                        {#each years as year}
                                <option value={year}>{year}</option>
                            {/each}
                        </select>
                        {#if selectedYear}
                            <p>Selected year: {selectedYear}</p>
                        {/if}
    </div>     
<br>                   
    {/if} 
     {#if main_parties && main_parties.length>0}
    <div>
        <select class="menu" 
        name="Partymenu" 
        id="Partymenu" 
        autocomplete="off"
        bind:value={selectedParty}>
        <option disabled selected value="">Select a party</option>
            {#each main_parties as party}
            <option value={party}>{party}</option>
            {/each}
        </select>

        {#if selectedParty}
            <p>Selected main party: {selectedParty}</p>
        {/if}
    </div>
    {/if} 



<style>
div
{
    display:'inline-grid';
    padding: 5px;
    /* width: 100%; */
    
    color: white;
}
</style>