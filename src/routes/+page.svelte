<script lang="ts">
    import Header from './Header.svelte';
    let name = $state("kasra")
  let status : 'OPEN' | 'CLOSED' = $state('OPEN')
  let derivedRes = $derived(name + " " + "niroumand")
  function onclick() {
    status = status === "OPEN" ? "CLOSED" : "OPEN"

  }
</script>

<Header
name={name}
full_name={derivedRes}
/>
<input type="text" bind:value={name} />
<!-- <button {onclick}>{status}</button> -->
<button onclick={() => {
  status = status === "OPEN" ? "CLOSED" : "OPEN"
}}>get status</button>

<p>{derivedRes}</p>
