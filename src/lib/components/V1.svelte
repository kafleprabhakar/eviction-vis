<script>
    import { onMount } from 'svelte';
  
    let infoSectionOpacity = 1;
    let leftBlockOpacity = 0;
    let mapImageOpacity = 0;
    let rightBlockOpacity = 0;
    let containersOpacity = 0;
    let returnToTopVisible = false;

    function handleMouseEnter(event) {
    event.currentTarget.querySelector('.text').style.display = 'block';
  }

  function handleMouseLeave(event) {
    event.currentTarget.querySelector('.text').style.display = 'none';
  }

  
    function handleScroll() {
      const scrollPosition = window.scrollY;
      infoSectionOpacity = scrollPosition > 200 ? 0 : 1;
      leftBlockOpacity = scrollPosition > 200 ? 1 : 0;
      mapImageOpacity = scrollPosition > 200 ? 1 : 0;
      rightBlockOpacity = scrollPosition > 200 ? 1 : 0;
      containersOpacity = scrollPosition > 200 ? 1 : 0;
      returnToTopVisible = scrollPosition > 200;
    }
  
    function returnToTop() {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }
  
    onMount(() => {
        window.addEventListener('scroll', handleScroll);
        // window.addEventListener('mouseenter', handleMouseEnter);
        // window.addEventListener('mouseleave', handleMouseLeave);
        handleScroll();
        // handleMouseEnter();
        // handleMouseLeave();

        const containers = document.querySelectorAll('.container');
    containers.forEach(container => {
      container.addEventListener('mouseenter', handleMouseEnter);
      container.addEventListener('mouseleave', handleMouseLeave);
    });

    // Clean up event listeners on component unmount
    return () => {
      containers.forEach(container => {
        container.removeEventListener('mouseenter', handleMouseEnter);
        container.removeEventListener('mouseleave', handleMouseLeave);
      });
    };
    });
  
  </script>
  
<style>
    div.container {
        position: absolute;
        display: inline-block;
        opacity: var(--containersOpacity); /* Start with opacity 0 */
        transition: opacity 0.5s ease;
        /* top: 50%; /* Initially center vertically */
        /* left: 50%;  */ 
        /* transform: translate(-50%, -50%);  */
    }
    .image {
        max-width: 100%;
        height: auto;
        margin: 0; /* Reset margin */
        padding: 0; /* Reset padding */
  
    }
    .text {
        display: none;
        position: absolute;
        top: 50%;
        width: 300px;
        transform: translateY(-50%);
        padding: 10px;
        background-color: #FEB64A;
        font-weight: bold;
        font-size: 14px;
        text-align: left;
        line-height: 1.5;
        z-index: 999; 
    }

    .text-container {
        display: flex;
        align-items: center;
    }
    .text-container img {
        margin-right: 10px;
    }
    .left-block {
        /* position: absolute; */

        width: 25%; /* Adjust the width as needed */
        background-color: #072B69;

        color: white;
        padding: .8rem 1.5rem;
        text-align: center;
        box-sizing: border-box; /* Include padding in the width */
        font-size: 1.7rem;
        font-weight: bold; /* Make the text bold */
        line-height: 1.5; /* Increase line height */
        opacity: var(--leftBlockOpacity); /* Start with opacity 0 */
        transition: opacity 0.5s ease;
        margin-left: 5rem;
        
    }
    .mapimage{
        /* position: absolute; */
        width: 450px;
        height: 300px;
        border: 1px solid #ccc;
        opacity: var(--mapImageOpacity); /* Start with opacity 0 */
        transition: opacity 0.5s ease;
        margin: 5rem 0; 
        margin-left: calc(100vw - 550px);
    }

    .info {
        display: flex;
        text-align: center;
        align-self: center;
        width: 75%;
        margin-bottom: 8rem;
        opacity: var(--infoSectionOpacity); /* Start with opacity 1 */
        transition: opacity 0.3s ease; /* Add transition for opacity */
    }

    #returnToTopBtn {
        position: fixed;
        bottom: 20px;
        left: 20px;
        z-index: 999;
        display: none; /* Initially hide the button */
        padding: 10px 20px;
        background-color: #ED5701;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        transition: opacity 0.3s ease; /* Add transition for opacity */
    }
    #returnToTopBtn:hover {
        background-color: #FF784E;
    }

    .all-blocks{
        display: flex;
        flex-direction: column;
        transition: opacity 0.5s ease;
    }
</style>

<!-- <div class="info" style="opacity:{infoSectionOpacity}"> -->

