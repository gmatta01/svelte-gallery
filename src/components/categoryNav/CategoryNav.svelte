<script>
    import { onMount } from 'svelte';
    import { links, Link, Router, Route } from 'svelte-routing';
    import "./style.css";
  
    let categories = [];
    let error = '';
  
    // Fetch categories from the API
    async function fetchCategories() {
      try {
        const response = await fetch('https://wptavern.com/wp-json/wp/v2/categories');
        if (!response.ok) throw new Error('Failed to fetch categories');
        categories = await response.json();
      } catch (err) {
        error = err.message;
      }
    }
  
    // Fetch categories on mount
    onMount(() => {
      fetchCategories();
    }); 
  </script>
  
  <style>
    .category-nav {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 20px;
      justify-content: center;
    }
    .navItem {
      padding: 10px 20px;
      background-color: #fff;
      border-radius: 50px;
      cursor: pointer;
      border: 1px solid #000;
      color: #000;
    }
    .navItem:hover {
      background-color: #2d55ff;
      border-color: #2d55ff;
      color:#fff;
    }
  </style>
  
  <div class="category-nav" use:links>
    {#if error}
      <p>{error}</p>
    {:else if categories.length === 0}
      <p>Loading categories...</p>
    {:else}
    <Router>
      {#each categories as category}
     
          <a href={`/category/${category.id}`} class="navItem">{category.name}</a> 
        
      {/each}
    </Router>
    {/if}
  </div>
  