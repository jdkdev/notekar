<script>
  import { ajx } from "$frontier";
  import { onMount } from "svelte";
  import { format, fromUnixTime } from "date-fns";
  import { Field } from "$frontier-c";
  import AceEditor from "$c/AceEditor.svelte";
  import PrazList from "$c/PrazList.svelte";
  const markdownIt = new markdownit();

  let editor = {};
  let showMarkdown = true;
  let files = [];
  let file = { email: "jordan+10@knight.works", password: "", site: "" };
  let newFileView = false;

  $: term = "";
  $: filteredFiles = files.filter(file => {
    if (term[0] == ":") {
      return file.markdown.includes(term.substr(1));
    } else {
      return file.id.includes(term);
    }
  });

  $: currentFile = { markdown: "", html: "", compiled: "" };
  $: currentMarkdown = markdownIt.render(currentFile.markdown);

  $: pFiles = filteredFiles.filter(file => file.praz === "p");
  $: rFiles = filteredFiles.filter(file => file.praz === "r");
  $: aFiles = filteredFiles.filter(file => file.praz === "a");
  $: zFiles = filteredFiles.filter(file => file.praz === "z");
  // move validation to some higher function so that mount becomes

  onMount(() => {
    editor = ace.edit("editor");
    getFiles();
  });

  function renderedTemplate(data) {
    //Can compiled be currentMarkdown?
    let html = markdownIt.render(data);
    currentFile.markdown = data;
    currentFile.html = html;
  }

  function updateMarkup({ detail }) {
    // markup = detail;
    renderedTemplate(detail);
  }

  async function getFiles(withDeleted = "") {
    let res = await ajx.get("/files/raw" + withDeleted);
    files = res.data;
  }

  async function deleteFiles(id) {
    let res = await ajx.destroy("/files/" + id);
    files = res.ok ? files.filter(u => u.id !== id) : files;
  }

  async function reactivate(id) {
    let res = await ajx.restore("/files/" + id);
    files = files.map(f => (f.id === id ? res : f));
  }

  async function save() {
    console.log(currentFile);
    let res = await ajx.patch("/files/" + currentFile.id, {
      data: currentFile
    });
    console.log(res);
    // files = res.error ? files : files.concat([res]);
  }
</script>

<style>
  #file-editor {
    min-width: 60vw;
    overflow-y: scroll;
    max-height: 60vh;
    padding-bottom: 3rem;
    margin-bottom: 5rem;
  }
  #files-list {
    overflow-y: scroll;
    max-height: 60vh;
    padding-right: 1rem;
    margin-bottom: 1rem;
  }
</style>

<section class=" p">
  <div class="grid l">
    <div class="">
      <button class="secondary" on:click={() => getFiles('/all')}>
        Show All
      </button>
      <button class="secondary" on:click={() => getFiles()}>Show Active</button>

      <button class="secondary" on:click={save}>Save</button>
      <button class="" on:click={() => (newFileView = !newFileView)}>
        New
      </button>
      <button class="" on:click={() => (showMarkdown = !showMarkdown)}>
        MD
      </button>
    </div>
    <Field name="search" bind:value={term} classes="" />
    {#if newFileView}
      <div>
        <Field name="email" bind:value={file.email} />
        <Field name="password" type="password" bind:value={file.password} />
        <Field name="site" bind:value={file.site} />
        <button on:click={save}>Save</button>
      </div>
    {/if}
  </div>

  <div class="core">
    <div tabindex="-1" id="files-list" class="">
      <PrazList {editor} {currentFile} files={rFiles} />
    </div>
    <div id="file-editor" class="">
      {#if !showMarkdown}
        <article>
          {@html currentMarkdown}
        </article>
      {:else}
        <AceEditor on:input={updateMarkup} />
      {/if}
    </div>
  </div>
</section>
