<script>
  // Svelte lifecycle to initialize Masonry and Magnific Popup
  import { onMount, tick } from "svelte";
  import Masonry from "masonry-layout"; // Import Masonry
  import imagesLoaded from "imagesloaded";
  import "./style.css";

  let images = [];
  let error = "";
  let isLoading = false;
  let page = 1;
  const limit = 30;
  let imageUrls = [];
  let imageData = [];
  let imagesObject = {};
  let masonryInstance;
  let container;

  // Fetch a batch of images from the API

  async function fetchImages() {
    if (isLoading) return;
    isLoading = true;
    try {
      const storedImages = localStorage.getItem("images");
      if (storedImages) {
        console.log("Images fetched from local storage:");
        images = JSON.parse(storedImages);
        imagesObject = JSON.parse(localStorage.getItem("imagesObject"));
        imageData = JSON.parse(localStorage.getItem("imageData"));
        page = JSON.parse(localStorage.getItem("page"));
        imageUrls = JSON.parse(localStorage.getItem("imageUrls"));
      } else {
        const res = await fetch(
          `https://wptavern.com/wp-json/wp/v2/posts?_embed&page=${page}&per_page=${limit}`
        );
        if (!res.ok) throw new Error("Error fetching images");
        const data = await res.json();

        // Fetch featured images for each post and resolve promises
        const imagePromises = data.map(async post => {
          if (post.featured_media) {
            // Resolve to the actual URL object, not the promise
            const imageData = await fetchFeaturedImage(post.featured_media);
            return imageData; // Return only the URL
          }
          return null; // If no featured media, return null
        });

        imageData = await Promise.all(imagePromises);
        imageUrls = [...imageUrls, ...imageData];

        imagesObject = imageUrls.reduce((acc, curr) => {
          if (curr) {
            return { ...acc, ...curr }; // Merge objects
          }
          return acc;
        }, {});

        images = [...images, ...data]; // Append new images to the existing array

        localStorage.setItem("images", JSON.stringify(images));
        localStorage.setItem("imagesObject", JSON.stringify(imagesObject));
        localStorage.setItem("imageData", JSON.stringify(imageData));
        localStorage.setItem("page", JSON.stringify(page));
        localStorage.setItem("imageUrls", JSON.stringify(imageUrls));
      }

      console.log(imagesObject);

      await tick();
      page += 1; // Increment page count for the next batch

      if (masonryInstance) {
        imagesLoaded(container, () => {
          const items = container.querySelectorAll(".box:not(.added)");
          items.forEach(item => item.classList.add("added")); // Mark new items as added
          masonryInstance.appended(items);
          masonryInstance.layout(); // Re-layout Masonry
          window.addEventListener("scroll", handleScroll); // Re-enable scroll listener
        });
      } else {
        initializeMasonry(); // Initialize for the first time
      }
    } catch (err) {
      console.error("Error loading images:", err);
    }

    isLoading = false;
  }
 // Initialize Masonry once
 function initializeMasonry() {
  
  if (container) {
    imagesLoaded(container, () => {
      document.removeEventListener('DOMContentLoaded', initializeMasonry);
      masonryInstance = new Masonry(container, {
        itemSelector: ".box",
        columnWidth: 236,
        percentPosition: true,
        fitWidth: true,
        gutter: 20,
      });
      window.addEventListener('scroll', handleScroll);
    });
  } else {
    console.error("Container is undefined");
  }
}

// Wait for page content to load before initializing Masonry
document.addEventListener('DOMContentLoaded', initializeMasonry);
  
  // Check if the fetch-more element is in view
  function isFetchMoreVisible() {
    const fetchMoreElement = document.getElementById('fetch-more');
    if (fetchMoreElement) {
      const rect = fetchMoreElement.getBoundingClientRect();
      return rect.top <= window.innerHeight && rect.bottom >= 0; // Check if the element is in view
    }
    return false;
  }

   // Infinite scroll handler
   function handleScroll() {
    masonryInstance.layout();
    if (isFetchMoreVisible()) {
     console.log("fetching") // Fetch new images when fetch-more is in view
     fetchImages();
     window.removeEventListener('scroll', handleScroll); // Remove listener until fetch completes
    }
   
  }

  async function fetchFeaturedImage(imageId) {
  try {
    const res = await fetch(`https://wptavern.com/wp-json/wp/v2/media/${imageId}`);
    if (!res.ok) throw new Error("Error fetching image");
    const imageData = await res.json();
    // Return an object with the imageId as the key and the URL as the value
    return { [imageId]: imageData.media_details.sizes.medium.source_url };
  } catch (err) {
    console.error("Error loading image:", err);
    return null; // Return null in case of error to handle it later
  }
}

function trimSentence(sentence, wordLimit) {
  return sentence.split(' ').slice(0, wordLimit).join(' ');
}


 // Initialize Masonry after component mounts
 onMount(() => {
  initializeMasonry()
  fetchImages();
    return () => {
      window.removeEventListener('scroll', handleScroll); // Cleanup
     // if (masonryInstance) masonryInstance.destroy(); // Clean up masonry
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
  
        {#each images as image, i}
          <div class="box drop-shadow lifted">
            <a href={image.link} target="_blank" class="image-link">
              <div class="image-src">
                <img
                  class="photo rounded_corner"
                  loading="lazy"
                  src={imagesObject[image.featured_media]}
                  alt={image.title.rendered}
                />
              </div>
              <div class="image-title">{trimSentence(image.title.rendered, 6)}</div>
            </a>
          </div>
        {/each}
        
    </div>
      <div id="fetch-more"> 
        {#if isLoading}
       <span class="fixed-load"> Loading .... </span>
        {:else}
        That's it
     {/if}
      </div>
    
  {/if}
</div>