<div class="info" style="opacity:{infoSectionOpacity}">
    <p>
        Boston is one of the most expensive places to live in the United States. With the second highest average monthly rent index in the entire country, a third of renters pay more than 30 percent of their income on housing costs. Any change in income including rent increase, a medical bill or a loss of a job can cause a low-income person to fall behind on payment. Those who can no longer keep up with rent are physically removed or forced to leave their own homes. Hover over the orange icons to hear their stories.
    </p>
</div>

<div class='all-blocks' style="opacity:{leftBlockOpacity}" >

    <div class="left-block" >
        <p>Between 2010 and 2019, property owners filed more than 50,000 evictions in Boston Housing Court</p>
    </div>

    <!-- map -->
    <img class ="mapimage" src="images/bostonMap.png" alt="Boston Map">

    <!-- All blanks -->
    <div class="container" style="top: 605px; left: 1225px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 800px; left: 1300px;" >
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px">
    </div>
    <div class="container" style="top: 620px; left: 830px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 1020px; left: 720px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 780px; left: 670px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 1330px; left: 1230px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 1600px; left: 530px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 950px; left: 200px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 1280px; left: 600px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 1150px; left: 300px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>
    <div class="container" style="top: 1500px; left: 1050px;">
        <img class="image" src="images/clear.png" alt="Person" style="width: 50px;">
    </div>

    <div class="container" style="top: 750px; left: 1050px;">
        <img class="image" src="images/orange.png" alt="Orange Person" style="width: 50px;">
        <div class="text" style="right: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <img src="images/domingo.png" alt="Hover Person" style="width: 200px;">
                <p>"Fannie Mae took over my [bank] account, and sold my house to investors, who sold it to a private company. Now, alone, I’m facing eviction. This ordeal has made me suffer more than you can possibly imagine."</p>
            </div>
        </div>
    </div>

    <div class="container" style="top: 1000px; left: 450px;">
        <img class="image" src="images/orange.png" alt="Orange Person" style="width: 50px;">
        <div class="text" style="left: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <img src="images/margarita.png" alt="Hover Person" style="width: 200px;">
                <p>"The first time I was evicted, I was given only a week to leave. I am no young lady, but I could not afford anything but to do the moving alone. I worried about the safety of my grandson, who lived with me."</p>
            </div>
        </div>
    </div>

    <div class="container" style="top: 1400px; left: 850px;">
        <img class="image" src="images/orange.png" alt="Orange Person" style="width: 50px;">
        <div class="text" style="right: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <img src="images/dora.png" alt="Hover Person" style="width: 200px;">
                <p>"A little over a year ago, all six families living in my building were given eviction notices. We were given 30 days to leave. I felt lost and scared."</p>
            </div>
        </div>
    </div>

    <div class = "left-block">
        <p>In Boston, around 80% of evictions are reportedly filed for non-payment of rent.</p>
    </div>

</div>

<button id="returnToTopBtn">Return to Top</button>

 
<!-- Add more container and image pairs as needed -->

<!-- <script>
    document.querySelectorAll('.container').forEach(container => {
        const text = container.querySelector('.text');

        container.addEventListener('mouseenter', () => {
            text.style.display = 'block';
        });

        container.addEventListener('mouseleave', () => {
            text.style.display = 'none';
        });
    });
    function handleScroll() {
        const scrollPosition = window.scrollY;
        const infoSection = document.querySelector('.info');
        const leftBlock = document.querySelector('.left-block');
        const mapImage = document.querySelector('.mapimage');
        const rightBlock = document.querySelector('.right-block');
        const containers = document.querySelectorAll('.container');

        if (scrollPosition > 200) {
            infoSection.style.opacity = '0'; // Hide info text by changing opacity to 0
            leftBlock.style.opacity = '1';
            mapImage.style.opacity = '1';
            rightBlock.style.opacity = '1';
            containers.forEach(container => {
                container.style.opacity = '1';
            });
        } else {
            infoSection.style.opacity = '1'; // Show info text by changing opacity to 1
            leftBlock.style.opacity = '0';
            mapImage.style.opacity = '0';
            rightBlock.style.opacity = '0';
            containers.forEach(container => {
                container.style.opacity = '0';
            });
        }
    }
    window.addEventListener('scroll', handleScroll);

    handleScroll(); 


    // return to top
    function returnToTop() {
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    const returnToTopBtn = document.getElementById('returnToTopBtn');

    if (returnToTopBtn) {
        returnToTopBtn.addEventListener('click', returnToTop);
    }

    window.addEventListener('scroll', () => {
        if (returnToTopBtn) {
            if (window.scrollY > 200) {
                returnToTopBtn.style.display = 'block';
            } else {
                returnToTopBtn.style.display = 'none';
            }
        }
    });
    
</script> -->

