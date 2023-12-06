<script>
// @ts-nocheck
    export let sprite = null;
    export let name = "UNDEFINED_NAME";
    export let stats = null;
    export let types = [];
    /**Button functions*/
    export let pushDraftedMon1;
    export let pushDraftedMon2;
    export let removeMon = undefined;
    export let drafted = false;
</script>

<div class="container" class:red={drafted}>
    <div id="name">{name}</div>
    <img src="{sprite}" alt="Sprite of {name}">
    <span class="stats">
        {#each stats as stat}
            <div>
                <p>{stat.stat.name}</p>
                <p>{stat.base_stat}</p>
            </div>  
        {/each}
    </span>
    <span class="types">
        {#each types as type}
            <div>
                <p>{type.type.name}</p>
            </div>
        {/each}
    </span>
    {#if removeMon !== undefined}
      <button on:click={() => pushDraftedMon1(name)} id="draft1">me</button>
      <button on:click={() => pushDraftedMon2(name)} id="draft2">them</button>
      <!-- <button on:click={() => removeMon(name)} id="remove">remove</button> -->
    {/if}
</div>

<style>
  * {
    font-size: 0.8rem;
  }

  #name {
    min-width: 7.5rem;
    overflow-wrap: break-word;
  }

   .container {
    display: flex;
    border: 1px solid black;
    justify-content: left;
    align-items: center;
    padding: 0.25rem;
  }

  .stats,
  .types {
    display: flex;
    gap: 1.5rem;
    border: 1px solid red; /* Adjust border color as needed */
  }

  p {
    font-size: 0.65rem;
  }

  button:hover {
    cursor: pointer;
  }

  img {
    width: 3rem;
    height: 3rem;
    border: 1px solid black;
  }

  p {
    max-width: 10rem;
  }

  .red {
    background-color: rgb(255, 165, 165);
  }

  .blue {
    background-color: rgb(140, 186, 255);
  }
</style>