<script lang="ts">
	import { browser, dev } from '$app/env';
	import QRCode from 'qrcode';
	import { onMount } from 'svelte';
	import { State } from '../enums';
	import { Auth } from '../lib/auth';
	import type { Tokens } from '../models';

	let tokens = null;
	let state = State.SignedOut;
	let qrCode: string | null = null;

	const code = browser ? window.location.href.match(/code=([A-Za-z0-9-_]+)/)?.[1] : null;

	if (code) {
		state = State.SigningIn;
		new Auth().fetchTokensFromCode(code).then((tokens) => {
			window.dispatchEvent(new CustomEvent('tokens'));
			window.close();
		});
	}

	function signin() {
		const url = new URL('https://api.soundcloud.com/connect');
		url.searchParams.append('response_type', 'code');
		url.searchParams.append(
			'client_id',
			dev ? 'Gbv3N4cjTjVMwfaHVbCdEB7W5Y3JQM28' : 'ttmRWSTGJ7pzm1s8znU3CSJ5mXSjtS0l'
		);
		url.searchParams.append(
			'redirect_uri',
			dev ? 'http://localhost:3000' : 'https://app.vulpine.fm/oauth'
		);

		window.open(url.toString());
	}

	function signout() {
		new Auth().clearTokens();
		state = State.SignedOut;
		tokens = null;
	}

	function createQRCode(tokens: Tokens) {
		QRCode.toDataURL(JSON.stringify(tokens), { errorCorrectionLevel: 'M' }, function (err, url) {
			if (err) return console.error(err);
			qrCode = url;
		});
	}

	onMount(async () => {
		tokens = await new Auth().getTokens();
		if (tokens) {
			state = State.SignedIn;
			createQRCode(tokens);
		}
	});
</script>

<main class="root">
	<div class="container">
		<img class="logo" src="icon_512.png" alt="" />

		<h1>RainCloud</h1>
		{#if state === State.SignedOut}
			<p>Welcome to RainCloud, a SoundCloud app for KaiOS.</p>
			<p>
				Let's get started. First, let's get signed into SoundCloud. After you sign in, refresh this
				page.
			</p>
			<button on:click={signin}>Sign in to SoundCloud</button>
		{:else if state === State.SigningIn}
			<p>Give me a second while I get your tokens...</p>
		{:else}
			<p>Great! Next, you'll need to scan the QR code below with the RainCloud app.</p>
			<img class="qr" src={qrCode} alt="" />
			<button on:click={signout}>Sign Out</button>
		{/if}
	</div>
</main>

<style>
	.root {
		height: 100vh;
		width: 100vw;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		background-color: #fefefe;
	}

	.container {
		padding: 20px;
		background-color: #fff;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		max-width: 400px;
		box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
		text-align: center;
	}

	.container > .logo {
		height: 192px;
		width: 192px;
		margin-top: -20px;
	}

	.container > h1 {
		margin: -20px 0 20px 0;
	}
</style>
