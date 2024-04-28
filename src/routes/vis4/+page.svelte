<script lang="ts">
  let currentImage = ""; // Store the path of the current image

  // Function to set the current image when a button is clicked
  function setImage(imagePath: string) {
    currentImage = imagePath;
  }
</script>

<p>
  Select a race to see the percentage of that group per neighborhood in Boston
  against the percentage of evictions in that neighborhood in Boston.
</p>

<!-- Buttons to select different images -->
<button on:click={() => setImage("white.png")}>White</button>
<button on:click={() => setImage("asian.png")}>Asian</button>
<button on:click={() => setImage("hispanic.png")}>Hispanic</button>
<button on:click={() => setImage("africanamerican.png")}
  >African American</button
>

<!-- Display the selected image -->
{#if currentImage}
  <img src={currentImage} alt="Selected Image" width="200" height="200" />
{/if}
