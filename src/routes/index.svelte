<script lang="ts">
	import { browser } from '$app/env';
	import min from 'lodash/min.js';
	let analyser: AnalyserNode;
	let length: number;
	let frequencyArray: Float32Array;
	let tempFrequencyArray: Float32Array;
	let timeArray: Float32Array;
	let minDb: number = 1;
	async function getMedia() {
		const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
		const audioCtx = new AudioContext();
		const source = audioCtx.createMediaStreamSource(stream);
		analyser = audioCtx.createAnalyser();
		analyser.fftSize = 2048;
		source.connect(analyser);
		length = Math.floor(analyser.frequencyBinCount * 0.67);
		timeArray = new Float32Array(length);
		frequencyArray = new Float32Array(length);
		tempFrequencyArray = new Float32Array(analyser.frequencyBinCount);
		paint();
	}

	function paint() {
		// analyser.getFloatTimeDomainData(timeArray);
		// timeArray = timeArray;
		analyser.getFloatFrequencyData(tempFrequencyArray);
		frequencyArray = tempFrequencyArray.slice(0, length);
    minDb = min(frequencyArray) ?? -70;
		requestAnimationFrame(paint);
	}
	if (browser) {
		getMedia();
	}
</script>

<div class="flex items-end h-screen w-screen bg-gray-200">
	{#if frequencyArray}
		{#each frequencyArray as db}
			{@const h = Math.abs(db) * 2}
			<div
				class="flex-1"
				style:height="{(1 / (db / minDb))}%"
				style:background="hsl({h}, 50%, 50%)"
			/>
		{/each}
	{/if}
</div>
