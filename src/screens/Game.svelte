<script>
	import { createEventDispatcher } from 'svelte'
	import { fly, crossfade } from 'svelte/transition'
	import * as eases from 'svelte/easing'
	import Card from '../components/Card.svelte'
	import { sleep, pick_random, loadImage } from '../utils'

	export let selection

	const dispatch = createEventDispatcher()

	const [send, receive] = crossfade({
		easing: eases.cubicOut,
		duration: 300,
	})

	const loadDetails = async (celeb) => {
		const resp = await fetch(`https://cameo-explorer.netlify.app/celebs/${celeb.id}.json`)
		const details = await resp.json()
		await loadImage(details.image)
		return details
	}

	const promises = selection.map(round => Promise.all([
		loadDetails(round.a),
		loadDetails(round.b),
	]))

	const results = Array(selection.length)

	let i = 0
	let last_result
	let done = false
	let ready = true

	$: score = results.filter(x => x === 'right').length

	const pickMessage = p => {
		if (p < 0.5) {
			return pick_random([
				'Ouch',
				`That wasn't very good`,
				'Must try harder',
			])
		}
		if (p < 0.8) {
			return pick_random([
				'Not bad',
				'Keep practicing',
			])
		}
		if (p < 1) {
			return pick_random([
				'So close',
				'Almost there',
			])
		}
		return pick_random(['You rock', 'Flawless victory'])
	}

	const submit = async (a, b, sign) => {
		last_result = Math.sign(a.price - b.price) === sign
			? 'right'
			: 'wrong'

		await sleep(1500)

		results[i] = last_result
		last_result = null

		await sleep(500)

		if (i < selection.length - 1) {
			i += 1
		} else {
			done = true
			// TODO end the game
		}
	}
</script>

<header>
	<p>Tap on the more monetisable celebrity's face, or tap 'same price' if society values them equally.</p>
</header>

<div class="game-container">
	{#if done}
		<div class="done">
			<strong>{score}/{results.length}</strong>
			<p>{pickMessage(score / results.length)}</p>
			<button on:click={() => dispatch('restart')}>Back to main screen</button>
		</div>
	{:else if ready}
		{#await promises[i] then [a, b]}
			<div
				class="game"
				in:fly={{duration: 200, y: 20}}
				out:fly={{duration: 200, y: -20}}
				on:outrostart={() => ready = false}
				on:outroend={() => ready = true}
			>
				<div class="card-container">
					<Card
						celeb={a}
						on:select={() => submit(a, b, 1)}
						showPrice={!!last_result}
						winner={a.price >= b.price}
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
						showPrice={!!last_result}
						winner={b.price >= a.price}
					/>
				</div>
			</div>
		{:catch}
			<p class="error">Failed to load data</p>
		{/await}
	{/if}
</div>

{#if last_result}
	<img
		class="giant-result"
		src="/icons/{last_result}.svg"
		alt="{last_result} answer"
		in:fly={{x: 100, duration: 200}}
		out:send={{key: i}}
	>
{/if}

<div
	class="results"
	style="grid-template-columns: repeat({results.length}, 1fr)"
>
	{#each results as result, j}
		<span class="result">
			{#if result}
				<img
					src="/icons/{result}.svg"
					alt="{result} answer"
					in:receive={{key: j}}
				>
			{/if}
		</span>
	{/each}
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

	.results {
        display: grid;
        grid-gap: 0.2em;
        width: 100%;
        max-width: 320px;
        margin: 1em auto 0 auto;
	}

	.result {
        background: rgba(255,255,255,0.1);
        border-radius: 50%;
        padding: 0 0 100% 0;
        transition: background 0.2s;
        transition-delay: 0.2s;
	}

	.result img {
        position: absolute;
        width: 100%;
        height: 100%;
        left: 0;
        top: 0;
	}

	.done {
		position: absolute;
		top: 0;
		left: 0;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: 100%;
		height: 100%;
	}

	.done strong {
		font-size: 6em;
		font-weight: 700;
	}

    @media (min-width: 640px) {
        .game {
			max-width: 100%;
			grid-template-rows: none;
			grid-template-columns: 1fr 8em 1fr;
		}

		.same {
			height: 8em;
		}
    }
</style>
