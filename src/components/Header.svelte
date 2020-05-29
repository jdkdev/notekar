<script>
  import { auth, currentUser } from "@frontierjs/frontend";
  import Brand from "$c/Brand.svelte";
  import Nav from "$c/Nav.svelte";
  import { isActive, url, goto } from "$router";
</script>

<header>
  {#if $currentUser}
    <article class="[ fl block _y t ] h@vh _p">
      <div>NOTEKAR</div>
      <div>
        <div class="dn d@lg">Logged in:</div>
        <div class="">{$currentUser.email}</div>
        <button class="link" on:click={$auth.logout()}>Logout</button>
      </div>
      <a class="" href={$url('/')}>Home</a>
      <a class="" href={$url('/dashboard')}>Dashboard</a>
      <a class="" href={$url('/example')}>Examples</a>
      <a class="" href={$url('/brain2')}>Brain2</a>
      <a class="" href={$url('/brain')}>Brain</a>
      <span class="spacer" />
    </article>
  {:else}
    <article class="py bg-black v-grey vc">
      <h1>
        <span>&lt;!--</span>
        Notekar
        <span>--></span>
      </h1>
      <h2 class="pt">Make comments at your own risk....</h2>
    </article>
  {/if}
</header>
