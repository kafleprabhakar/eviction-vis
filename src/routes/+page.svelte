<script lang="ts">
    import V1 from "$lib/components/V1.svelte";
	import TemporalMap from '$lib/components/TemporalMap.svelte';
	import CorpLineChart from '$lib/components/CorpLineChart.svelte';
    import EvictionJudgement from '$lib/components/EvictionJudgement.svelte';
	import V4 from "$lib/components/V4.svelte";
	import V5 from "$lib/components/V5.svelte";
	import V6 from "$lib/components/V6.svelte";
	import Credits from "$lib/components/Credits.svelte";

	function scrollTo(elementId: string) {
        const element = document.getElementById(elementId);
        if (element) {
            element.scrollIntoView({ behavior: 'smooth' });
        }
    }
</script>
<div class='sidebar'>
    <!-- Sidebar navigation links -->
    <ul class="sidebar-nav">
        <li class='scroll-link'><a on:click={() => scrollTo('v1')}><svg width="100" height="50"><circle class='scroll-circle' cx="20" cy="20" r="10" fill="#363533"></circle></svg></a></li>
        <li class='scroll-link'><a on:click={() => scrollTo('v2')}><svg  width="100" height="50"><circle class='scroll-circle' cx="20" cy="20" r="10" fill="#363533"></circle></svg></a></li>
        <li class='scroll-link'><a on:click={() => scrollTo('v3')}><svg width="100" height="50"><circle class='scroll-circle' cx="20" cy="20" r="10" fill="#363533"></circle></svg></a></li>
        <li class='scroll-link'><a on:click={() => scrollTo('v4')}><svg width="100" height="50"><circle class='scroll-circle' cx="20" cy="20" r="10" fill="#363533"></circle></svg></a></li>
		<li class='scroll-link'><a on:click={() => scrollTo('v5')}><svg width="100" height="50"><circle class='scroll-circle' cx="20" cy="20" r="10" fill="#363533"></circle></svg></a></li>
		<li class='scroll-link'><a on:click={() => scrollTo('v6')}><svg width="100" height="50"><circle class='scroll-circle' cx="20" cy="20" r="10" fill="#363533"></circle></svg></a></li>
	</ul>
</div>

<div class='everything'>
	<!-- VIS 1 -->
	<div class='vis-section' id="v1">
		<header class='header-title'>
			<h1 class='first-title'>EVICTIONS IN BOSTON</h1>
		</header>
		<V1/>		
	</div>


	<!-- VIS 2 -->
	<div class='vis-section' id='v2'>
		<h2 class='header-title'>Eviction Rates Over Time</h2>
		<TemporalMap/>
	</div>


	<div class='vis-spacer1'>
		<h2 class='who'>BUT WHO IS ISSUING THESE EVICTIONS?</h2>
	</div>

	<!-- VIS 3 -->
	<div class='vis-section' id="v3">
		<h2 class='header-title'>Corporate Evictions</h2>
		<CorpLineChart />
		<EvictionJudgement />
	</div>

	<div class='vis-spacer2'>
		<h3 class='toWho'>Who are these Evictions Impacting?</h3>
		<p> Despite Boston's reputation for diversity and its plethora of vibrant
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
			Boston.</p>
	</div>

	<!-- VIS 4 -->
	<div class='vis-section' id="v4">
		<V4/>
	</div>


	<!-- VIS 5 -->
	<div class='vis-section' id="v5">
		<h2 class='header-title5'>What if it was you?</h2>
		<V5/>
	</div>

	<!-- Resources -->
	<div class='vis-section' id="v6">
		<h2 class='header-title5'>What Can We Do?</h2>
		<V6/>
	</div>

</div>

<div class='footer'>
	<Credits/>
</div>

<style>
	.everything {
		display: flex;
		flex-direction: column;
	}

	.scroll-circle:hover {
		fill:#ED5701;
		cursor: pointer;
	}

    .sidebar {
		z-index: 1000;
        position: fixed;
        top: 0;
        left: 0;
        width: 100px;
        height: 100%;
        background-color: none;
		display: flex;
		flex-direction: column;
		justify-content: center;
    }

    .sidebar-nav {
        list-style-type: none;
        padding: 0;
        margin: 0;
    }

    /* .sidebar-nav li {
        margin-bottom: 10px;
    } */

    .sidebar-nav a {
        display: block;
        padding: 10px;
        text-decoration: none;
    }

	.first-title{
		letter-spacing:0.2rem;
	}

	.header-title{
		align-self: center;
	}

	.header-title5{
		align-self: center;
		margin-bottom: 2.5rem;
	}

	#v1 {
		margin-bottom: 15rem;
	}

	#v4 {
		margin-bottom: 7rem;
	}

	#v5 {
		margin-bottom: 8rem;
	}

	#v6 {
		min-height: 90vh;
	}

	.vis-section {
		display: flex;
		flex-direction: column;
		min-height: 100vh;
	}

	.who {
		align-self: center;
		background-color: #feb64a;
		color: #F8F7F5;
		padding: 7rem;
		font-size: 4rem;
		text-align: center;
		margin-top: 11rem;
		margin-bottom: 11rem;
		letter-spacing: .2rem;
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
	}

	.toWho {
		color: #F8F7F5;
		margin: 0.5rem;
		font-size: 3rem;
	}

	.footer {
		background-color: #a9a9a9;
		color: #F8F7F5;
		width:100%;
		position:absolute;
		left:0; 
		overflow-x: hidden;
		display: flex;
		align-items: center;
		justify-content: center;
		outline:none;
	}

</style>
