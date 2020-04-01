<script>
  import { auth, currentUser } from "@frontierjs/frontend";
</script>

<svelte:head>
  <title>FrontierJS</title>
</svelte:head>

<section class="mb-auto p">
  <article>
    <div class="mb-auto">Logged in: {$currentUser.email}</div>
  </article>
</section>
