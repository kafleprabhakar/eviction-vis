<script>

    import { style } from "d3";

</script>

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hover Effect</title>
<style>
    html, body {
        height: 100%;
        overflow: auto; /* Allow scrolling */
    }
    header{
        text-align: center;
        padding: 20px 0;
    }
    header h1{
        color: #ED5701;
    }
    div.container {
        position: absolute;
        display: inline-block;
        opacity: 0; /* Start with opacity 0 */
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
        font-family: Arial, sans-serif; 
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
        position: absolute;
        top: 650px;
        left: 75px;
        width: 500px; /* Adjust the width as needed */
        height: 360; /* Make the block take up the full height of the viewport */
        background-color: #072B69;
        color: white;
        padding: 20px;
        text-align: center;
        box-sizing: border-box; /* Include padding in the width */
        font-size: 25px;
        font-weight: bold; /* Make the text bold */
        font-family: Arial, sans-serif; 
        line-height: 1.5; /* Increase line height */
        opacity: 0; /* Start with opacity 0 */
        transition: opacity 0.5s ease;
        
    }
    .mapimage{
        position: absolute;
        width: 400px;
        height: 300px;
        border: 1px solid #ccc;
        top: 1000px;
        left: 800px;
        opacity: 0; /* Start with opacity 0 */
        transition: opacity 0.5s ease;


    }
    .right-block {
        position: absolute;
        top: 1400px;
        left: 50px;
        width: 400px; /* Adjust the width as needed */
        height: 360; /* Make the block take up the full height of the viewport */
        background-color: #072B69;
        color: white;
        padding: 20px;
        text-align: center;
        box-sizing: border-box; /* Include padding in the width */
        font-size: 30px;
        font-weight: bold; /* Make the text bold */
        font-family: Arial, sans-serif; 
        line-height: 2; /* Increase line height */
        opacity: 0; /* Start with opacity 0 */
        transition: opacity 0.5s ease;
        
    }
    /* .scrollhold{
        position:absolute;
        top: 1400px;
        left:1px;
        
    } */
    .info {
        position: absolute;
        width: 800px;
        left: 300px;
        opacity: 1; /* Start with opacity 1 */
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
</style>
</head>
<body>
<header>
    <h1>Evictions in Boston</h1>
</header>
<div class="info">
    <p>
        Boston is one of the most expensive places to live in the United States. With the second highest average monthly rent index in the entire country, a third of renters pay more than 30 percent of their income on housing costs. Any change in income including rent increase, a medical bill or a loss of a job can cause a low-income person to fall behind on payment. Those who can no longer keep up with rent are physically removed or forced to leave their own homes. Hover over the orange icons to hear their stories.
    </p>
</div>
<!-- <div class = "scrollhold">
    <p>
        hi
    </p>
</div> -->
<div class="left-block">
    <p>Between 2010 and 2019, property owners filed more than 50,000 evictions in Boston Housing Court</p>
</div>

<!-- map -->
<img class ="mapimage" src="images/bostonMap.png" style = "width: 40%" alt="Boston Map">

<!-- All blanks -->
<div class="container" style="top: 605px; left: 1225px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 820px; left: 1300px;" >
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px">
</div>
<div class="container" style="top: 620px; left: 830px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 830px; left: 730px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 1330px; left: 1230px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 1600px; left: 530px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 950px; left: 200px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 1280px; left: 600px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 1150px; left: 300px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>
<div class="container" style="top: 1500px; left: 1050px;">
    <img class="image" src="images/clear.png" alt="Image" style="width: 50px;">
</div>

<div class="container" style="top: 750px; left: 1050px;">
    <img class="image" src="images/orange.png" alt="Image" style="width: 50px;">
    <div class="text" style="right: calc(100% + 10px); width: 400px;">
        <div class="text-container">
            <img src="images/domingo.png" alt="Tooltip Image" style="width: 200px;">
            <p>"Fannie Mae took over my [bank] account, and sold my house to investors, who sold it to a private company. Now, alone, I’m facing eviction. This ordeal has made me suffer more than you can possibly imagine."</p>
        </div>
    </div>
</div>

<div class="container" style="top: 1000px; left: 450px;">
    <img class="image" src="images/orange.png" alt="Image" style="width: 50px;">
    <div class="text" style="left: calc(100% + 10px); width: 400px;">
        <div class="text-container">
            <img src="images/margarita.png" alt="Tooltip Image" style="width: 200px;">
            <p>"The first time I was evicted, I was given only a week to leave. I am no young lady, but I could not afford anything but to do the moving alone. I worried about the safety of my grandson, who lived with me."</p>
        </div>
    </div>
</div>

<div class="container" style="top: 1400px; left: 850px;">
    <img class="image" src="images/orange.png" alt="Image" style="width: 50px;">
    <div class="text" style="right: calc(100% + 10px); width: 400px;">
        <div class="text-container">
            <img src="images/dora.png" alt="Tooltip Image" style="width: 200px;">
            <p>"A little over a year ago, all six families living in my building were given eviction notices. We were given 30 days to leave. I felt lost and scared."</p>
        </div>
    </div>
</div>

<div class = "right-block">
    <p>SINCE @xxxxxx HAVE BEEN AFFECTED</p>
</div>
<button id="returnToTopBtn">Return to Top</button>

 

<!-- Add more container and image pairs as needed -->

<script>


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


    
</script>


</body>
</html>