<script>
// @ts-nocheck
    export let sprite;
    export let name = "UNDEFINED_NAME";
    export let stats;
    export let types = [];
</script>

<div class="container">
    <p>{name}</p>
    <img src="{sprite}" alt="Sprite of {name}">
    <div class="stats">
        {#each stats as stat}
            <div>
                <p>{stat.stat.name}</p>
                <p>{stat.base_stat}</p>
            </div>  
        {/each}
    </div>
    <div class="types">
        {#each types as type}
            <div>
                <p>{type.type.name}</p>
            </div>
        {/each}
    </div>
    <button id="draft">draft</button>
    <button id="remove">remove</button>
</div>

<style>
    .container {
        display: flex;
        border: 1px solid black;
        justify-content: left;
        align-items: center;
    }

    .stats {
        display: flex;
        gap: 2rem;
        border: 1px solid red;
    }

    .types {
        display: flex;
        gap: 2rem;
        border: 1px solid blue;
    }

    button:hover {
        cursor: pointer;
    }

    img {
        width: 4rem;
        height: 4rem;
    }
</style>