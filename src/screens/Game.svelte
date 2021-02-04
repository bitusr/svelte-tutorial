<script>
	import Card from '../components/Card.svelte'
	import { sleep } from '../utils'

	export let selection

	const loadDetails = async (celeb) => {
		const resp = await fetch(`https://cameo-explorer.netlify.app/celebs/${celeb.id}.json`)
		return await resp.json()
	}

	const promises = selection.map(round => Promise.all([
		loadDetails(round.a),
		loadDetails(round.b),
	]))

	let i = 0
	let last_result

	const submit = async (a, b, sign) => {
		last_result = Math.sign(a.price - b.price) === sign
			? 'right'
			: 'wrong'

		await sleep(1500)

		last_result = null

		if (i < selection.length - 1) {
			i += 1
		} else {
			// TODO end the game
		}
	}
</script>

<header>
	<p>Tap on the more monetisable celebrity's face, or tap 'same price' if society values them equally.</p>
</header>

<div class="game-container">
	{#await promises[i] then [a, b]}
		<div class="game">
			<div class="card-container">
				<Card
					celeb={a}
					on:select={() => submit(a, b, 1)}
				/>
			</div>

			<div>
				<button class="same" on:click={submit(a, b, 0)}>
					same price
				</button>
			</div>

			<div class="card-container">
				<Card
					celeb={b}
					on:select={() => submit(a, b, -1)}
				/>
			</div>
		</div>
	{:catch}
		<p class="error">Failed to load data</p>
	{/await}
</div>

{#if last_result}
	<img
		src="/icons/{last_result}.svg"
		alt="{last_result} answer"
		class="giant-result"
	>
{/if}

<div class="results">
	<p>results</p>
</div>

<style>
	.game-container {
		flex: 1;
	}

	.game {
		display: grid;
		grid-template-rows: 1fr 2em 1fr;
		grid-gap: 0.5em;
		width: 100%;
		max-width: min(100%, 40vh);
		height: 100%;
		margin: 0 auto;
	}

	.game > div {
		display: flex;
		align-items: center;
	}

	.error {
		color: red;
	}

	.giant-result {
		position: fixed;
		top: calc(50vh - 25vmin);
		left: calc(50vw - 25vmin);
		width: 50vmin;
		height: 50vmin;
		opacity: 0.5;
	}

    @media (min-width: 640px) {
        .game {
			max-width: 100%;
			grid-template-rows: none;
			grid-template-columns: 1fr 8em 1fr;
			/* Safari bug */
			/*max-height: calc(100vh - 60em);*/
		}

		.same {
			height: 8em;
		}
    }
</style>
