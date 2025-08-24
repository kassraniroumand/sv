<script lang="ts">
  let {name, full_name = null}: {name: string, full_name?: string | null} = $props();
</script>

<div>
    <h1>Header</h1>
    <p>Hello {name}</p>
    <p>{full_name}</p>
</div>
