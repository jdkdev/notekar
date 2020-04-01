<script>
  import { ajx } from "$frontier";
  import { onMount } from "svelte";
  import AceEditor from "$c/AceEditor.svelte";
  import { format, fromUnixTime } from "date-fns";
  import { Field } from "$frontier-c";

  let EditSession = require("ace/edit_session").EditSession;
  let editor = {};
  let editoroptions = {
    mode: "markdown",
    // theme: "solarized_dark",
    // theme: "tomorrow_night_eighties",
    // theme: "terminal",
    theme: "twilight",
    fontSize: 16,
    fontFamily: "monospace",
    highlightActiveLine: false,
    highlightGutterLine: false
  };
  let file = { email: "jordan+10@knight.works", password: "", site: "" };
  let files = [];
  let newFileView = false;
  const markdownIt = new markdownit();
  $: term = "";
  $: filteredFiles = files.filter(file => {
    if (term[0] == ":") {
      return file.contents.includes(term.substr(1));
    } else return file.name.includes(term);
  });
  $: currentFile = { contents: "", compiled: "" };
  $: currentMarkdown = markdownIt.render(currentFile.contents);

  let renderedTemplate = function(data) {
    //Can compiled be currentMarkdown?
    var compiled = markdownIt.render(data);
    currentFile.contents = data;
    currentFile.compiled = compiled;
  };

  // move validation to some higher function so that mount becomes
  onMount(() => {
    loadEditor();
    getFiles();
  });
  function loadEditor() {
    // editor = this.$children[0].editor;
    editor = ace.edit("editor");
    setEditorToMd();
  }

  function setEditorToMd() {
    setupEditor(currentFile.contents, "markdown", editoroptions.theme);
  }

  function setupEditor(value, mode, theme = "solarized_dark") {
    let session = new EditSession(value);
    editor.setOptions(editoroptions);
    editor.setSession(session);
    editor.session.setMode(`ace/mode/${mode}`);
    editor.setTheme(`ace/theme/${theme}`);
    editor.setKeyboardHandler("ace/keyboard/vim");
    // editor.resize();
  }
  function updateMarkup({ detail }) {
    // markup = detail;
    renderedTemplate(detail);
  }

  let getFiles = async function(withDeleted = "") {
    let res = await ajx.get("/files" + withDeleted);

    files = res.data.filter(file => file.date_added !== null);
  };

  let deactivate = async function(id) {
    let res = await ajx.destroy("/files/" + id);
    files = res.ok ? files.filter(u => u.id !== id) : files;
  };

  let reactivate = async function(id) {
    let res = await ajx.restore("/files/" + id);
    files = files.map(f => (f.id === id ? res : f));
  };

  let save = async function() {
    console.log(currentFile);
    let res = await ajx.patch("/files/" + currentFile.id, {
      data: currentFile
    });
    console.log(res);
    // files = res.error ? files : files.concat([res]);
  };
  let edit = function(file) {
    currentFile = file;
    setupEditor(currentFile.contents, "markdown", editoroptions.theme);
  };
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

<section class="gx-span-full p">
  <div>
    <button class="secondary" on:click={() => getFiles('/all')}>
      Show All
    </button>
    <button class="secondary" on:click={() => getFiles()}>Show Active</button>

    <button class="secondary" on:click={save}>Save</button>
    <button class="" on:click={() => (newFileView = !newFileView)}>New</button>
    {#if newFileView}
      <div>
        <Field name="email" bind:value={file.email} />
        <Field name="password" type="password" bind:value={file.password} />
        <Field name="site" bind:value={file.site} />
        <button on:click={save}>Save</button>
      </div>
    {/if}
  </div>

  <div class="core lm-display-grid">
    <Field name="search" bind:value={term} classes="gx-span-full" />
    <div
      tabindex="-1"
      id="files-list"
      class="grid-all gx-span-3 padding-bottom-jordan">
      {#each filteredFiles as file (file.id)}
        <article class="mb">
          <span class="">{file.name}</span>
          <span class="display-none sm-display lm-display-none">
            {file.perm}
          </span>
          <span class="display-none sm-display lm-display-none">
            {format(new Date(file.stats.ctime), 'MM/dd/yyyy')}
          </span>
          <span class="ml-auto pr">
            {#if file.is_deleted}
              <button on:click={() => reactivate(file.id)}>Reactivate</button>
            {:else}
              <button
                class="link"
                on:focus={() => edit(file)}
                on:click={() => edit(file)}>
                Edit
              </button>
            {/if}
          </span>
        </article>
      {:else}
        <div>No files found</div>
      {/each}
    </div>
    <div id="file-editor" class="gx-span-half">
      <AceEditor on:input={updateMarkup} />
    </div>
  </div>
</section>
<!-- <section id="output" class="max-w-5xl pr-20 mx-20">
  <article>
    {@html currentMarkdown}
  </article>
</section> -->
