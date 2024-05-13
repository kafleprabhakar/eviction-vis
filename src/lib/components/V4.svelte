<script lang="ts">
    let currentImage = "images/africanamerican.png"; // Store the path of the current image
    let selectedButton = "african-american"; // Store the ID of the selected button
  
    // Function to set the current image when a button is clicked
    function setImage(imagePath: string, buttonID: string) {
      currentImage = imagePath;
      selectedButton = buttonID;
    }
</script>
  
<div class='v4-container'>
  <h3>
    Select a race to see the percentage of that group per neighborhood in Boston
    against the percentage of evictions in that neighborhood in Boston
  </h3>
  <div class="contain">
    {#if currentImage}
      <img src={currentImage} alt="Selected" height="500" />
    {/if}
    
    <div class="button-container">
      <!-- Buttons to select different images -->
      <button on:click={() => setImage("images/africanamerican.png", "african-american")} class:selected={selectedButton === "african-american"}>African American</button>
      <button on:click={() => setImage("images/asian.png", "asian")} class:selected={selectedButton === "asian"}>Asian</button>
      <button on:click={() => setImage("images/hispanic.png", "hispanic")} class:selected={selectedButton === "hispanic"}>Hispanic</button>
      <button on:click={() => setImage("images/white.png", "white")} class:selected={selectedButton === "white"}>White</button>
    </div>

    <!-- Display the selected image -->
  </div>

</div>
  
<style>
  .v4-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .contain {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .button-container {
    display: flex;
    margin-top: 2rem;
    margin-bottom: 1rem;
  }

  button {
    background-color: #072B69;
    color: #F8F7F5;
    font-weight: bold;
    font-size: 1.25rem;
    line-height: 1.5;
    margin:.5rem;
    border-radius:.8rem;
    min-width: 15rem;
    padding: .4rem 0;
    outline: none;
  }

  button:hover {
    background-color: #feb64a;
    cursor: pointer;
    outline: none;

  }

  .selected {
    background-color: #feb64a; /* Change to desired color for the selected button */
  }
</style>
  