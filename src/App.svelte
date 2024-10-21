<script lang="ts">
	import { onMount } from 'svelte';
	import Carousel from './Carousel.svelte';
	import CarouselMobile from './CarouselMobile.svelte';
	let isMobile: boolean = false;
  
	const checkMobile = () => {
	  isMobile = window.innerWidth < 768;
	};
  
	onMount(() => {
	  checkMobile();
	  window.addEventListener('resize', checkMobile);
	  
	  return () => {
		window.removeEventListener('resize', checkMobile);
	  };
	});
  </script>
  
  {#if isMobile}
	<CarouselMobile autoplay={2000} />
  {:else}
	<Carousel autoplay={2000} />
  {/if}
  