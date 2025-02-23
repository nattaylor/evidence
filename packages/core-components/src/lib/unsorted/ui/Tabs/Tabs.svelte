<script context="module">
	export const evidenceInclude = true;
</script>

<script>
	import { onMount, setContext } from 'svelte';
	import { writable } from 'svelte/store';

	const classes = {
		notActive: 'text-gray-600 hover:text-gray-800 hover:bg-gray-200',
		active: 'text-black border-b-2 border-[--borderColor] bg-[--bgColor]'
	};

	/**
	 * id can be provided to enable tab selection to persist across reloads (e.g. with query params)
	 * @type {string}
	 */
	export let id;

	/**
	 * color can be provided to set custom background color for active tab.
	 * @type {string}
	 */
	export let color = 'hsla(207, 65%, 39%, 1)';
	color = color.replace(/\s+/g, ''); // clean string

	const bgColor = isValidColorString(color) ? addOpacityToColor(color) : 'hsla(207, 65%, 39%, 0.1)';
	const borderColor = isValidColorString(color) ? color : 'hsla(207, 65%, 39%, 1)';

	function isHex(inputColor) {
		const hexRegex = /^#([0-9A-Fa-f]{3}|[0-9A-Fa-f]{6})$/i;
		return hexRegex.test(inputColor);
	}

	function isRGB(inputColor) {
		const rgbRegex = /^rgb\(\s*(\d{1,3})\s*,\s*(\d{1,3})\s*,\s*(\d{1,3})\s*\)$/i;
		return rgbRegex.test(inputColor);
	}

	function isHSL(inputColor) {
		const hslRegex = /^hsl\(\s*(\d{1,3})\s*,\s*(\d{1,3}%)\s*,\s*(\d{1,3}%)\s*\)$/i;
		return hslRegex.test(inputColor);
	}

	function isValidColorString(inputColor) {
		return isHex(inputColor) || isRGB(inputColor) || isHSL(inputColor);
	}

	function addOpacityToColor(colorString) {
		if (isHex(colorString)) {
			return colorString + '1a';
		} else if (isRGB(colorString) || isHSL(colorString)) {
			return colorString.replace(/(\)|\s|$)/, ', 0.1$1');
		}
	}

	/**
	 * @type {import("svelte/store").Writable<{ tabs: {label: string, id: string}[], active: string, tabsId: string}>}
	 */
	const tabItems = writable({ tabs: [], active: null });

	onMount(() => {
		const url = new URL(window.location.href);
		const urlActive = url.searchParams.get(id);
		if (urlActive) {
			$tabItems.active = urlActive;
		}
	});

	$: if (!$tabItems.active && $tabItems.tabs.length)
		// Select the first tab by default
		$tabItems.active = $tabItems.tabs[0].id;

	$: if ($tabItems.active && id) {
		// Keep the Query in sync
		const url = new URL(window.location.href);
		url.searchParams.set(id, $tabItems.active);
		history.replaceState({}, '', url);
	}

	setContext('TAB_REGISTRATION', tabItems);
</script>

<section>
	<nav class="my-6 flex flex-wrap gap-x-4 gap-y-1">
		{#each $tabItems.tabs as tab}
			<button
				style:--bgColor={bgColor}
				style:--borderColor={borderColor}
				on:click={() => ($tabItems.active = tab.id)}
				class="mt-2 p-2 rounded-t flex-1 text-sm font-sans whitespace-nowrap transition duration-200 ease-in active:bg-gray-100 {$tabItems.active ===
				tab.id
					? classes.active
					: classes.notActive} "
			>
				{tab.label}
			</button>
		{/each}
	</nav>
	<div class="text-base">
		<slot />
	</div>
</section>
