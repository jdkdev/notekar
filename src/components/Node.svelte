<script>
  import YJS from "js-yaml";
  import { ajx } from "$frontier";
  import { createEventDispatcher } from "svelte";
  import { onMount, afterUpdate } from "svelte";
  const dispatch = createEventDispatcher();
  const mdIt = new markdownit();

  export let id;
  export let grandchild = false;
  export let node = { name: "", matter: { md: "", html: "" } };
  export let current = false;
  export let show = {
    descendants: false
  };

  // afterUpdate(() => {
  //   console.log("test");
  //   dispatch("node-updated", { node });
  // });
  //This will come from the store
  let brain = {
    nodes: [{ meta: { name: "node 1" } }, { meta: { name: "node 2" } }]
  };

  $: filteredNodes = brain.nodes.filter(n => {
    return n;
    // return n.meta.name.includes(node.meta.name);
  });

  function ts(kind = "ts") {
    let timestamps = {};
    let ts = new Date().toJSON().replace(/[-T:.Z]/g, "");
    timestamps.d = Number(ts.substr(2, 6));
    timestamps.t = Number(ts.substr(8, 6));
    timestamps.ts = Number(ts);
    return timestamps[kind];
  }
  function* idGenerator() {
    let i = 50000;
    while (true) {
      yield "node-" + i++;
    }
  }
  let idGen = idGenerator();

  function allowDrop(ev) {
    ev.preventDefault();
  }

  function drag(ev) {
    console.log("drag start");
    ev.dataTransfer.setData("text", ev.target.id);
  }

  function drop(ev) {
    ev.preventDefault();
    var data = ev.dataTransfer.getData("text");
    ev.target.appendChild(document.getElementById(data));
  }

  // onMount(() => {
  // })

  function setNode(otherNode) {
    node = otherNode;
  }
  function selectChanged(node) {
    console.log({ selectedNode: node });
  }
  function keydown({ altKey, ctrlKey, shiftKey, key, target }) {
    console.log({ keydowntarget: target });
    if (target.contentEditable === "true") {
      return;
    }
    if (key === "a") {
      let newNode = addNode("descendants");
    }
    if (key === "c") {
      addNode("descendants");
      dispatch("making-current", { node });
    }
  }
  function clicked({ target }) {
    if (target) {
      console.log({ target });
    }
  }
  function createNode(rel) {
    return {
      self: node.path + "/",
      path: node.path + "/",
      state: "new",
      meta: {
        id: ts(),
        name: node.meta.name + "-"
      },
      matter: { md: "", html: "" },
      fathers: [{ descendants: [{ meta: { name: "sixxbing node!" } }] }],
      descendants: [],
      associates: [],
      edges: [{ rel, source: node.id }]
    };
  }
  function toggleView() {
    node.viewing = !node.viewing;
  }
  function makeCurrent() {
    console.log("making-current");
    dispatch("making-current", { node });
  }
  function update() {
    node.editing = false;
    node.matter.html = mdIt.render(node.matter.md);
  }
  function addNode(rel) {
    let freshNode = createNode(rel[0]);
    console.log({ freshNode });
    // currentNode.addingRel = rel;
    console.log("adding a " + rel, node);
    let newNode = node;
    newNode[rel].push(freshNode);
    node = newNode;
    //need to save relationship
    dispatch("adding-node", { rel, node });
    dispatch("node-changed", { node });
    return freshNode;
  }
  async function save() {
    console.log(node);
    let res;
    if (node.state === "stored") {
      res = await ajx.patch("/files", { data: node });
    } else {
      res = await ajx.post("/files", { data: node });
    }
    if (res.data) {
      node = res.data;
    }
  }
  function nodeChanged() {
    console.log({ change: node });
    dispatch("node-changed", { node });
  }
</script>

<style lang="scss">
  .node-grid {
    background-color: #aaaaaa50;
    display: grid;
    grid-template-columns: 5px 15px 1fr;
    // grid-template-rows: 10px 1fr 5px;
    grid-template-rows: 10px 10px 5px 1fr;
    grid-template-areas:
      "father father father"
      "associate link name"
      "descendant descendant descendant"
      "other other other";
  }
  .other {
    grid-area: other;
  }
  .father-link {
    visibility: hidden;
    grid-area: father;
    border: 1px solid black;
    border-radius: 50%;
    width: 6px;
    height: 6px;
    margin-top: -10px;
    cursor: pointer;
  }
  .descendant-link {
    visibility: hidden;
    grid-area: descendant;
    border: 1px solid black;
    border-radius: 50%;
    width: 8px;
    height: 8px;
    margin-bottom: -3px;
    cursor: pointer;
    margin-right: -30px;
  }
  .associate-link {
    visibility: hidden;
    grid-area: associate;
    border: 1px solid black;
    border-radius: 50%;
    margin-left: -8px;
    width: 6px;
    height: 6px;
    cursor: pointer;
  }
  .node-link {
    grid-area: link;
  }
  .node-name {
    grid-area: name;
  }
  .border {
    border: 1px solid transparent;
    border-radius: 2px;
    &:hover {
      border: 1px solid #aaa;
    }
  }
  .link-hover {
    &:hover {
      & > .father-link,
      & > .associate-link,
      & > .descendant-link {
        visibility: visible;
      }
    }
  }
  .main-node-border {
    border: 1px solid #aaa;
  }
</style>

<div
  {id}
  draggable="true"
  on:keydown={keydown}
  on:click={clicked}
  on:dragstart={drag}
  on:focus={() => dispatch('focusing-node', { node })}
  tabindex="0"
  class="border grid _c my__ node-grid w++ {current ? 'main-node-border' : ''}
  {grandchild ? '' : 'link-hover'}">
  <div class="father-link" on:click={() => addNode('fathers')} />
  <div class="associate-link" on:click={() => addNode('associates')} />
  <div class="node-link">
    {#if node.meta.link}
      <a class="p__" href={node.meta.link} target="_blank">
        <img
          src="http://s2.googleusercontent.com/s2/favicons?domain_url={node.link}" />
      </a>
    {/if}
  </div>
  <div title={node.state} class="node-name v-- -l ml">
    {#if current}
      <span
        contenteditable="true"
        on:input={nodeChanged}
        bind:innerHTML={node.meta.name} />
    {:else}
      <span
        contenteditable="true"
        on:input={nodeChanged}
        bind:innerHTML={node.meta.name} />
      {#each filteredNodes as option}
        <span on:click={() => setNode(option)}>Option: {option.name}</span>
      {/each}
    {/if}
    <span on:click={save}>=></span>
  </div>
  <div class="descendant-link" on:click={() => addNode('descendants')} />
  <div class="other -t">
    {#if show.descendants && node.descendants}
      {#each node.descendants as d}
        <svelte:self node={d} grandchild="true" />
      {/each}
    {/if}
  </div>
</div>
