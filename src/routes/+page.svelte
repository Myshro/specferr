<script>
// @ts-nocheck

import { stats } from "$lib/stats"
import PokemonPreview from "$lib/components/PokemonPreview.svelte";
import FilterBox from "$lib/components/FilterBox.svelte";
import DraftBox from "$lib/components/DraftBox.svelte";

/**@type Object*/
let res;
/**@type number*/
let totalPlayers = 0;
/**@type string*/
let url = "https://pokeapi.co/api/v2/pokemon/";
/**@type string*/
const BASE_URL = "https://pokeapi.co/api/v2/";
const TEAM_SIZE = 100000;

let errorThrown = false;
let invalidInputs = [];

let teamOneMonNames = [];
let teamTwoMonNames = [];

const pushNameToTeam1 = (name) => {
    if (teamOneMonNames.length >= TEAM_SIZE || teamOneMonNames.includes(name)) return;
    teamOneMonNames.push(name);
    teamOneMonNames = teamOneMonNames;
}

const pushNameToTeam2 = (name) => {
    if (teamTwoMonNames.length >= TEAM_SIZE || teamTwoMonNames.includes(name)) return;
    teamTwoMonNames.push(name);
    teamTwoMonNames = teamTwoMonNames;
}

const removeNameFromTeams = (name) => {
    const index1 = teamOneMonNames.indexOf(name);
    if (index1 !== -1) {
        teamOneMonNames.splice(index1, 1);
    }
    const index2 = teamTwoMonNames.indexOf(name);
    if (index2 !== -1) {
        teamTwoMonNames.splice(index1, 1);
    }
    teamOneMonNames = teamOneMonNames;
    teamTwoMonNames = teamTwoMonNames;
}


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

const STAT = {
    HP :"hp",
    ATTACK :"attack",
    DEFENSE :"defense",
    SPECIAL_ATTACK :"special-attack",
    SPECIAL_DEFENSE :"special-defense",
    SPEED :"speed"
}

const STAT_DATA = stats;
let currentStat; // current stat to filter by
let sortedStat = null; // stat to sort by

/**@type Object
 * Contains actual pokemon 
*/
let filtered = {
    /**@type Array*/
    ability: [],
    /**@type Array*/
    move: [],
    /**@type Array*/
    type: [],
    /**@type Array*/
    stat: [],
    /**@type Array*/
    union_name: [],
    /**@type Array*/
    union_data: [],
};

/**
 * Contains the filters that will be used to populate filtered
 */
const filters = {
    ability: null,
    move: null,
    type: null,
    stat: null,
    name: null,
}

// $: sortedStat {
//     if (filtered.union_data.length === 0) return;
//     filtered.union_data.sort((a, b) => {
//         return b.stats.find(s => s.stat.name === sortedStat).base_stat - a.stats.find(s => s.stat.name === sortedStat).base_stat;
//     });
//     filtered.union_data = filtered.union_data;
// }

const createUrl = (endpoint, target) => {
    return `${BASE_URL}${endpoint}${target}`;
}

// ability, move, type use this func to fetch data from the api
const fetchDetail = async (endpoint, target) => {
    filters[endpoint.toLowerCase().substring(0, endpoint.length - 1)] = target;
    const link = createUrl(endpoint, target);
    try {
        const r = (await fetch(link, {mode: 'cors'})).json();  
        errorThrown = false;
        return r; 
    } catch (error) {
        printError(error);
        errorThrown = true;
    }
}

// stat filter is quirky, gets its own func
const fetchStat = (attribute, min) => {
    const s = STAT_DATA.filter(p => {
        return p[attribute] >= min;
    });
    return s;
}

const printError = (error) => console.log(`FATAL ERROR:\n${error}`);



// used to make sure dont push same pokemon into the same filtered.X
const isInFiltered = (specification, name) => {
    return specification.includes(name);
}

const isSharedInFiltered = (pokemonName) => {
    return (
        (isEmpty(filtered.ability) || filtered.ability.includes(pokemonName)) &&
        (isEmpty(filtered.move)    || filtered.move.includes(pokemonName)) &&
        (isEmpty(filtered.type)    || filtered.type.includes(pokemonName)) &&
        (isEmpty(filtered.stat)    || filtered.stat.includes(pokemonName))
    );
}

