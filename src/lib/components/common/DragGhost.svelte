<script lang="ts">
	import { onDestroy, onMount } from 'svelte';

	export let x;
	export let y;

	let popupElement = null;

	// Use transform instead of top/left for better performance
	$: transformStyle = `transform: translate3d(${x + 10}px, ${y + 10}px, 0)`;

	onMount(() => {
		document.body.appendChild(popupElement);
		// Don't set overflow: hidden as it's not necessary and causes layout shifts
	});

	onDestroy(() => {
		if (popupElement && popupElement.parentNode) {
			document.body.removeChild(popupElement);
		}
	});
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->

<div
	bind:this={popupElement}
	class="fixed top-0 left-0 w-screen h-[100dvh] z-50 touch-none pointer-events-none"
>
	<div class="absolute text-white z-99999 will-change-transform" style={transformStyle}>
		<slot></slot>
	</div>
</div>
