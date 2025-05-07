<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/state';
	import PizzaFall from '$lib/PizzaFall.svelte';
	import { calculatePizzas, pizza } from '$lib/pizza';
	import { onMount } from 'svelte';

	const COUNT_PARAM = 'count';
	const MAX = 1000;

	let countParam = Number(page.url.searchParams.get(COUNT_PARAM));
	let count = $state(!isNaN(countParam) ? countParam : 0);
	let inputRef = $state<HTMLInputElement | null>(null);

	$effect(() => {
		const params = new URLSearchParams(page.url.search);

		/**
		 * Update the count param in the URL when the count changes.
		 * Remove the param if the count is 0.
		 */
		if (count) params.set(COUNT_PARAM, count.toString());
		else params.delete(COUNT_PARAM);

		if (params.toString() !== page.url.searchParams.toString()) {
			goto(`${page.url.pathname}?${params.toString()}`, { replaceState: true, keepFocus: true });
		}
	});

	$effect(() => {
		if (count && count > MAX) {
			count = MAX;
		}

		if (count && count < 0) {
			count = 0;
		}
	});

	onMount(() => {
		const handleCmdK = (e: KeyboardEvent) => {
			if (e.key === 'k' && (e.metaKey || e.ctrlKey)) {
				e.preventDefault();
				inputRef?.focus();
			}
		};

		window.addEventListener('keydown', handleCmdK);

		return () => {
			window.removeEventListener('keydown', handleCmdK);
		};
	});

	onMount(() => {
		/**
		 * Console log the pizza ascii art on mount
		 */
		console.log(pizza);
	});
</script>

<div class="bg-background min-h-screen w-full">
	<PizzaFall />

	<main class="py-24 max-w-[648px] w-full text-text z-30 mx-auto space-y-12 px-4">
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
				bind:this={inputRef}
				bind:value={count}
				placeholder="Antall personer"
			/>

			<div class="text-2xl">
				{#if count && count > 0}
					{@const pizzas = calculatePizzas(count)}
					<p>
						{#each { length: count } as _}
							<span>🍕</span>
						{/each}
						<span>=</span> <span>{pizzas}</span>
					</p>
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
