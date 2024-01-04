<script>
// @ts-nocheck

    export let lable;
    /**@type string*/
    let value;
    export let appliedValue = "";
    export let placeholder = "";
    switch (lable) {
        case "abil": {
            placeholder = "intimidate";
            break;
        }
        case "move": {
            placeholder = "toxic-spikes";
            break;
        }
        case "type": {
            placeholder = "fire";
            break;
        }
        case "stat": {
            placeholder = "70";
            break;
        }
        case "name": {
            placeholder = "pikachu";
            break;
        }
    }
    $: {
        appliedValue = processString(value);
        console.log(appliedValue)
    }

    const processString = (inputString) => {
        if (inputString === null || typeof inputString !== 'string') return null;
        if (inputString === '') return null;
        const stringWithoutSpaces = inputString.replace(/\s/g, '');
        const resultArray = stringWithoutSpaces.split(',').filter(item => item.trim() !== '');
        return resultArray;
    }

    const handleSubmit = () => {
        appliedValue = value.toLowerCase();
        if (appliedValue === "") {
            appliedValue = null;
        }
    }
</script>

<form on:submit|preventDefault={handleSubmit}>
    <label>
        {lable}:
        <input
            placeholder={placeholder}
            bind:value
        >
    </label>    
</form>

<style>
    .container {
        display: flex;
        border: 1px solid black;
        width: 100%;
    }

    label {
        font-weight: normal;
    }
</style>

