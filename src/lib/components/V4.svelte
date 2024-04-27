<script lang="ts">
    let currentImage = "images/africanamerican.png"; // Store the path of the current image
  
    // Function to set the current image when a button is clicked
    function setImage(imagePath: string) {
      currentImage = imagePath;
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
      <button on:click={() => setImage("images/africanamerican.png")}
        >African American</button
      >
      <button on:click={() => setImage("images/asian.png")}>Asian</button>
      <button on:click={() => setImage("images/hispanic.png")}>Hispanic</button>
      <button on:click={() => setImage("images/white.png")}>White</button>
    </div>

    <!-- Display the selected image -->
  </div>

  <!-- <div class="text-container">
    <h1 class="text">
      Despite Boston's reputation for diversity and its plethora of vibrant
      neighborhoods, there exists a troubling pattern of inequality, particularly
      concerning eviction rates among minority-heavy communities. While the city
      boasts a rich tapestry of cultures and identities, certain neighborhoods
      with higher minority populations often bear the brunt of eviction crises
      than those that boost a higher white or asian population. This disparity
      underscores systemic issues of housing discrimination, economic disparity,
      and unequal access to resources. Despite efforts to address these
      disparities, such as through community outreach programs and policy
      initiatives, the persistent prevalence of evictions in minority-heavy
      neighborhoods highlights the urgent need for comprehensive and equitable
      solutions to ensure fair and just housing opportunities for all residents of
      Boston.
    </h1>
  </div> -->
</div>
  
<style>
  .v4-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  /* .text {
    top: 50%;
    width: 800px;
    padding: 10px;
    background-color: #feb64a;
    color: #F8F7F5;
    font-weight: bold;
    font-size: 14px;
    line-height: 1.5;
    justify-content: center;
  }
  .text-container {
    display: flex;
    align-items: center;
    justify-content: center;
  } */

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
    background-color: #ED5701;
    cursor: pointer;
    outline: none;

  }
</style>
  