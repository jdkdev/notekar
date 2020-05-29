<script>
  import { auth, currentUser } from "@frontierjs/frontend";
</script>

<svelte:head>
  <title>FrontierJS</title>
</svelte:head>

<section class="p">
  <article>
    <div class="">Logged in: {$currentUser.email}</div>
  </article>
</section>
