<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import PizzaFall from '$lib/PizzaFall.svelte';
	import { PIZZA, PIZZA_ART } from '$lib/pizza';
	import VirtualList from '@sveltejs/svelte-virtual-list/VirtualList.svelte';

	const COUNT_PARAM = 'count';

	let countParam = Number($page.url.searchParams.get(COUNT_PARAM));
	let count = $state(!isNaN(countParam) ? countParam : 0);

	$effect(() => {
		const params = new URLSearchParams($page.url.search);

		/**
		 * Update the count param in the URL when the count changes
		 *
		 * Remove the param if the count is 0
		 */
		if (count) params.set(COUNT_PARAM, count.toString());
		else params.delete(COUNT_PARAM);

		goto(`?${params.toString()}`, { replaceState: true, noScroll: true, keepFocus: true });
	});

	$effect(() => {
		/**
		 * Console log the pizza ascii art
		 */
		console.log(PIZZA_ART);
	});

	let pizzaWidth = 24;
	let start = $state();
	let end = $state();
	let containerWidth = $state<number>(0);

	const partition = (items: Array<string>, size: number) => {
		const result = [];
		for (let i = 0; i < items.length; i += size) {
			result.push(items.slice(i, i + size));
		}
		return result;
	};

	let items = $derived.by(() => {
		const emojis = Array.from({ length: count })
			.fill(null)
			.map(() => PIZZA);
		const pizzasPerRow = Math.floor(containerWidth / pizzaWidth);
		return partition(emojis, pizzasPerRow);
	});
</script>

<div class="bg-background min-h-screen w-full">
	<PizzaFall />

	<main class="py-24 max-w-[648px] w-full text-text z-30 mx-auto mx-auto space-y-12 px-4">
		<h1 class="text-5xl font-medium font-serif">Pizza-formelen</h1>

		<section class="space-y-4">
			<h2 class="text-3xl font-medium mb-4 font-serif">Formelen</h2>

			<p
				class="font-mono bg-white px-4 py-2 flex items-center justify-start border rounded-lg border-border"
			>
				⌈3 / 8 × antall personer⌉ = antall pizza
			</p>
		</section>

		<section class="space-y-4">
			<h2 class="text-3xl font-medium mb-4 font-serif">Kalkulator</h2>

			<input
				type="number"
				class="border border-gray-300 bg-white px-4 py-2 w-full rounded-lg"
				bind:value={count}
				placeholder="Antall personer"
			/>

			<div bind:clientWidth={containerWidth} class="h-[400px]">
				{#if count > 0}
					<VirtualList {items} bind:start bind:end>
						{#snippet children({ item }: { item: Array<string> })}
							<div class="w-ful h-fit gap-2 flex">
								{#each item as emoji}
									<span style="width: {pizzaWidth}px;">{emoji}</span>
								{/each}
							</div>
						{/snippet}
					</VirtualList>
				{:else}
					<p class="text-muted text-xl">Fyll inn antall personer</p>
				{/if}
			</div>
		</section>
	</main>

	<footer>
		<div class="py-4 text-center">
			<p class="text-sm text-muted hover:text-muted-hover transition-colors">
				<a href="https://echo.uib.no" class="hover:underline">echo.uib.no</a>
			</p>
		</div>
	</footer>
</div>
