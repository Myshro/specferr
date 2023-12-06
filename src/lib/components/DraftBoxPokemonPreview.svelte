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
            stats={data.stats} types={data.types} 
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
        display: inline-block;
        /* margin-top: 2rem; */
    }

    * {
        display: inline;
    }

    .container {
        /* border: 1px solid black; */
        /* display: flex; */
    }
    .container:hover {
        cursor: pointer;
    }
</style>