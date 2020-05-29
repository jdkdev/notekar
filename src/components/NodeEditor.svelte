<script>
  import YJS from "js-yaml";
  import { ajx } from "$frontier";
  import { createEventDispatcher } from "svelte";
  import { onMount, afterUpdate } from "svelte";
  const dispatch = createEventDispatcher();
  const MD = new markdownit();

  export let node;
  $: yata = YJS.safeDump(node.meta);

  // afterUpdate(() => {
  //   console.log(node.meta);
  // });
  function update() {
    node.matter.html = MD.render(node.matter.md);
    dispatch("node-changed", { node });
  }
  function updateMd() {
    node.editing = false;
    node.matter.html = MD.render(node.matter.md);
    dispatch("node-changed", { node });
  }
</script>

<div class="node-content-editor">
  <textarea
    class="v-- w-100 h--"
    name=""
    id=""
    value={yata}
    on:input={update} />
  {#if node.editing}
    <!--TODO bring in ACEEditor-->
    <textarea
      on:blur={updateMd}
      name=""
      id=""
      cols="44"
      rows="10"
      bind:value={node.matter.md} />
  {:else}
    <div
      style="min-height:220px;"
      class="mh++"
      tabindex="0"
      on:focus={() => (node.editing = true)}>
      {@html node.matter.html}
    </div>
  {/if}
</div>
