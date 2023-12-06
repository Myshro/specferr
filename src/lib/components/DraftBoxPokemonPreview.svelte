<script>
// @ts-nocheck

import PokemonPreview from "./PokemonPreview.svelte";

export let removeMon;
export let fetchMon;

export let name;
let data = fetchMon(name);
let isHovered = false;
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="container">
    <p
    on:mouseenter={() => {isHovered = true}} on:mouseleave={() => isHovered = false}>
        {name}
    </p>
    {#if isHovered}
    <div class="preview">
            <PokemonPreview 
            name={data.name} sprite={data.sprites.front_default} 
            stats={data.stats} types={data.types} disableName={true}
            />
    </div>
    {/if}
    <button on:click={() => removeMon(name)}>X</button>
</div>

<style>
    button {
        padding: 0px 10px;
        background-color: rgb(255, 165, 165);
    }

    .preview {
        /* display: inline-block; */
        /* margin-top: 2rem; */
        opacity: 0;
        transition: opacity 1s ease-out;
    }

    * {
        display: inline;
    }

    .container {
        /* border: 1px solid black; */
        display: flex;
        flex-direction: column;
    }
    .container:hover {
        cursor: pointer;
    }

    .container:hover .preview {
        opacity: 1;
        transform: translateY(0);
  }
</style>