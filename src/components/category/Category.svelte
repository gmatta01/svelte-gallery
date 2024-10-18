<!-- Category.svelte -->
<script>
    import { onMount } from 'svelte';
  
    export let categoryId; // Accept categoryId as a prop
    let posts = [];
    let error = '';
  
    onMount(async () => {
      try {
        const response = await fetch(`https://wptavern.com/wp-json/wp/v2/posts?categories=${categoryId}`);
        if (!response.ok) throw new Error('Failed to fetch posts');
        posts = await response.json();
      } catch (err) {
        error = err.message;
      }
    });
  </script>
  
  {#if error}
    <p>{error}</p>
  {:else}
    <div>
      {#each posts as post}
        <div class="post">
          <h2>{post.title.rendered}</h2>
          <p>{@html post.excerpt.rendered}</p>
        </div>
      {/each}
    </div>
  {/if}
  