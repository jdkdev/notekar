<script>
  import { ajx } from "$frontier";
  import { writable } from "svelte/store";
  import { onMount } from "svelte";
  import Node from "$c/Node.svelte";
  import Modal from "$c/Modal.svelte";
  import NodeEditor from "$c/NodeEditor.svelte";

  // let kanbanView = true;
  let kanbanView = false;
  let show = {
    descendants: kanbanView
  };
  $: focusedNode = {
    meta: {
      name: ""
    },
    matter: { md: "", html: "" },
    fathers: [
      {
        meta: { name: "father one" },
        descendants: [{ meta: { name: "silbing node!" } }]
      }
    ],
    descendants: [],
    associates: []
  };
  $: currentNode = {
    meta: {
      name: ""
    },
    matter: { md: "", html: "" },
    fathers: [
      {
        meta: { name: "other" },
        descendants: [{ meta: { name: "silbing node!" } }]
      }
    ],
    descendants: [],
    associates: []
  };

  $: siblingNodes = [];

  $: brain = { nodes: [] };
  $: freshNode = { meta: { name: "I'm fresh!" } };
  $: searchText = "";
  $: filteredNodes = brain.nodes.filter(node => {
    let searchString = node.matter.md + node.self;
    return searchString.includes(searchText);
  });

  onMount(() => {
    getBrain();
  });
  function updateNode({ detail }) {
    console.log({ updatedNode: detail });
    focusedNode = detail.node;
  }
  function updateCurrentNode({ detail }) {
    let children = currentNode.descendants.length;
    currentNode = detail.node;
    console.log({ children });
    if (currentNode.descendants.length > children) {
      document.getElementById("node-" + (children + 2)).focus();
    }
  }
  function* Counter(i = 0) {
    while (true) {
      yield "node-" + i++;
    }
  }
  let idGen = Counter();

  async function getNode(node = currentNode) {
    //console.log({ currentNode });
    // let id = typeof node === "object" ? node.id : node;
    // let res = await ajx.get("/node/" + id);
    // currentNode = res.data;
  }
  async function getBrain() {
    let res = await ajx.get("/files/brain2");
    //TODO change this
    let node = getClosest(res.data.root);
    currentNode = node;
    //TODO need to store the brain in svelte store and make it accessible everywhere
    brain = res.data.brain;
    getNode();
    siblingNodes = currentNode.fathers.reduce((result, father) => {
      console.log({ father });
      result = result.concat(father.descendants);
    }, []);
  }

  function createNode() {
    return {
      meta: { name: "# new thought! in Markdown!" },
      matter: { md: "", html: "" }
    };
  }
  function addNode({ detail }) {
    console.log({ detail });
    // freshNode = createNode();
    // setCurrentNode({ detail: detail.node });
    // currentNode.addingRel = detail.rel;
    // detail.node[detail.rel].push(focusedNode);
  }

  function setCurrentNode({ detail }) {
    currentNode = detail.node;
    getNode(); //will need async
  }
  function getClosest(node) {
    console.log({ beginning: node });
    node.descendants = node.descendants.map(descendant =>
      brain.nodes.find(bNode => bNode.id === descendant)
    );
    node.fathers = node.fathers.map(father =>
      brain.nodes.find(bNode => bNode.id === father)
    );
    node.associates = node.associates.map(associate =>
      brain.nodes.find(bNode => bNode.id === associate)
    );
    console.log({ closet: node });
    return node;
  }
  function setToCurrent(node) {
    console.log({ currently: node });
    node = getClosest(node);
    currentNode = node;
    getNode(); //will need async
  }

  function setFocusedNode({ detail }) {
    console.log({ detail });
    console.log({ focusedNode });
    focusedNode = detail.node;
  }

  function update() {
    node.editing = false;
    node.matter.html = mdIt.render(node.matter.md);
  }
  function showEditor({ detail }) {
    //TODO we just need a watcher onchange for when a node changes
    focusedNode = detail.node;
  }
  function updateFocusedNode({ detail }) {
    focusedNode = detail.node;
  }
  function nodeUpdated({ detail }) {
    console.log({ updateddd: detail });
  }
</script>

