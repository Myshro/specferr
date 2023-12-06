<script>
    // @ts-nocheck
    
        /**
         * - 
         * - input multiple moves 
         * - handle invalid input for all filters
        */
    
    import { stats } from "$lib/stats"
    import PokemonPreview from "$lib/components/PokemonPreview.svelte";
        import FilterBox from "$lib/components/FilterBox.svelte";
    // import { filtered } from "$lib/store"
    
    /**@type Object*/
    let res;
    /**@type number*/
    let totalPlayers = 0;
    /**@type string*/
    let url = "https://pokeapi.co/api/v2/pokemon/";
    /**@type string*/
    const BASE_URL = "https://pokeapi.co/api/v2/";
    
    let errorThrown = false;
    let invalidInputs = [];
    
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
    
    /**@type Object*/
    const filtered = {
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
    
    const filters = {
        ability: null,
        move: null,
        type: null,
        stat: null,
    }
    
    const createUrl = (endpoint, target) => {
        return `${BASE_URL}${endpoint}${target}`;
    }
    
    // ability, move, type use this func
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
            // console.log(`${p.name}: ${attribute}: ${p[attribute]}`)
            return p[attribute] >= min;
        });
        console.log(s)
        return s;
    }
    
    const printError = (error) => console.log(`FATAL ERROR:\n${error}`);
    
    const fmt = (json) => JSON.stringify(json, null ,4);
    
    const pushTypeFilter = (result) => {
        const tf = filtered.type;
        result.pokemon.map(p => {
            if (!isInFiltered(tf, p.pokemon.name)) {
                tf.push(p.pokemon.name);
            }
        })
    }
    
    const pushMoveFilter = (result) => {
        const mf = filtered.move;
        result.learned_by_pokemon.map(p => {
            if (!isInFiltered(mf, p.name)) {
                mf.push(p.name);
            }
        })
    } 
    
    const pushAbilityFilter = (result) => {
        const af = filtered.ability;
        result.pokemon.map(p => {
            if (!isInFiltered(af, p.pokemon.name)) {
                af.push(p.pokemon.name);
            }
        })
    }
    
    const pushStatFilter = (result) => {
        const sf = filtered.stat;
        result.map(p => {
            if (!isInFiltered(sf, p.name)) {
                sf.push(p.name);
            }
        })
    }
    
    const pushFilter = (result, filterType, propertyName) => {
        const filterList = filtered[filterType];
        result.map(item => {
            const itemName = item[propertyName];
            if (!isInFiltered(filterList, itemName)) {
                filterList.push(itemName);
            }
        });
    };
    
    // Example usage:
    
    
    
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
        // return filtered.ability.includes(pokemonName) &&
        //        filtered.move.includes(pokemonName) &&
        //        filtered.type.includes(pokemonName) //&&
        //     //    filtered.stat.includes(pokemonName);
    }
    
    const isEmpty = (arr) => arr.length === 0;
    
    const unionizePokemonNames = () => {
        if (noFiltersExist()) {
            errorThrown = true;
            invalidInputs = "Currently no filters are set";
            return;
        }
        const firstNonEmptyValues = Object.values(filtered).find(arr => !isEmpty(arr));
    
        filtered.union_name = firstNonEmptyValues.filter(isSharedInFiltered);
    }
    
    const noFiltersExist = () => {
        return Object.values(filtered).every(arr => isEmpty(arr));
    }
    
    // (async function() {
    //     let result;
    //     // result = await fetchDetail(ENDPOINT.TYPE, TYPE.DRAGON);
    //     // console.log(result);
    //     // pushTypeFilter(result);
    //     // console.log(filtered);
    
    //     result = await fetchDetail(ENDPOINT.MOVE, "tackle");
    //     pushMoveFilter(result);
    
    //     result = await fetchDetail(ENDPOINT.ABILITY, "sturdy");
    //     pushAbilityFilter(result);
    //     // result = fetchStat(STAT.SPECIAL_DEFENSE, 80);
    //     // console.log(result);
    //     // pushStatFilter(result);
    //     // console.log(filtered);
    
    //     unionizePokemonNames();
    //     console.log(filtered);
    // })();
    
    // this func will do all the fetching of filters
    const applyFilters = async () => {
        let result;
        filtered.type = [];
        filtered.move = [];
        filtered.ability = [];
        filtered.stat = [];
        if (filters.type !== null) {
            result = await fetchDetail(ENDPOINT.TYPE, filters.type);
            pushTypeFilter(result);
        }
    
        if (filters.move !== null) {
            result = await fetchDetail(ENDPOINT.MOVE, filters.move);
            pushMoveFilter(result);
    
        }
        
        if (filters.ability !== null) {
            try {
                result = await fetchDetail(ENDPOINT.ABILITY, filters.ability);
                pushAbilityFilter(result);
            } catch (error) {
                printError(error);
                errorThrown = true;
                invalidInputs.push(filters.ability);
            }
    
        }
    
        if (filters.stat !== null) {
            result = fetchStat(STAT.SPECIAL_DEFENSE, filters.stat);
            pushStatFilter(result);
        }
    
        unionizePokemonNames();
        // console.log(filtered);
    }
    
    const notNullOrEmpty = (filterType) => filterType !== null
    
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
    }
    
    const applyFiltersAndPopulateData = async () => {
        console.log(filters);
        await applyFilters()
        await populateUnionData();
    }
    
    
    </script>
    
    <header>
        <ul class="filters">
            {#if errorThrown}
                <h1>Invalid input: {invalidInputs}</h1>
            {/if}
            {#each Object.entries(filters) as [key, val]}
                <FilterBox lable={key} value={val} bind:appliedValue={filters[key]}/>
            {/each}
        </ul>
        <a href="./">saves</a>
    </header>
    
    <form on:submit|preventDefault={null}>
        <button type="submit" on:click|preventDefault={applyFiltersAndPopulateData}>Display</button>
    </form>
    
    {#if true}
        <!-- <pre>
            {fmt(filtered.union_data)}
        </pre> -->
        <ul>
            {#each filtered.union_data as { name, sprites, stats, types }}
                <PokemonPreview name={name} sprite={sprites.front_default} stats={stats} types={types}/>
            {/each}
            {#if filtered.union_data.length === 0}
                No pokemon found.
            {/if}
        </ul>
    {:else}
    {/if}
    
    <style>
        form {
            display: flex;
            flex-direction: column;
        }
    
        .filters {
            border: 1px solid black;
        }
    
    </style>