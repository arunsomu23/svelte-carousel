<script lang="ts">
    import Siema from "siema";
    import { onMount, onDestroy, createEventDispatcher } from "svelte";

    interface Slide {
        src: string;
        alt: string;
        video: string;
        title: string;
        description: string;
    }

    export let perPage: number | { [key: string]: number } = 4;
    export let loop: boolean = true;
    export let autoplay: number = 0;
    export let duration: number = 200;
    export let easing: string = "ease-out";
    export let startIndex: number = 0;
    export let draggable: boolean = true;
    export let multipleDrag: boolean = false;
    export let controls: boolean = true;
    export let threshold: number = 20;
    export let rtl: boolean = false;

    let currentIndex: number = startIndex;
    let siema: HTMLDivElement;
    let controller: Siema;
    let timer: number | undefined;
    const dispatch = createEventDispatcher();
    let hoveredIndex: number | null = null;

    let slides: Slide[] = [
        {
            src: "/static/img1.jpeg",
            alt: "Image 1",
            video: "/static/video1.mp4",
            title: "Title 1 - Spectacular View",
            description:
                "Description 1 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
        {
            src: "/static/img2.jpeg",
            alt: "Image 2",
            video: "/static/video2.mp4",
            title: "Title 2 - Spectacular View",
            description:
                "Description 2 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
        {
            src: "/static/img3.jpeg",
            alt: "Image 3",
            video: "/static/video3.mp4",
            title: "Title 3 - Spectacular View",
            description:
                "Description 3 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
        {
            src: "/static/img4.jpeg",
            alt: "Image 4",
            video: "/static/video4.mp4",
            title: "Title 4 - Spectacular View",
            description:
                "Description 4 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
        {
            src: "/static/img5.jpeg",
            alt: "Image 5",
            video: "/static/video5.mp4",
            title: "Title 5 - Spectacular View",
            description:
                "Description 5 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
    ];

    $: pips = controller ? controller.innerElements : [];
    $: currentPerPage = controller ? controller.perPage : perPage;
    $: totalDots = controller
        ? Math.ceil(controller.innerElements.length / currentPerPage)
        : [];

    onMount(() => {
        controller = new Siema({
            selector: siema,
            perPage: typeof perPage === "object" ? perPage : Number(perPage),
            loop,
            duration,
            easing,
            startIndex,
            draggable,
            multipleDrag,
            threshold,
            rtl,
            onChange: handleChange,
        });

        if (autoplay) {
            timer = setInterval(right, autoplay);
        }

        return () => {
            autoplay && clearInterval(timer);
            controller.destroy();
        };
    });

    function handleChange(event: Event) {
        currentIndex = controller.currentSlide;
        dispatch("change", {
            currentSlide: controller.currentSlide,
            slideCount: controller.innerElements.length,
        });
    }

    function hoverCard(index: number) {
        pause();
        hoveredIndex = index;
    }

    function resetHover() {
        resume();
        hoveredIndex = null;
    }

    function left() {
        controller.prev();
    }

    function right() {
        controller.next();
    }

    function go(index: number) {
        controller.goTo(index);
    }

    function pause() {
        clearInterval(timer);
    }

    function resume() {
        if (autoplay) {
            timer = setInterval(right, autoplay);
        }
    }
</script>

<div class="carousel">
    <div class="slides" bind:this={siema}>
        {#each slides as slide, index (slide.src)}
            <div
                class="carousel-card carousel-item"
                class:selected={index === hoveredIndex}
                class:hovered={index === hoveredIndex}
                on:mouseenter={() => hoverCard(index)}
                on:mouseleave={resetHover}
            >
                {#if index === hoveredIndex && slide.video}
                    <video
                        class="carousel-video"
                        src={slide.video}
                        autoplay
                        loop
                        muted
                    ></video>
                {:else}
                    <img
                        src={slide.src}
                        alt={slide.alt}
                        class="carousel-image"
                    />
                {/if}
                {#if index === hoveredIndex}
                    <div
                        class="info {index === hoveredIndex
                            ? 'visible'
                            : ''} hover-content">
                        <h3>{slide.title}</h3>
                        <p>{slide.description}</p>
                        <button class="lmbutton">Learn More <i class="fa-regular fa-hand"></i></button>
                    </div>
                {/if}
            </div>
        {/each}
    </div>
    {#if controls}
        <button class="custom-left-button" on:click={left} aria-label="left">
            <i class="fas fa-angle-double-left"></i>
        </button>

        <button class="custom-right-button" on:click={right} aria-label="right">
            <i class="fas fa-angle-double-right"></i>
        </button>
    {/if}
</div>

<style>
    .carousel {
        position: relative;
        width: 100%;
        justify-content: center;
        align-items: center;
        overflow: hidden;
    }

    .slides {
        transition: transform 0.5s ease;
    }

    .carousel-card {
        position: relative;
        overflow: hidden;
        margin: 10px;
        height: auto;
        background-color: #ccc;
        border-radius: 15px;
    }

    .hover-content {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(0, 0, 0, 0.5);
        color: #fff;
        opacity: 0;
        transition: opacity 0.3s ease;
        pointer-events: none;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .carousel-card:hover .hover-content {
        opacity: 1;
    }

    .hover-content h3 {
        font-size: 20px;
        margin-bottom: 10px;
        transform: translateY(-20px);
        opacity: 0;
        transition:
            transform 0.3s ease,
            opacity 0.3s ease;
    }

    .carousel-card:hover .hover-content h3 {
        transform: translateY(0);
        opacity: 1;
    }

    .hover-content p {
        font-size: 14px;
        margin-bottom: 10px;
        opacity: 0;
        transition: opacity 0.3s ease;
    }

    .carousel-card:hover .hover-content p {
        opacity: 1;
    }

    .hover-content button {
        padding: 10px 20px;
        font-size: 16px;
        color: #000;
        background-color: #fff;
        border: none;
        cursor: pointer;
        transform: scale(1);
        transition:
            transform 0.3s ease,
            background-color 0.3s ease;
    }

    .hover-content button:hover {
        transform: scale(1.1);
        background-color: #ddd;
    }

    .carousel-card.selected {
        flex: 1 0 30%;
        transform: scale(1.1);
        z-index: 1;
    }

    .carousel-card.hovered {
        border-radius: 30px;
    }

    .carousel-image {
        width: 100%;
        height: 100%;
        object-fit: cover;
        border-radius: 15px;
    }

    .carousel-video {
        width: 100%;
        height: 100%;
        object-fit: cover;
        border-radius: 15px;
    }

    .carousel-item {
        flex: 1;
        position: relative;
        overflow: hidden;
        height: 470px;
        transition:
            transform 0.3s ease,
            width 0.3s ease;
    }

    .info {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        background: rgba(0, 0, 0, 0.5);
        color: #fff;
        padding: 10px;
        opacity: 0;
        transition: opacity 0.3s ease;
        border-radius: 15px;
    }

    .info.visible {
        opacity: 1;
    }

    button {
        position: absolute;
        width: 40px;
        height: 40px;
        top: 50%;
        transform: translateY(-50%);
        border: none;
        background-color: transparent;
        color: #000;
        font-size: 24px;
        cursor: pointer;
    }

    button:focus {
        outline: none;
    }

    .lmbutton {
        position: absolute;
        width: 150px;
        height: 40px;
        top: 80%;
        transform: translateY(-50%);
        border: none;
        background-color: transparent;
        color: #2b14f8 !important;
        font-size: 24px;
        cursor: pointer;
        white-space: nowrap;
    }

    .lmbutton:focus {
        outline: none;
    }

    .custom-left-button {
        position: absolute;
        left: 11px;
        width: 60px;
        height: 75px;
        background-color: rgba(0, 0, 0, 0.7);
        color: #fff;
        font-size: 24px;
        cursor: pointer;
        border-radius: 50%;
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 0 10rem 10rem 0;
    }

    .custom-right-button {
        position: absolute;
        right: 11px;
        width: 60px;
        height: 75px;
        background-color: rgba(0, 0, 0, 0.7);
        color: #fff;
        font-size: 24px;
        cursor: pointer;
        border-radius: 50%;
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 10rem 0 0 10rem;
    }

    button:focus {
        outline: none;
    }

    button i {
        pointer-events: none;
    }
</style>
