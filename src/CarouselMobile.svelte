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

    export let perPage: number | { [key: string]: number } = 1;
    export let loop: boolean = true;
    export let autoplay: number = 0;
    export let duration: number = 200;
    export let easing: string = "ease-out";
    export let startIndex: number = 0;
    export let draggable: boolean = true;
    export let multipleDrag: boolean = false;
    export let dots: boolean = true;
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
            video: "/static/video1.mp4",
            title: "Title 2 - Spectacular View",
            description:
                "Description 2 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
        {
            src: "/static/img3.jpeg",
            alt: "Image 3",
            video: "/static/video1.mp4",
            title: "Title 3 - Spectacular View",
            description:
                "Description 3 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
        {
            src: "/static/img4.jpeg",
            alt: "Image 4",
            video: "/static/video1.mp4",
            title: "Title 4 - Spectacular View",
            description:
                "Description 4 - Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s.",
        },
        {
            src: "/static/img5.jpeg",
            alt: "Image 5",
            video: "/static/video1.mp4",
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
            perPage:
                window.innerWidth < 768
                    ? 1
                    : typeof perPage === "object"
                      ? perPage
                      : Number(perPage),
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
        console.table(controller);

        if (autoplay) {
            timer = setInterval(right, autoplay);
        }

        window.addEventListener("resize", updatePerPage);

        return () => {
            autoplay && clearInterval(timer);
            controller.destroy();
            window.removeEventListener("resize", updatePerPage);
        };
    });

    function updatePerPage() {
        const newPerPage =
            window.innerWidth < 768
                ? 1
                : typeof perPage === "object"
                  ? perPage
                  : Number(perPage);
        controller.changePerPage(newPerPage);
    }

    function handleChange() {
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

    function isDotActive(currentIndex: number, dotIndex: number): boolean {
        if (currentIndex < 0) currentIndex = pips.length + currentIndex;
        return (
            currentIndex >= dotIndex * currentPerPage &&
            currentIndex < dotIndex * currentPerPage + currentPerPage
        );
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
                on:mouseover={() => hoverCard(index)}
                on:focus={() => hoverCard(index)}
                on:mouseout={resetHover}
                on:blur={resetHover}
            >
                {#if index === hoveredIndex && slide.video}
                    <div class="video-container">
                        <video
                            class="carousel-video"
                            src={slide.video}
                            autoplay
                            loop
                            muted
                        ></video>
                    </div>
                {:else}
                    <img
                        src={slide.src}
                        alt={slide.alt}
                        class="carousel-image"
                    />
                {/if}
                <div class="info">
                    <h3>{slide.title}</h3>
                    <p>{slide.description}</p>
                </div>
                <button class="lmbutton">Learn More <i class="fa-regular fa-hand"></i></button>
            </div>
        {/each}
    </div>
    {#if dots}
        <ul>
            {#each { length: totalDots } as _, i}
                <li
                    on:click={() => go(i * currentPerPage)}
                    class={isDotActive(currentIndex, i) ? "active" : ""}
                ></li>
            {/each}
        </ul>
    {/if}
</div>

<style>
    .carousel {
        position: relative;
        width: 100%;
        justify-content: center;
        align-items: center;
        overflow: hidden;
        height: 100%;
    }

    .slides {
        transition: transform 0.5s ease;
        height: 100%;
        overflow: hidden;
        position: relative;
    }

    .carousel-card {
        position: relative;
        overflow: hidden;
        margin: 10px;
        height: 80%;
        background-color: #fff;
        border-radius: 15px;
        flex: 0 0 80%;
    }

    .carousel-image {
        width: 100%;
        height: 80%;
        object-fit: cover;
        border-radius: 15px;
    }

    .video-container {
        width: 100%;
        height: 100%;
        border-radius: 15px;
    }

    .carousel-video {
        height: auto;
        max-height: 60%;
        object-fit: cover;
        border-radius: 15px;
        width: 100%;
    }

    .carousel-item {
        flex: 1;
        position: relative;
        overflow: hidden;
        height: 100%;
        transition:
            transform 0.3s ease,
            width 0.3s ease;
    }

    .info {
        position: relative;
        background: #fff;
        color: #000;
        padding: 10px;
        border-radius: 15px;
        text-align: center;
    }

    .lmbutton {
        display: block;
        margin: 10px auto 0;
        padding: 10px 20px;
        font-size: 16px;
        color: #4d0bbf;
        background-color: #fcfc70;
        border: none;
        cursor: pointer;
        border-radius: 15px;
        width: fit-content;
    }

    .lmbutton:focus {
        outline: none;
    }

    ul {
        list-style-type: none;
        position: absolute;
        display: flex;
        justify-content: center;
        width: 100%;
        margin-top: -30px;
        padding: 0;
    }

    ul li {
        margin: 6px;
        border-radius: 100%;
        background-color: #8b8888;
        height: 8px;
        width: 8px;
    }

    ul li:hover {
        background-color: rgba(81, 75, 75, 0.85);
    }

    ul li.active {
        background-color: rgba(81, 75, 75, 0.85);
    }

    @media (max-width: 767px) {
        .carousel {
            height: 100%;
        }

        .slides {
            height: 100%;
        }

        .carousel-item {
            height: auto;
        }

        .info {
            position: static;
            padding: 10px;
            text-align: center;
            border-radius: 0 0 15px 15px;
        }

        .carousel-video {
            width: 100%;
            height: auto;
            display: block;
            max-height: 60vh;
            border-radius: 15px;
        }

        .video-container {
            height: auto;
            max-height: 60vh;
        }

        .carousel-card:active .carousel-video {
            display: block;
        }

        .lmbutton {
            position: static;
        }
    }
</style>
