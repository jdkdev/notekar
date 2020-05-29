<script>
  import { onMount, createEventDispatcher } from "svelte";
  export let value = "Nothing here";
  export let id;
  export let opts;

  let editor = {};
  let div;
  let oldValue = "";
  const dispatch = createEventDispatcher();

  /*
  this was a vue watch function
  $: value = function() { 
      //  two way binding – emit changes to parent
      dispatch('input', value)
      
      //  update value on external model changes
      if (oldValue !== value) { 
          editor.setValue(value, 1)
      }
  }
  */
  onMount(() => {
    createEditor();
  });

  function createEditor() {
    editor = window.ace.edit(div);

    editor.$blockScrolling = Infinity; //  deprecation fix

    const session = editor.getSession();
    session.on("changeAnnotation", () => {
      //  ignore doctype warnings
      const a = session.getAnnotations();
      const b = a.slice(0).filter(item => item.text.indexOf("DOC") == -1);
      if (a.length > b.length) session.setAnnotations(b);
    });
    //  editor options
    let options = opts || {};

    //  opinionated option defaults
    options.maxLines = options.maxLines || Infinity;
    options.printMargin = options.printMargin || false;
    options.highlightActiveLine = options.highlightActiveLine || false;

    //  hide cursor
    if (options.cursor === "none" || options.cursor === false) {
      editor.renderer.$cursorLayer.element.style.display = "none";
      delete options.cursor;
    }

    //  add missing mode and theme paths
    if (options.mode && options.mode.indexOf("ace/mode/") === -1) {
      options.mode = `ace/mode/${options.mode}`;
    }
    if (options.theme && options.theme.indexOf("ace/theme/") === -1) {
      options.theme = `ace/theme/${options.theme}`;
    }
    console.log({ editor });
    editor.setOptions(options);
    // editor.resize();

    //  set model value
    //  if no model value found – use slot content
    if (!value || value === "") dispatch("input", editor.getValue());
    else editor.setValue(value + "", -1);

    //  editor value changes
    editor.on("change", () => {
      console.log("changing values");
      console.log(editor.getValue());
      //  oldValue set to prevent internal updates
      value = oldValue = editor.getValue();
      dispatch("input", value);
    });
  }
  function getEditor() {
    return editor;
  }
</script>

<style>
  #editor {
    min-height: 500px;
  }
</style>

<div id="editor" class="flex-grow" bind:this={div}>{value}</div>
