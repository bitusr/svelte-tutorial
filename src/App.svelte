<script>
	import { onMount } from 'svelte'
	import Welcome from './screens/Welcome.svelte'
	import Game from './screens/Game.svelte'
	import { select } from './select'

	let celebsPromise
	let state = 'welcome' // or playing
	let selection

	const start = async (e) => {
		const { celebs, lookup } = await celebsPromise
		selection = select(celebs, lookup, e.detail.category.slug)
		state = 'playing'
	}

	const loadCelebs = async () => {
		const resp = await fetch('https://cameo-explorer.netlify.app/celebs.json')
		const data = await resp.json()

		const lookup = new Map()

		data.forEach((c) => {
			lookup.set(c.id, c)
		})

		const subset = new Set()
		data.forEach((c) => {
			if (c.reviews >= 50) {
				subset.add(c)
				c.similar.forEach((id) => {
					subset.add(lookup.get(id))
				})
			}
		})

		return {
			celebs: Array.from(subset),
			lookup
		}
	}

	onMount(() => {
		 celebsPromise = loadCelebs()
	})
</script>

<main>
	{#if state === 'welcome'}
		<Welcome on:select={start}/>
	{:else if state === 'playing'}
		<Game {selection} on:restart={() => state = 'welcome'}/>
	{/if}
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		justify-content: center;
		max-width: 800px;
		height: 100%;
		padding: 1em;
		margin: 0 auto;
		text-align: center;
	}
</style>