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
      infoSectionOpacity = scrollPosition > 180 ? 0 : 1;
      leftBlockOpacity = scrollPosition > 180 ? 1 : 0;
      mapImageOpacity = scrollPosition > 180 ? 1 : 0;
      rightBlockOpacity = scrollPosition > 180 ? 1 : 0;
      containersOpacity = scrollPosition > 180 ? 1 : 0;
      returnToTopVisible = scrollPosition > 180;
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
        width: 30px;
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
    .caption{
        display: none;
    } 

    .text-container {
        display: flex;
        align-items: center;
        padding:10px;
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
        margin-bottom: 7rem;
        opacity: var(--infoSectionOpacity); /* Start with opacity 1 */
        transition: opacity 0.3s ease; /* Add transition for opacity */
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
        Boston is one of the most expensive places to live in the United States. With the second highest average monthly rent index in the entire country, a third of renters pay more than 30 percent of their income on housing costs. Any change in income including rent increase, a medical bill or a loss of a job can cause a low-income person to fall behind on payment. Those who can no longer keep up with rent are physically removed or forced to leave their own homes. <b>Hover over the orange icons to hear their stories.</b>
    </p>
</div>

<div class='all-blocks' style="opacity:{leftBlockOpacity}" >

    <div class="left-block" >
        <p>Between 2010 and 2019, property owners filed more than 50,000 evictions in Boston Housing Court</p>
    </div>

    <!-- map -->
    <img class ="mapimage" src="images/bostonMap.png" alt="Boston Map">

    <!-- All blanks -->
    <div class="container" style="top: 570px; left: 1225px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">1</p>
    </div>

    <div class="container" style="top: 800px; left: 1300px;" >
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">2</p>
    </div>
    <div class="container" style="top: 620px; left: 1030px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">3</p>
    </div>
    <div class="container" style="top: 630px; left: 530px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">3.1</p>
    </div>
    <div class="container" style="top: 1050px; left: 620px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">4</p>
    </div>
    <div class="container" style="top: 720px; left: 670px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">5</p>
    </div>
    <div class="container" style="top: 800px; left: 880px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">5.1</p>
    </div>
    <div class="container" style="top: 1290px; left: 1130px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">6</p>
    </div>
    <div class="container" style="top: 1590px; left: 500px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">7</p>
    </div>
    <div class="container" style="top: 900px; left: 250px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">8</p>
    </div>
    <div class="container" style="top: 1000px; left: 380px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">8.1</p>
    </div>
   
    <div class="container" style="top: 1280px; left: 570px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">9</p>
    </div>
    <div class="container" style="top: 1150px; left: 200px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">10</p>
    </div>
    <div class="container" style="top: 1350px; left: 970px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">11</p>
    </div>
    <div class="container" style="top: 1500px; left: 1150px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">12</p>
    </div>
    <div class="container" style="top: 1150px; left: 820px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">13</p>
    </div>
    <div class="container" style="top: 1450px; left: 670px;">
        <img class="image" src="images/clear.png" alt="Person" >
        <p class="caption">14</p>
    </div>

    <div class="container" style="top: 780px; left: 1120px;">
        <img class="image" src="images/orange.png" alt="Orange Person" >
        <p class="caption">O1</p>
        <div class="text" style="right: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <p>"Now I don't know who people think are, like you know, [who] get evicted or face homelessness, but I always paid my rent on time." - Kimberley O’Connor </p>
            </div>
        </div>
    </div>

    <div class="container" style="top: 580px; left: 780px;">
        <img class="image" src="images/orange.png" alt="Orange Person">
        <p class="caption">O2.1</p>
        <div class="text" style="left: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <img src="images/margarita.png" alt="Hover Person" style="width: 200px;">
                <p>"Eviction affected me emotionally and mentally, but it fundamentally took away my sense of a home. I carry that burden with me." - Margarita</p>
            </div>
        </div>
    </div>
    
    <div class="container" style="top: 850px; left: 550px;">
        <img class="image" src="images/orange.png" alt="Orange Person">
        <p class="caption">O2</p>
        <div class="text" style="right: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <p>I was petrified and felt like the biggest delinquent in the world for not being able to pay my rent. - Kettia </p>
            </div>
        </div>
    </div>
    <div class="container" style="top: 1170px; left: 420px;">
        <img class="image" src="images/orange.png" alt="Orange Person">
        <p class="caption">O1.2</p>
        <div class="text" style="left: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <img src="images/dora.png" alt="Hover Person" style="width: 200px;">
                <p>"We are trying to prove that we are honest, hard-working people, who simply can’t afford that kind of a rent increase. I hope things end for the better soon." - Dora</p>
            </div>
        </div>
    </div>
   
    <div class="container" style="top: 950px; left: 100px;">
        <img class="image" src="images/orange.png" alt="Orange Person">
        <p class="caption">O2.2</p>
        <div class="text" style="left: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <p>"It was like the only power and inspiration and the motivation had been taken out of me." - Whitney</p>
            </div>
        </div>
    </div>
    <div class="container" style="top: 920px; left: 770px;">

        <img class="image" src="images/orange.png" alt="Orange Person">
        <p class="caption">O2.3</p>
        <div class="text" style="left: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <img src="images/domingo.png" alt="Hover Person" style="width: 200px;">
                <p>"Now, alone, I’m facing eviction. This ordeal has made me suffer more than you can possibly imagine." - Domingo</p>
            </div>
        </div>
    </div>

    <div class="container" style="top: 1550px; left: 900px;">
        <img class="image" src="images/orange.png" alt="Orange Person" >
        <p class="caption">O3</p>
        <div class="text" style="right: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <p>"My medical conditions make having a house a necessity. The prospect of being homeless terrifies me. I’m afraid I won’t survive homelessness." - Seren</p>
            </div>
        </div>
    </div>
    <div class="container" style="top: 1330px; left: 1330px;">
        <img class="image" src="images/orange.png" alt="Orange Person">
        <p class="caption">O2.1</p>
        <div class="text" style="right: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <p>"We are trying to find a new place to live, but no one will rent to us due to us having rent arrears and a pending eviction. I am not sure what will happen to my family. We have a young son also." - Lisa</p>
            </div>
        </div>
    </div>
    <div class="container" style="top: 1600px; left: 230px;">
        <img class="image" src="images/orange.png" alt="Orange Person">
        <p class="caption">O3.2</p>
        <div class="text" style="left: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <p>I don’t know where to turn for help. I lose sleep every night worrying about what I’m going to do." - Marlaina</p>
            </div>
        </div>
    </div>
    <div class="container" style="top: 1300px; left: 750px;">
        <img class="image" src="images/orange.png" alt="Orange Person" >
        <p class="caption">O3.3</p>
        <div class="text" style="right: calc(100% + 10px); width: 400px;">
            <div class="text-container">
                <p>" If you have been evicted, it is extremely difficult to find housing." - Kelly</p>
            </div>
        </div>
    </div>



    <div class = "left-block">
        <p>In Boston, around 80% of evictions are reportedly filed for non-payment of rent</p>
    </div>

</div>
 
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

