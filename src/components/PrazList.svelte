<script>
  export let files;
  export let currentFile;
  export let editor;
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
  let EditSession = require("ace/edit_session").EditSession;
  function setupEditor(value, mode, theme = "solarized_dark") {
    let session = new EditSession(value);
    editor.setOptions(editoroptions);
    editor.setSession(session);
    editor.session.setMode(`ace/mode/${mode}`);
    editor.setTheme(`ace/theme/${theme}`);
    editor.setKeyboardHandler("ace/keyboard/vim");
    // editor.resize();
  }
  let edit = function(file) {
    currentFile = file;
    setupEditor(currentFile.markdown, "markdown", editoroptions.theme);
  };
</script>

{#each files as file (file.id)}
  <article class="mb">
    <span class="">{file.id}</span>
    <span class="">{file.meta.mode}</span>
    <!-- <span class="">
            {format(new Date(file.meta.created), 'MM/dd/yyyy')}
          </span> -->
    <span>{file.meta.created}</span>
    <span class="ml pr">
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
