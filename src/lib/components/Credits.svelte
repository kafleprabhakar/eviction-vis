<div class='credits-container'>

    <div class='authors'> Prabhakar Kafle, Madeline Leano, Jessica Xu, Shelley Zheng | Spring 2024</div>

    <div class='acknowledgements'>
        <h3 class='credit-header'>Acknowledgements</h3>
        <p class='credit-words'>This project was developed with guidance and feedback from the <a class='credit-link' href='https://www.mapc.org/' target='_blank'>Metropolitan Area Plannning Commission (MAPC)</a></p>
        <p class='credit-words'>We would like to thank the MAPC and teaching staff of 6.C35/6.C85 for all their support this semester</p>
    </div>

    <div class='references'>
        <h3 class='credit-header'>References</h3>
        <div class='reference-links'>
            <div class='reference-cluster'>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://www.kaggle.com/datasets/yingzhou474/boston-neighborhoods-geojson'>Boston Neighborhoods</a></p>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://data.boston.gov/dataset/census-2020-block-group-neighborhoods'>Boston Neighboohood Boundaries Data</a></p>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://www.bostonevictions.org/data'>Boston Evictions</a></p>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://data.boston.gov/dataset/property-assessment'>Boston Property Assessment</a></p>
            </div>

            <div class='reference-cluster'>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://data.boston.gov/dataset/2020-census-for-boston#:~:text=In%20the%202020%20Census%2C%20the,tracts%20to%20approximate%20Boston%20neighborhoods.'>Boston 2020 Census</a></p>           
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://data.boston.gov/dataset/property-assessment'>Corporate Ownership Data</a></p>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://www.masscourts.org/eservices/home.page.2'>Court Cases Data</a></p>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://www.urbandisplacement.org/maps/housing-precarity-risk-model/'>Eviction Lab</a></p>
                <p class='credit-words'><a class='credit-link ref' target='_blank' href='https://justshelter.org/stories/#stories-image-1'>Testimonials</a></p>               
            </div>
        </div>

    </div>

</div>

<style>
    .ref {
        padding-left: 1.5rem;
        padding-right: 1.5rem;
    }
    .reference-links {
        display: flex;
        align-items: center;
        flex-direction: column;
    }
    .reference-cluster{
        display: flex;
    }
    .credits-container {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .authors {
        margin-top: 3rem;
        font-size: 1.2rem;
    }

    .acknowledgements {
        text-align: center;
    }

    .references {
        text-align: center;
        margin-bottom: 5rem;
    }

    .credit-header {
        margin-top: 2rem;
        margin-bottom: 0rem;
        color: #072B69;
    }

    .credit-words{
        font-size: 1.3rem;
        margin-top:0.5rem;
        margin-bottom:0rem;
    }

    .credit-link {
        text-decoration: none;
        color: #1658d2;
    }

    .credit-link:hover {
        cursor: pointer;
        text-decoration: underline;
        /* color: #ED5701; */
    }
</style>