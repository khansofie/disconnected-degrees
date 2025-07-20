<!---<h1>Welcome to SvelteKit</h1>-->
<!--<p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p>-->

<script>
    import {onMount} from 'svelte';
    import * as d3 from 'd3';
    import {fly} from 'svelte/transition';

    let data = [];
    let currentStep = 0;
    let steps = [0,1,2];

    onMount(async () => {
        const raw = await d3.csv('/data/edu_02.csv');
        data = raw.map(d => ({
            school: d.school,
            dropout_rate: +d.dropout_rate
        }));
    });

    function handleScroll(e) {
        const stepElements = document.querySelectorAll('.step');
        const observer = new IntersectionObserver(
            (entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        currentStep = +entry.target.dataset.step;
                    }
                });
            },
            {threshold: 0.5}
        );
        stepElements.forEach(el => observer.observe(el));
    }
</script>

<svelte:window on:load={handleScroll} />

<style>
    .scroller {
        display:flex;
        flex-direction: row;
    }
    .text {
        flex: 1;
        padding: 2rem;
    }
    .graphic {
        flex: 1;
        position: sticky;
        top: 10vh;
        height: 80vh;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .step {
        margin: 8rem 0;
        font-size: 1.5rem;
    }
    .person {
        width: 60px;
        height: 120px;
        background-color: #f57c00;
        border-radius: 30px 30px 10px 10px;
        margin: 4px;
        display: inline-block;
        transform: scale(1);
        transition: transform 0.3s ease-in-out;
    }

</style>

<main class="scroller">
    <section class="text">
        <div class="step" data-step="0">
            <p>Each figure here represents a number of students</p>
        </div>
        <div class="step" data-step="1">
            <p>Some of them don't graduate. Let's look at that.</p>
        </div>
        <div class="step" data-step="2">
            <p>These are the dropout rates by school.</p>
        </div>
    </section>

    <section class="graphic">
        {#if currentStep === 0}
        <div>
            {#each Array(20) as _, i}
                <!-- svelte-ignore element_invalid_self_closing_tag -->
                <div class="person" in:fly={{y: 50, duration: 300 + i *20}}/>
            {/each}
        </div>
        {:else if currentStep === 1}
        <div>
            {#each Array(20) as _, i}
            <div 
                class="person"
                 style="background-color: {i < 5 ? '#999' : '#f57c00'}"
                 in:fly={{y:50,duration:300 + i*20}}
            ></div>
            {/each}
        </div>
        {:else if currentStep === 2 && data.length}
        <svg width="400" height="300">
            {#each data as d, i}
            <rect
            x = {i*60+20}
            y = {300-d.dropout_rate*10}
            width="40"
            height={d.dropout_rate*10}
            fill="#d32f2f"
            />
            {/each}
        </svg>
        {/if}
    </section>
</main>