const isEmpty = (arr) => arr.length === 0;

const unionizePokemonNames = () => {
    if (isEmptyFiltered()) {
        errorThrown = true;
        invalidInputs = "Currently no filters are set";
        return;
    }
    const firstNonEmptyFiltered = Object.values(filtered).find(arr => !isEmpty(arr));

    filtered.union_name = firstNonEmptyFiltered.filter(isSharedInFiltered);
    console.log(filtered.union_name);
}

const isEmptyFiltered = () => {
    return Object.values(filtered).every(arr => isEmpty(arr));
}

const clearFiltered = () => {
    filtered.type = [];
    filtered.move = [];
    filtered.ability = [];
    filtered.stat = [];
    filtered.union_name = [];
    filtered.union_data = [];
}

const pushStatFilter = (result) => {
    const sf = filtered.stat;
    result.map(p => {
        if (!isInFiltered(sf, p.name)) {
            sf.push(p.name);
        }
    })
}

Object.byString = function(o, s) {
    s = s.replace(/\[(\w+)\]/g, '.$1'); // convert indexes to properties
    s = s.replace(/^\./, '');           // strip a leading dot
    var a = s.split('.');
    for (var i = 0, n = a.length; i < n; ++i) {
        var k = a[i];
        if (k in o) {
            o = o[k];
        } else {
            return;
        }
    }
    return o;
}

const pushFilter = (result, filteredTarget, propertyName, pathToName) => {
    let tempList = [];
    result[propertyName].map(mon => {
        tempList.push(Object.byString(mon, pathToName));
    });
    if (filteredTarget.length === 0) {
        filteredTarget.splice(0, filteredTarget.length, ...tempList); // modify the original array
        console.log(filteredTarget)
        return;
    }
    const temp = tempList.filter(e => filteredTarget.includes(e));
    filteredTarget.splice(0, filteredTarget.length, ...temp);
    console.log(filteredTarget);
};


// this func will do all the fetching of filters
const applyFilters = async () => {
    let result;
    clearFiltered();

    if (filters.name !== null) {
        const data = await fetchMon(filters.name);
            if (data === undefined) {
                errorThrown = true;
                invalidInputs.push(filters.name);
                return;
            }
        filtered.union_name.push(data.name);
        filtered.union_data.push(data);
        return;
    }

    const catchError = (e, filterPath) => {
        printError(e);
        errorThrown = true;
        invalidInputs.push(filters[filterPath]);
    }

    const tryFilterFetch = async (filterType, endpoint, filteredTarget, propertyName, pathToName) => {
        if (filterType !== null) {
            try {
                for (let i = 0; i < filterType.length; i++) {
                    const criteria = filterType[i];
                    result = await fetchDetail(endpoint, criteria);
                    pushFilter(result, filteredTarget, propertyName, pathToName);
                }

            } catch (error) {
                catchError(error, "type")
            }
        }
    }

    await tryFilterFetch(filters.ability, ENDPOINT.ABILITY, filtered.ability, "pokemon", "pokemon.name");
    await tryFilterFetch(filters.type, ENDPOINT.TYPE, filtered.type, "pokemon", "pokemon.name");
    await tryFilterFetch(filters.move, ENDPOINT.MOVE, filtered.move, "learned_by_pokemon", "name");

    // if (filters.move !== null) {
    //     try {
    //         result = await fetchDetail(ENDPOINT.MOVE, filters.move);
    //         pushMoveFilter(result);
    //     } catch (error) {
    //         catchError(error, "move");
    //     }

    // }

    if (filters.stat !== null) {
        try {
            result = fetchStat(currentStat, filters.stat);
            pushStatFilter(result);
        } catch (error) {
            catchError(error, "stat")
        }
    }

    unionizePokemonNames();
}



//-----

const fetchMon = async (str) => {
    return (await fetch(`${url}${str}`, {mode: 'cors'})).json();
}
// this func actually assigns union_data so that svelte will render it
const populateUnionData = async () => {
    filtered.union_data = [];
    for (let i = 0; i < filtered.union_name.length; i++) {
        const data = await fetchMon(filtered.union_name[i]);
        if (!isInFiltered(filtered.union_data, data)) {
            filtered.union_data.push(data);
        }
    }
    filtered.union_data = filtered.union_data; // require this for reactivity :(
        console.log(filtered.union_data)
}

