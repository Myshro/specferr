<script>
// @ts-nocheck

    export let lable;
    /**@type string*/
    export let value;
    export let placeholder = "...empty";
    export let appliedValue = "";

    const handleSubmit = () => {
        appliedValue = value.toLowerCase();
        if (appliedValue === "") {
            appliedValue = null;
        }
        console.log(`APPVAL: ${appliedValue}`)
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
    <button type="submit" on:click|preventDefault={handleSubmit}>Apply</button>
</form>

<style>
    .container {
        display: flex;
        border: 1px solid black;
    }
</style>

