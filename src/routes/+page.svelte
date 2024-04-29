<script lang="ts">
	import { onMount } from 'svelte';
	import Prism from 'prismjs';
	import 'prismjs/themes/prism.css';
	import 'prismjs/components/prism-python';
	import { PYTHON, VALID_KEY_PRESSES } from '$lib/const';

	let codeAreaDiv: HTMLElement;
	let code: string;
	let writtenCodeDiv: HTMLElement;
	let writtenCode: string;
	let writtenCodeSpan: HTMLElement;
	let flashRed = false;
	let changeGreen = false;

	function getNextValidNodeWithText(element: HTMLElement): HTMLElement | null {
		if (!element) throw new Error('Invalid element');
		for (let node of element.childNodes as NodeListOf<HTMLElement>) {
			if (node?.innerHTML === '' || node.nodeValue === '') {
				element.removeChild(node);
				return getNextValidNodeWithText(element);
			}
			if (node.nodeType === Node.TEXT_NODE) {
				return node;
			} else if (node.nodeType === Node.ELEMENT_NODE) {
				return getNextValidNodeWithText(node);
			}
		}
		return null;
	}

	function handleKey(k: KeyboardEvent) {
		k.preventDefault();
		let key = k.key;
		if (!VALID_KEY_PRESSES.includes(key)) {
			return;
		}
		if (k.key === 'Enter') key = '\n';
		let n = getNextValidNodeWithText(writtenCodeDiv);
		if (n == null) {
			changeGreen = true;
			return;
		}
		if (n?.nodeValue?.at(0) == key) {
			n.nodeValue = n.nodeValue.substring(1);
			if (getNextValidNodeWithText(writtenCodeDiv) == null) changeGreen = true;
			writtenCode += key;
		} else {
			flashRed = true;
			setTimeout(() => (flashRed = false), 200);
		}
	}

	onMount(async () => {
		let prettyCode = Prism.highlight(PYTHON, Prism.languages.py, 'python');
		code = prettyCode;
	});
</script>

<div id="timer">
	<div>TEST</div>
	<svg height="100" width="100">
		<path
			d="m 50,2
          a 48,48 0 1,0 0,96
          a 48,48 0 1,0 0,-96"
			fill="none"
			stroke="#d9d9d9"
			stroke-width="4"
		></path>
		<path
			d="m 50,2
          a 48,48 0 1,0 0,96
          a 48,48 0 1,0 0,-96"
			fill="none"
			stroke="rgba(0, 71, 119, 1)"
			stroke-linecap="round"
			stroke-width="4"
			stroke-dasharray="301.59289474462014"
			stroke-dashoffset="0"
		></path>
	</svg>
</div>

<div
	id="codearea"
	role="button"
	tabindex="-1"
	on:keydown={() => {}}
	on:click={() => writtenCodeSpan.focus()}
	bind:this={codeAreaDiv}
	class:flash-red={flashRed}
	class:change-green={changeGreen}
>
	<span bind:innerText={writtenCode} contenteditable="false"></span>&#8203;
	<span
		role="button"
		tabindex="-1"
		contenteditable="true"
		bind:this={writtenCodeSpan}
		on:keydown={handleKey}
	></span>
	&#8203;
	<div contenteditable="false" bind:innerHTML={code} bind:this={writtenCodeDiv}></div>
</div>

<style>
	#timer > div {
		position: absolute;
		top: 50px;
		left: 35px;
	}

	@keyframes changeGreen {
		from {
			background: initial;
		}
		to {
			background: rgba(0, 255, 0, 0.2);
		}
	}

	.change-green {
		animation: changeGreen 0.5s linear;
		animation-fill-mode: forwards;
	}

	@keyframes flashRed {
		0% {
			background-color: initial;
		}
		50% {
			background-color: rgba(255, 0, 0, 0.1);
		}
		100% {
			background-color: initial;
		}
	}

	.flash-red {
		animation: flashRed 0.2s ease-in-out;
	}
	#codearea {
		margin: 5em 10vmax;
		border: 2px solid #ccc;
		border-radius: 16px;
		padding: 10px;
		background: white;
	}
	#codearea > div {
		opacity: 0.7;
		display: inline;
		white-space: pre-wrap;
	}
	#codearea > span {
		outline: 0px solid transparent;
		display: inline;
		color: green;
	}
</style>