<style>
  .current-node {
    background-color: aliceblue;
  }
  .border {
    border: 1px solid black;
  }
  .brain-grid-kanban {
    background: #eee;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 20%;
    grid-template-rows: 35px 30px 60px 1fr 150px;
    grid-template-areas:
      "menu menu menu menu"
      "pins pins pins sidebar"
      "current current current sidebar"
      "descendants descendants descendants sidebar"
      "crumbs crumbs crumbs sidebar";
  }
  .brain-grid {
    background: #eee;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 30%;
    grid-template-rows: 35px 30px 200px 200px 325px 150px;
    grid-template-areas:
      "menu menu menu menu"
      "pins pins pins sidebar"
      "associates fathers sibilings sidebar"
      "associates current siblings sidebar"
      "descendants descendants descendants sidebar"
      "crumbs crumbs crumbs sidebar";
  }
  .menu {
    grid-area: menu;
  }
  .father-nodes {
    grid-area: fathers;
  }
  .descendants-nodes {
    grid-area: descendants;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(270px, 1fr));
  }
  .sibling-nodes {
    grid-area: siblings;
  }
  .associates-nodes {
    grid-area: associates;
  }
  .d2-nodes {
    /* grid-area: d2; */
  }
  .current-node {
    grid-area: current;
  }
  .pinned-nodes {
    grid-area: pins;
  }
  .breadcrumbs {
    grid-area: crumbs;
  }
  .sidebar {
    grid-area: sidebar;
  }
</style>

<div class={kanbanView ? 'brain-grid-kanban' : 'brain-grid'}>
  <div class="menu border">
    <input
      placeholder="Search.."
      class="w- "
      type="text"
      bind:value={searchText} />
    <div class="row ( c fr v-- ) ">
      <span>Sidebar:</span>
      <div class="spacer" />
      <div>
        <button class="" on:click={() => (kanbanView = !kanbanView)}>
          Kanban
        </button>
        <button class="">Split</button>
        <button class="">Hide</button>
        <button class="">Full</button>
      </div>
    </div>
  </div>
  <div class="pinned-nodes">this for pinned nodes</div>
  <div class="{kanbanView ? 'dn' : 'd'} father-nodes border m p">
    {#if currentNode.fathers}
      {#each currentNode.fathers as node}
        <Node
          id={idGen.next().value}
          {node}
          on:adding-node={addNode}
          on:focusing-node={setFocusedNode}
          on:making-current={setCurrentNode} />
      {/each}
    {/if}
    <div class={currentNode.addingRel == 'father' ? 'd' : 'dn'}>
      <Node
        id={idGen.next().value}
        node={freshNode}
        on:adding-node={addNode}
        on:making-current={setCurrentNode} />
    </div>
  </div>
  <div class="{kanbanView ? 'dn' : 'd'} sibling-nodes">
    {#each siblingNodes as node}
      <Node
        id={idGen.next().value}
        {node}
        on:adding-node={addNode}
        on:focusing-node={setFocusedNode}
        on:making-current={setCurrentNode} />
    {/each}
  </div>
  <div class="{kanbanView ? 'dn' : 'd'} associates-nodes border m p">
    {#if currentNode.associates}
      {#each currentNode.associates as node}
        <Node {node} on:adding-node={addNode} />
      {/each}
    {/if}
    <div class={currentNode.addingRel == 'associate' ? 'd' : 'dn'}>
      <Node
        id={idGen.next().value}
        node={freshNode}
        on:adding-node={addNode}
        on:focusing-node={setFocusedNode}
        on:making-current={setCurrentNode} />
    </div>
  </div>
  <div class="xblock c current-node">
    <Node
      node={currentNode}
      on:node-changed={updateCurrentNode}
      on:node-changed={updateNode}
      on:adding-node={addNode}
      on:focusing-node={setFocusedNode}
      on:showing-editor={showEditor}
      current="true" />
  </div>

  <div class="descendants-nodes border m p">
    <div class={currentNode.addingRel == 'descendant' ? 'd' : 'dn'}>
      <Node
        id={idGen.next().value}
        node={freshNode}
        on:adding-node={addNode}
        on:focusing-node={setFocusedNode}
        on:making-current={setCurrentNode} />
    </div>
    {#each currentNode.descendants as node}
      <Node
        {node}
        {show}
        id={idGen.next().value}
        on:adding-node={addNode}
        on:node-changed={updateNode}
        on:focusing-node={setFocusedNode}
        on:making-current={setCurrentNode} />
    {/each}
  </div>
  <div class="breadcrumbs">this for breadcrumbs</div>
  <div class="border sidebar">
    <div class="notes">
      <NodeEditor node={focusedNode} on:node-changed={updateFocusedNode} />
    </div>
    <div>
      {#each filteredNodes as node}
        <span on:click={() => setToCurrent(node)}>{node.self}</span>
      {:else}
        <!-- empty list -->
        <span>Nothing found</span>
      {/each}
    </div>
  </div>
</div>
