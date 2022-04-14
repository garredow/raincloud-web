<script lang="ts">
	import { browser } from '$app/env';
	import { Auth } from '../../lib/auth';

	const code = browser ? window.location.href.match(/code=([A-Za-z0-9-_]+)/)?.[1] : null;

	if (code) {
		new Auth().fetchTokensFromCode(code).then(() => {
			window.dispatchEvent(new CustomEvent('tokens'));
			window.close();
		});
	}
</script>

<main class="root">
	<div class="container">
		<img class="logo" src="assets/images/icon_512.png" alt="" />
		<h1>RainCloud</h1>
		<p>Give me a second while I get your tokens...</p>
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
