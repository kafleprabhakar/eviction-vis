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

	<div class='vis-spacer2'>
		<h3 class='toWho'>Who Are These Evictions Impacting?</h3>
		<p style='color:#403E3A; text-align:left; margin-left:1.5rem; margin-right:1.5rem'> While Boston holds a reputation for its diversity and vibrant
			neighborhoods, there exists a troubling pattern of inequality within the area, particularly
			concerning eviction rates among minority-heavy communities. The city
			boasts a rich tapestry of cultures and identities, however certain neighborhoods
			with <b>higher minority populations often bear the brunt of eviction crises</b>
			compared to those that boast a higher White or Asian population. This disparity
			underscores systemic issues of housing discrimination, economic disparity,
			and unequal access to resources. Efforts such as community outreach programs and policy initiatives
			attempt to address these disparities, but the persistent prevalence of evictions in minority-heavy
			neighborhoods highlights <b>the vital need for comprehensive and equitable
			solutions</b> to ensure fair and just housing opportunities for all residents of
			Boston. </p>
		<p style='color:#403E3A; text-align:left; margin-left:1.5rem; margin-right:1.5rem'>To further examine the relationship between race and evictions, we can visualize the <b>correlation between 
			different racial demographic populations and eviction rates</b> in the Boston area. Below, the graphs display the percentage 
			of 4 different racial demographic populations graphed against the percentage of evictions per neighborhood. </p>
	</div>

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

    <div class='takeaway'>
        <p>The correlations depict an <b>upward trend for Asian and White populations</b> while depicting a <b>downward trend for Black and Hispanic populations</b>. 
        This means that neighborhoods with a higher population of historically marginalized minorities tend to face more evictions. While there are many factors at play, it is generally more common for Black and Hispanic individuals to be evicted in the greater Boston area. 
        </p>
    </div>
  </div>

</div>
  
<style>
  .takeaway{
    margin-left: 10rem;
    margin-right: 10rem;
  }
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

	.vis-spacer2 {
		background-color: #feb64a;
		color: #F8F7F5;
		align-self: center;
		display: flex;
		flex-direction: column;
		padding: 5.5rem;
		/* padding-right: 10rem;
		padding-left: 10rem; */
		margin-bottom: 5rem;
		text-align: center;
	}

	.toWho {
		color: #F8F7F5;
		margin: 0.5rem;
		font-size: 3rem;
	}
</style>
  