<script lang="ts">
	let sounds = [
		{ name: 'Retro alarm', file: 'alarm-clock-ringing-fascinatedsound-1-00-03.mp3' },
		{ name: 'Retro phone', file: 'retro-alarm-clock-ringing-soundroll-1-00-05.mp3' },
		{ name: 'Meme fail', file: 'meme-fail-alert-locran-1-00-01.mp3' },
		// { name: 'Applause', file: 'applause.mp3' },
		// { name: 'Drum Roll', file: 'drumroll.mp3' },
		// { name: 'Ta-da!', file: 'tada.mp3' },
		// { name: 'Boing', file: 'boing.mp3' },
		// { name: 'Whoosh', file: 'whoosh.mp3' },
		// { name: 'Ding', file: 'ding.mp3' },
		// { name: 'Buzzer', file: 'buzzer.mp3' },
		// { name: 'Pop', file: 'pop.mp3' }
	];

	let activeAudios: { audio: HTMLAudioElement; file: string }[] = [];
	let progressBars: { [key: string]: HTMLElement } = {};

	$: hasActiveSounds = activeAudios.length > 0;

	function playSound(soundFile: string) {
		// Stop existing playback of this sound
		const existing = activeAudios.find(a => a.file === soundFile);
		if (existing) {
			existing.audio.pause();
			existing.audio.currentTime = 0;
			activeAudios = activeAudios.filter(a => a !== existing);
			if (progressBars[soundFile]) {
				progressBars[soundFile].style.transition = 'none';
				progressBars[soundFile].style.width = '0%';
			}
		}
		
		try {
			const audio = new Audio(`/sounds/${soundFile}`);
			activeAudios = [...activeAudios, { audio, file: soundFile }];
			
			audio.addEventListener('loadedmetadata', () => {
				if (progressBars[soundFile]) {
					progressBars[soundFile].style.transition = `width ${audio.duration}s linear`;
					progressBars[soundFile].style.width = '100%';
				}
			});
			
			audio.addEventListener('ended', () => {
				activeAudios = activeAudios.filter(a => a.audio !== audio);
				if (progressBars[soundFile]) {
					progressBars[soundFile].style.transition = 'none';
					progressBars[soundFile].style.width = '0%';
				}
			});
			
			audio.play().catch(e => console.warn('Could not play sound:', e));
		} catch (e) {
			console.warn('Audio not supported:', e);
		}
	}

	function stopAllSounds() {
		activeAudios.forEach(({ audio }) => {
			audio.pause();
			audio.currentTime = 0;
		});
		activeAudios = [];
		Object.values(progressBars).forEach(bar => {
			bar.style.transition = 'none';
			bar.style.width = '0%';
		});
	}
</script>

<div class="p-4 h-full overflow-auto">
	<div class="mb-4 text-center">
		<button
			class="px-6 py-2 font-semibold rounded-lg shadow-lg transition-colors {hasActiveSounds ? 'bg-red-500 hover:bg-red-600 text-white' : 'bg-gray-300 text-gray-500 cursor-not-allowed'}"
			on:click={hasActiveSounds ? stopAllSounds : null}
			disabled={!hasActiveSounds}
		>
			Stop All Sounds
		</button>
	</div>
	<div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4 max-w-4xl mx-auto">
		{#each sounds as sound}
			<button
				class="aspect-square bg-blue-500 hover:bg-blue-600 text-white font-semibold rounded-lg shadow-lg transition-colors flex items-center justify-center text-center p-2 relative overflow-hidden"
				on:click={() => playSound(sound.file)}
			>
				<div 
					class="absolute inset-0 bg-green-400"
					style="width: 0%"
					bind:this={progressBars[sound.file]}
				></div>
				<span class="relative z-10">{sound.name}</span>
			</button>
		{/each}
	</div>
</div>