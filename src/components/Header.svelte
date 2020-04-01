<script>
  import { auth, currentUser } from "@frontierjs/frontend";
  import Brand from "$c/Brand.svelte";
  import Nav from "$c/Nav.svelte";
  import { isActive, url, goto } from "$router";
</script>

<header class="gx-span-full">
  {#if $currentUser}
    <article class="m">
      <span class="stretch">
        <span class="show-md">Logged in:</span>
        {$currentUser.email}
      </span>
    </article>
    <article class="row space-around">
      <a class="p p-md" href={$url('/')}>Home</a>
      <a class="" href={$url('/dashboard')}>Dashboard</a>
      <a class="" href={$url('/example')}>Examples</a>
    </article>
  {:else}
    <article style="background-color: #000; color:#ccc;" class="p-xl">
      <h1 class="font-size-xxl" style="font-style:italic">
        &lt;!-- Notekar -->
      </h1>
      <h2 class="pt">Make comments at your own risk....</h2>
    </article>
  {/if}
</header>