const applyFiltersAndPopulateData = async () => {
    await applyFilters()
    await populateUnionData();
    console.log(filters)
    console.log(filtered)
}

const clearFilters = () => {
    filters.type = null;
    filters.move = null;
    filters.ability = null;
    filters.stat = null;
    console.log(filters)
}

const fetchMonFromUnionData = (monName) => {
    for (let i = 0; i < filtered.union_data.length; i++) {
        const data = filtered.union_data[i];
        if (data.name === monName) return data;
    }
    return null;
}


</script>

<div id="top">
    <h1 id="title">Spec-Ferr Drafter</h1>
    <a href="https://github.com/Myshro/specferr" target="_blank">source</a>
    <a href="https://pokeapi.co/" target="_blank">api</a>
</div>
<header>
    <ul class="filters">
        {#if errorThrown}
            <h4>Invalid input</h4>
        {/if}
        <!-- <FilterBox lable={"name"} value={filters.name} bind:appliedValue={filters.name}/> -->
        {#each Object.entries(filters) as [key, val]}
            <!-- <FilterBox lable={key.substring(0, 4)} value={val} bind:appliedValue={filters[key]}/> -->
            <FilterBox lable={key.substring(0, 4)} bind:appliedValue={filters[key]}/>
            <!-- <FilterBox lable={key.substring(0, 4)} value={val}/> -->
        {/each}
        <label for="flavor">stat:</label>
        <select id="flavor" name="flavor" bind:value={currentStat} on:change={(event) => currentStat = event.target.value}>
            {#each Object.entries(STAT) as [key, val]}
                <option value={val}>{val}</option>
            {/each}
            <!-- Add more options as needed -->
        </select>
        <label for="sort-by">sort:</label>
        <select id="flavor" name="sort-by" bind:value={sortedStat} on:change={(event) => {
            sortedStat = event.target.value;
            if (filtered.union_data.length === 0) return;
            filtered.union_data.sort((a, b) => {
                return b.stats.find(s => s.stat.name === sortedStat).base_stat - a.stats.find(s => s.stat.name === sortedStat).base_stat;
            });
            filtered.union_data = filtered.union_data;
        }}>
            {#each Object.entries(STAT) as [key, val]}
                <option value={val}>{val}</option>
            {/each}
            <!-- Add more options as needed -->
        </select>
    <!-- <button id="clear" on:click|preventDefault={() => {clearFilters(); filtered.union_data=[]}}>Clear</button> -->
    </ul>
    <DraftBox fetchMon={fetchMonFromUnionData} removeMon={removeNameFromTeams} teamName={"me"} selectedMons={teamOneMonNames}/>
    <DraftBox fetchMon={fetchMonFromUnionData} removeMon={removeNameFromTeams} teamName={"them"} selectedMons={teamTwoMonNames}/>
    
</header>

<form on:submit|preventDefault={null}>
    <button type="submit" on:click|preventDefault={applyFiltersAndPopulateData}>Display</button>
</form>

    <ul>
        {#each filtered.union_data as { name, sprites, stats, types }}
            <PokemonPreview 
            name={name} sprite={sprites.front_default} 
            stats={stats} types={types} 
            drafted={teamOneMonNames.includes(name)}
            draftedByEnemy={teamTwoMonNames.includes(name)}
            pushDraftedMon1={pushNameToTeam1} 
            pushDraftedMon2={pushNameToTeam2}
            removeMon={removeNameFromTeams}/>
        {/each}
        {#if filtered.union_data.length === 0}
            No pokemon found / Loading.
        {/if}
    </ul>

<style>

    #top {
        display: flex;
        justify-content: left;
        align-items: center;
        gap: 3rem;
    }

    a {
        text-decoration: none;
        font-weight: bold;
    }

    a:hover {
        background-color: black;
        color: white;
    }

    #title {
        font-size: 3rem;
    }


    form {
        display: flex;
        flex-direction: column;
    }

    .filters {
        border: 1px solid black;
        width: 100%;
        padding: 1rem 1rem 1rem 1rem;
    }

    header {
        display: flex;
        justify-content: space-between;
        
    }
    

</style>