<script>
// @ts-nocheck

import { stats } from "$lib/stats"
// import { filtered } from "$lib/store"

/**@type string*/
let value;
/**@type string*/
let submittedValue;
/**@type Object*/
let res;
/**@type number*/
let totalPlayers = 0;
/**@type string*/
let url = "https://pokeapi.co/api/v2/pokemon/";
/**@type string*/
const baseUrl = "https://pokeapi.co/api/v2/";

/**@type {Object} ability, move, type endpoints*/
const ENDPOINT = {
    ABILITY: "ability/",
    MOVE: "move/",
    TYPE: "type/"
}

const TYPE = {
  NORMAL: "normal",
  GRASS: "grass",
  ELECTRIC: "electric",
  WATER: "water",
  FIRE: "fire",
  BUG: "bug",
  GHOST: "ghost",
  PSYCHIC: "psychic",
  STEEL: "steel",
  DARK: "dark",
  FLYING: "flying",
  ICE: "ice",
  POISON: "poison",
  GROUND: "ground",
  ROCK: "rock",
  DRAGON: "dragon",
  FIGHTING: "fighting",
  FAIRY: "fairy",
};

/**@type Object*/
const filtered = {
    /**@type Array*/
    abilityFiltered: [],
    /**@type Array*/
    moveFiltered: [],
    /**@type Array*/
    typeFiltered: [],
    /**@type Array*/
    statFiltered: [],
    /**@type Array*/
    union: [],
};

let sprite;

const createUrl = (endpoint, target) => {
    return `${baseUrl}${endpoint}${target}`;
}

const fetchDetail = async (endpoint, target) => {
    const link = createUrl(endpoint, target);
    return (await fetch(link, {mode: 'cors'})).json();
}

const printError = (error) => console.log(`FATAL ERROR:\n${error}`);

const fmt = (json) => JSON.stringify(json, null ,4);

const pushTypeFilter = (result) => {
    result.pokemon.map(p => {
        if (!filtered.typeFiltered.includes(p.pokemon.name)) {
            filtered.typeFiltered.push(p.pokemon.name);
        }
    })
}

const pushMoveFilter = (result) => {
    result.learned_by_pokemon.map(p => {
        if (!filtered.moveFiltered.includes(p.name)) {
            filtered.moveFiltered.push(p.name);
        }
    })
} 

const pushAbilityFilter = (result) => {
    result.pokemon.map(p => {
        if (!filtered.abilityFiltered.includes(p.pokemon.name)) {
            filtered.abilityFiltered.push(p.pokemon.name);
        }
    })
}

const isSharedInFiltered = (pokemonName) => {
    console.log(`${pokemonName}`)
    return filtered.abilityFiltered.includes(pokemonName) &&
           filtered.moveFiltered.includes(pokemonName) &&
           filtered.typeFiltered.includes(pokemonName) //&&
        //    filtered.statFiltered.includes(pokemonName);
}

const unionize = () => {
    filtered.union = filtered.abilityFiltered.filter(isSharedInFiltered);
}


(async function() {
    let result = await fetchDetail(ENDPOINT.TYPE, TYPE.DRAGON);
    console.log(result);
    pushTypeFilter(result);
    console.log(filtered);

    result = await fetchDetail(ENDPOINT.MOVE, "absorb");
    console.log(result);
    pushMoveFilter(result);
    console.log(filtered);

    result = await fetchDetail(ENDPOINT.ABILITY, "hydration");
    console.log(result);
    pushAbilityFilter(result);
    console.log(filtered);

    unionize();
    console.log(filtered);

})();
//-----

const fetchMon = async (str) => {
    return (await fetch(`${url}${str}`, {mode: 'cors'})).json();
}

const handleSubmit = async () => {
    try {
        res = await (fetchMon(submittedValue));

    } 
    catch(e) {
        res = {};
        printError(e);
    }
    sprite = res.sprites.front_default;
}


</script>

<h1>{submittedValue !== undefined ? submittedValue : "...input something"}</h1>
<form on:submit|preventDefault={null}>
    <label>
        Name:
        <input
            placeholder="...pikachu"
            on:change={() => submittedValue = value.toLowerCase()}
            bind:value
        >
    </label>    
    <button type="submit" on:click|preventDefault={handleSubmit}>input</button>
</form>

{#if submittedValue !== ""}
    <img src={sprite} alt="pic of pokemon">
    <pre>{JSON.stringify(res, null, 4)}</pre>
{:else}
    <p>invalid input {submittedValue}</p>
{/if}

<style>
    form {
        display: flex;
        flex-direction: column;
    }
</style>