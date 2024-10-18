<script>
   
    // Svelte lifecycle to initialize Masonry and Magnific Popup
    import { onMount } from 'svelte';
    import Masonry from 'masonry-layout';  // Import Masonry
    import imagesLoaded from 'imagesloaded';
    import './style.css';

    let images = [];
    let error = '';
    let isLoading = true;
    let page = 1;
    const limit = 40;
    // Unsplash API configuration
    const ACCESS_KEY = 'j4iFY9lw4Vig1ZVQPt-c4RDDj26RBGFQlhU8mGKgPCU'; // Replace this with your Unsplash API key
    const UNSPLASH_API_URL = `https://api.unsplash.com/photos?client_id=${ACCESS_KEY}&per_page=40`;

  
    let masonryInstance;
    let container;

    // Fetch images from Unsplash API on component mount
    onMount(async () => {
    try {
        const response = await fetch(UNSPLASH_API_URL);
        if (!response.ok) throw new Error('Error fetching images');
        images = await response.json();
        isLoading = false;
        console.log('Images fetched:', images);

       // Ensure images are loaded before initializing Masonry
       setTimeout(() => {
            if (container) {
                imagesLoaded(container, () => {
                masonryInstance = new Masonry(container, {
                    itemSelector: '.box',
                    columnWidth: 236,
                    percentPosition: true,
                    gutter: 30,
                });
            });
 
                console.log('Masonry instance:', masonryInstance);

            } else {
                console.error('Container is undefined');
            }
        }, 0);
    } catch (err) {
        error = err.message;
        isLoading = false;
    }


    return () => {
      // Cleanup masonryInstance when component is destroyed
      if (masonryInstance) {
        masonryInstance.destroy();
        masonryInstance = null;
      }
    };

    });
    console.log(masonryInstance);

    // Cleanup Masonry instance
    // onDestroy(() => {
    // if (masonryInstance) masonryInstance.destroy();
    // });
  </script>
  

  <div class="gallery-wrapper">
  {#if error}
    <p>{error}</p>
  {:else}
  <div bind:this={container} class="gallery">
     
    {#if isLoading}
      {#each Array(10) as _, i}
        <!-- Placeholder skeleton boxes before images load -->
        <div class="box">
          <div class="skeleton"></div>
        </div>
      {/each}
    {:else}
    
    {#each images as image, i}
    
        <div class="box drop-shadow lifted">
          <a href={`#pop${i}`} class="image-link">
            <div class="image-src"><img class="photo rounded_corner" loading="lazy" src={image.urls.small} alt={image.alt_description} /> </div>
         <div class="image-title">{image.alt_description}</div>
          </a>
          <div style="display: none;" id={`pop${i}`} class="white-popup mfp-hide">
            <div class="pop-it">
              <img src={image.urls.regular} alt={image.alt_description} />
            </div>
          </div>
        </div>
      {/each}
      {/if}
    </div>
  {/if}
</div>