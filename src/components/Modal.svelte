<script>
  export let header = "";
  export let content = "";

  let open = false;
  let overlay = false;

  function toggleModal() {
    open = !open;
    overlay = !overlay;
  }
</script>

<div class="">
  <div style="">
    <button id="mybtn" on:click={() => toggleModal()}>

      <slot name="modal-button">Open</slot>
    </button>
  </div>
</div>
<div
  on:click={() => toggleModal()}
  class={overlay ? 'modal-overlay-open' : ''}
  id="modal-overlay" />
<div id="modal" class="modal {open ? 'open' : ''}">
  <div class="modal-content">
    <header>{header}</header>
    <p contenteditable="true">{content}</p>
    <slot />
  </div>
</div>
