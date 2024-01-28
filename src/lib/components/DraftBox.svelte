<!-- This is a component that holds pokemon previews, it is the "me" and "them" boxes in the UI.  -->
<script>
// @ts-nocheck

import DraftBoxPokemonPreview from "./DraftBoxPokemonPreview.svelte";
import PokemonPreview from "./PokemonPreview.svelte";

    export let selectedMons = [];
    export let teamName;
    export let removeMon;
    export let fetchMon;

    let isHovered = false;
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="container">
    <h1>{teamName}</h1>
    {#each selectedMons as mon}
        <div>
          <DraftBoxPokemonPreview fetchMon={fetchMon} removeMon={removeMon} name={mon}/>
          <!-- <button on:click={() => removeMon(mon)}>X</button> -->
        </div>
    {/each}
</div>

<style>  
  h1 {
    font-size: 1rem
  }
    * {
        font-size: 0.6rem;
    }
    /* Add your styling for the grid here */
  .container {
    display: flex;
    flex-direction: column;
    border: 1px solid black;
    font-weight: normal;
    width: 100%;
    padding: 0.25rem 1rem 1rem 1rem;
  }

  p {
    border-bottom: 1px solid black;
    height: 100%;
    
  }
</style>