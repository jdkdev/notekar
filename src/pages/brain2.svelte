<script>
  import { ajx } from "$frontier";
  import { onMount } from "svelte";
  import Node from "$c/Node.svelte";
  import Modal from "$c/Modal.svelte";
  import cytoscape from "cytoscape";
  import dagre from "cytoscape-dagre";

  cytoscape.use(dagre);
  $: cy = {};

  $: currentNode = {
    name: "",
    matter: { md: "", html: "" },
    parents: [{ children: [{ name: "silbing node!" }] }],
    children: [],
    partners: []
  };

  $: siblingNodes = (currentNode.parents || []).reduce(
    (result, parent) => (result = result.concat(parent.children)),
    []
  );

  $: brain = {};
  $: freshNode = { name: "I'm fresh!" };

  onMount(() => getBrain());

  async function getNode(node = currentNode) {
    //console.log({ currentNode });
    // let id = typeof node === "object" ? node.id : node;
    // let res = await ajx.get("/node/" + id);
    // currentNode = res.data;
  }
  async function getBrainX() {
    let res = await ajx.get("/files/brain");
    //TODO change this
    currentNode = res.data.brain.root;
    brain = res.data.brain;
    getNode();
    siblingNodes = currentNode.parents.reduce(
      (result, parent) => (result = result.concat(parent.children)),
      []
    );
  }
  function getBrain() {
    cy = cytoscape({
      container: document.getElementById("cy"),
      zoom: 4,
      userZoomingEnabled: false,
      autoungrabify: true,

      elements: [
        // nodes
        { data: { id: "Jordan" } },
        { data: { id: "Projects" } },
        { data: { id: "Resources" } },
        { data: { id: "Learn" } },
        { data: { id: "Archive" } },
        { data: { id: "Random" } },
        // edges
        {
          data: {
            id: "J-P",
            source: "Jordan",
            target: "Projects"
          }
        },
        {
          data: {
            id: "J-R",
            source: "Jordan",
            target: "Resources"
          }
        },
        {
          data: {
            id: "J-L",
            source: "Jordan",
            target: "Learn"
          }
        },
        {
          data: {
            id: "J-A",
            source: "Jordan",
            target: "Archive"
          }
        }
      ],

      layout: {
        name: "dagre"
      },

      style: [
        {
          selector: "node",
          style: {
            label: "data(id)",
            fontSize: 2,
            "background-color": "white",
            "border-color": "black",
            "border-style": "solid",
            "border-width": ".1",
            width: "label",
            height: "label",
            textHalign: "center",
            textValign: "center",
            padding: "2",
            "text-max-width": 100,
            "text-wrap": "ellipsis"
          }
        },
        {
          selector: "edge",
          style: {
            width: 0.2,
            "line-color": "#ccc",
            "target-arrow-color": "#ccc",
            "target-arrow-shape": "triangle",
            "curve-style": "unbundled-bezier",
            "arrow-scale": 0.1
          }
        }
      ]
    });
  }

  function createNode() {
    return {
      name: "# new thought! in Markdown!",
      matter: { md: "", html: "" }
    };
  }
  function addNode({ detail }) {
    freshNode = createNode();
    setCurrentNode({ detail: detail.node });
    currentNode.addingRel = detail.rel;
  }
  function setCurrentNode({ detail }) {
    currentNode = detail;
    getNode(); //will need async
  }

  function update() {
    node.editing = false;
    node.matter.html = mdIt.render(node.matter.md);
  }
</script>

<style>
  #cy {
    width: 100%;
    height: 100%;
    position: absolute;
  }
</style>

<div class="brain-grid">
  <div id="cy">test</div>
</div>
