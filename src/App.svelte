<script>
	export let name;
	import { Router, Link, Route } from 'svelte-routing';
	import Gallery from './components/gallery/Gallery.svelte';
	import CategoryNav from './components/categoryNav/CategoryNav.svelte';
	import Category from './components/category/Category.svelte';
	
</script>

<main>
	<h1>Hello {name}!</h1>
	<p>Aim to build gallery plugin with masonry taking feel from multiple sources. Elementor to start with. Using Svelte.</p>
	<ul>
		<li>** Setup Local Storage for faster response</li>
	</ul>
	<main>
		<Router>
			<Route path="/" component={CategoryNav} />
			<Route path="/category/:categoryId" component={Category} />
			
		  </Router>

		
		<Gallery />
	</main>
</main>
 
<style>
	main { 
		text-align: center;
		padding: 1em;
		margin: 0 auto;
	}

	h1 {
		color: #333;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>