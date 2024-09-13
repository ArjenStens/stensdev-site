<script lang="ts">


	import {NPool, NRelay1, NSecSigner} from "@jsr/nostrify__nostrify";

	let errorMessage: string | undefined;
	let successMessage: string | undefined;
	let secretKeyHex: string;
	let repoNaddr: string;

	async function submit() {
		if (isNullOrEmpty(repoNaddr) || isNullOrEmpty(secretKeyHex)) {
			errorMessage = "Please provide a secret key and repo naddr to start watching a repository."
			return;
		}

		try {
			const relays = [
				'wss://nos.lol',
				'wss://relay.damus.io',
				'wss://relay.primal.net',
				'wss://relay.stens.dev',
			];

			const pool = new NPool({
				open(url) {
					return new NRelay1(relays[0]);
				},
				reqRouter: async (filters) => {
					return new Map(relays.map((relay) => {
						return [relay, filters];
					}));
				},
				eventRouter: async event => {
					return relays;
				},
			});

			const signer = new NSecSigner(secretKeyHex);
			const signerPubkey = await signer.getPublicKey();

			var note = {
				kind: 68001,
				pubkey: signerPubkey,
				content: `Sent trough StensDev at ${Date.now()}`,
				created_at: nostrNow(),
				tags: [
				[
					"i",
					repoNaddr,
					"event",
					"wss://relay.stens.dev"
				],
				[
					"param",
					"watch_untill",
					`${nostrNow() + 60000}`
				],
				[
					"output",
					"text"
				],
				[
					"relays",
					"wss://relay.stens.dev"
				],
				[
					"j",
					"git-proposal-commit-watch"
				]
			],
			}
			const evnt = await signer.signEvent(note);

			await pool.event(evnt)

			successMessage = `Succesfully sent event ${evnt.id} at ${Date.now()}`
			return;
		} catch (ex) {
			console.log(`Error sending event: ${ex}`)
		}

		errorMessage = "Something went wrong sending the event, please try again later."
	}

	const nostrNow = (): number => Math.floor(Date.now() / 1000);

	export function isNullOrEmpty(input: string | undefined): boolean {
		return !(input && input.trim());
	}
</script>

<svelte:head>
	<title>StensDev</title>
	<meta name="description" content="StensDev Site" />
</svelte:head>

<section>

	<div hidden="{isNullOrEmpty(errorMessage)}" class="alert alert-danger mb-2" role="alert" style="color: red">
		<strong>(!)</strong> {errorMessage}
	</div>
	<div hidden="{isNullOrEmpty(successMessage)}" class="alert alert-danger mb-2" role="alert" style="color: green">
		<strong>(!)</strong> {successMessage}
	</div>
	<input type="text" bind:value={secretKeyHex} placeholder="SecretKey (HEX!)">
	<textarea id="repoNaddr" bind:value={repoNaddr} rows="10" class="form-control" name="feedback-text"
			  placeholder="Nostr address to watch: naddr1b4..."></textarea>

	<button type="button" on:click={submit} class="btn btn-blue">Watch repository</button>
<!--	<Counter />-->
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		width: 100%;
	}
</style>
