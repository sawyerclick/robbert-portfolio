<script context="module">
    // Vanilla lazy loading: https://dev.to/ekafyi/lazy-loading-images-with-vanilla-javascript-2fbj 
    // Svelte lazy loading: https://css-tricks.com/lazy-loading-images-in-svelte/ 
    // MDN responsive images: https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images
    // Sharp documentation: https://sharp.pixelplumbing.com 
    import {client} from '@lib/api/graphql-client'
    import {projectsQuery} from '@lib/api/graphql-queries'

    export const load = async () => {
        
        let width = 1500;
        let height = 1200;
        const variables = {width: width, height: height}

        const {projects} = await client.request(projectsQuery, variables)

        return {
            props: {
                projects,
            }
        }
    }
</script>
<script>
    import Image from '@components/Image/Image.svelte'
    import {inview} from 'svelte-inview'
    import {fade} from 'svelte/transition'
    
    export let projects;
    let height;
    let width;

    let base64pixel = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAJCAQAAACRI2S5AAAAEklEQVR42mNMkmTACxhHFYABAMtfBF25QVgfAAAAAElFTkSuQmCC"

    let isInView;
    const options = {
        rootMargin: '-20%',
        threshold: '0.05',
        unobserveOnEnter: true,
    };

    const handleChange = ({ detail }) => (isInView = detail.inView);
</script>




<div class="grid grid-cols-1 md:grid-cols-2 gap-4 md:gap-8 pb-4">
    {#each projects.slice(0, 2) as project}
        <div bind:offsetHeight={height} bind:offsetWidth={width}>
            <img {height} {width} alt="project img" src="{project.image[0].url}"/>
        </div>
    {/each}
</div>
<div class="grid grid-cols-1 gap-4 md:gap-8">
    {#each projects.slice(2, projects.length) as project}
        <div bind:offsetHeight={height} bind:offsetWidth={width} use:inview="{options}" on:change="{handleChange}">
            {#if isInView}
                <div in:fade="{{duration: 300 }}">
                    <Image {height} {width} alt="project img" src="{project.image[0].url}"/>
                </div>
            {:else}
                <img {height} {width} alt="project img" src="{base64pixel}"/>
            {/if}
        </div>
    {/each}
</div>

<!--Pass width and height as props to Image component. Inside image component, graphQlRequest for that specific image.-->
<!--
    <IntersectionObserver {element} let:intersecting>
            <Image bind:this={element} {height} {width} alt="project img" src="{intersecting ? project.image[0].url : base64pixel}"/>
        </IntersectionObserver>
-->