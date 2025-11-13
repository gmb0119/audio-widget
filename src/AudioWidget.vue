<script setup lang="ts">
import { computed, ref, watch } from 'vue';

import type { SongInfo } from '@/types';

import AudioProgress from '@/AudioProgress.vue';
import HeartAnimation from '@/HeartAnimation.vue';

const props = defineProps<{
	songs: SongInfo[];
}>();

const emit = defineEmits<{
	( e: 'like', songId: string ): void;
}>();

const audio = new Audio();

const currentSongIndex = ref( 0 );
const currentSong = computed( () => props.songs[ currentSongIndex.value ] );

const isPlaying = ref( false );

const currentTime = ref( 0 );
const buffered = ref( 0 );
const duration = ref( 0 );

watch( currentSong, () => {

	if ( !currentSong.value ) return;

	audio.src = currentSong.value.audioPath;
	audio.load();

	buffered.value = 0;
	currentTime.value = 0;

	if ( isPlaying.value ) {
		audio.play();
	}

	audio.addEventListener( 'loadedmetadata', () => {
		duration.value = audio.duration;
	} );

	audio.addEventListener( 'progress', () => {
		buffered.value = audio.buffered.length > 0 ? audio.buffered.end( 0 ) : 0;
	} );

	audio.addEventListener( 'timeupdate', () => {
		currentTime.value = audio.currentTime;
	} );

	audio.addEventListener( 'play', () => {
		isPlaying.value = true;
	} );

	audio.addEventListener( 'pause', () => {
		isPlaying.value = false;
	} );

	// todo: This should keep playing after advancing. 'pause' gets called
	// first, setting `isPlaying` to false. We would need another ref like
	// `userRequestsPlayback` to manage this.
	audio.addEventListener( 'ended', () => {
		handleNext();
	} );

}, { immediate: true } );

const handlePlayPause = () => {
	if ( isPlaying.value ) {
		audio.pause();
	} else {
		audio.play();
	}
};

const handleNext = () => {
	currentSongIndex.value++;
	if ( currentSongIndex.value >= props.songs.length ) {
		currentSongIndex.value = 0;
	}
};

const handlePrevious = () => {
	currentSongIndex.value--;
	if ( currentSongIndex.value < 0 ) {
		currentSongIndex.value = props.songs.length - 1;
	}
};

const handleRepeat = () => {
	alert( 'Repeat not implemented' );
};

const handleSeek = ( time: number ) => {
	audio.currentTime = time;
};

const likeAnimationTrigger = ref( 0 );

const handleLike = () => {
	if ( !currentSong.value ) return;
	emit( 'like', currentSong.value.id );
	if ( currentSong.value.liked ) likeAnimationTrigger.value++;
};

</script>

<template>
	<div v-if="currentSong" class="audio-widget">
		<div class="now-playing">
			<div class="album-art">
				<img :src="currentSong.albumArtPath" alt="Album Art" />
			</div>
			<div class="song-info">
				<div class="song-title">{{ currentSong.title }}</div>
				<div class="song-artist">{{ currentSong.artist }}</div>
			</div>
		</div>

		<AudioProgress
			class="audio-progress"
			:current-time="currentTime"
			:duration="duration"
			:buffered="buffered"
			@seek="handleSeek"
		/>

		<div class="controls">
			<button
				class="control-button"
				aria-label="Repeat"
				@click="handleRepeat"
			>
				<span class="button-inner">
					<span class="material-symbols-rounded size-s">repeat</span>
				</span>
			</button>

			<button
				class="control-button"
				aria-label="Previous track"
				@click="handlePrevious"
			>
				<span class="button-inner">
					<span class="material-symbols-rounded size-m">skip_previous</span>
				</span>
			</button>

			<button
				class="control-button play-pause-button"
				:aria-label="isPlaying ? 'Pause' : 'Play'"
				@click="handlePlayPause"
			>
				<span class="button-inner">
					<span
						class="material-symbols-rounded size-l"
					>{{ isPlaying ? 'pause' : 'play_arrow' }}</span>
				</span>
			</button>

			<button
				class="control-button"
				aria-label="Next track"
				@click="handleNext"
			>
				<span class="button-inner">
					<span class="material-symbols-rounded size-m">skip_next</span>
				</span>
			</button>

			<button
				class="control-button favorite-button"
				:aria-label="currentSong.liked ? 'Unlike' : 'Like'"
				@click="handleLike"
			>
				<span class="button-inner">
					<span
						class="material-symbols-rounded size-s"
						:class="{ 'liked': currentSong.liked }"
					>favorite</span>
				</span>
				<HeartAnimation :trigger="likeAnimationTrigger" />
			</button>
		</div>
	</div>
</template>

<style scoped>
.audio-widget {
	width: 100%;
	max-width: 480px;
	border-radius: 16px;
	padding: 32px;

	font-family: 'Google Sans Text', sans-serif;
	background-color: var(--color-background);
	color: var(--color-foreground);

	display: flex;
	flex-direction: column;
	align-items: stretch;
	justify-content: center;

	overflow: hidden;
	user-select: none;

	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-rendering: optimizeLegibility;
}

.now-playing {
	display: flex;
}

.song-info {
	padding: 16px;
	display: flex;
	flex-direction: column;
	gap: 8px;
	overflow: hidden;
	margin-bottom: 24px;
}

.song-title {
	font-family: 'Google Sans', sans-serif;
	font-size: 24px;
	font-weight: 500;
	text-overflow: ellipsis;
	overflow: hidden;
	white-space: nowrap;
}

.song-artist {
	font-size: 16px;
	font-weight: 400;
	text-overflow: ellipsis;
	overflow: hidden;
	white-space: nowrap;
	opacity: 0.5;
}

.album-art {
	width: 88px;
	height: 88px;
	border-radius: 4px;
	overflow: hidden;
	flex-shrink: 0;
	> img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}
}

.audio-progress {
	margin-bottom: 8px;
}

.controls {
	max-width: 312px;
	width: 100%;
	align-self: center;
	display: flex;
	justify-content: space-evenly;
	align-items: center;
	gap: 16px;
}

.control-button {
	border-radius: 50%;
	background: transparent;
	border: none;
	color: var(--color-foreground);
	width: 48px;
	height: 48px;
	display: flex;
	align-items: center;
	justify-content: center;
	cursor: pointer;
	outline: 1px solid #fff0;
	outline-offset: -1px;
	position: relative;
	overflow: visible;
	pointer-events: all;
	&:focus {
		outline-color: #fff2;
	}
	&:hover {
		outline-color: #fff0;
	}
}

/* Since hover effects for these buttons involve scale, scale an *inner* element
so we don't mess with the hit area. */
.button-inner {
	display: flex;
	align-items: center;
	justify-content: center;
	border-radius: 50%;
	width: 100%;
	height: 100%;
	background: transparent;
	transition:
		transform 170ms var(--ease-out-elastic),
		background-color 100ms linear;
	.control-button:hover & {
		background-color: #fff1;
	}
	.control-button:active & {
		transform: scale(0.85);
		transition: none;
	}
}

.play-pause-button {
	width: 72px;
	height: 72px;

	.button-inner {
		background-color: var(--color-selected);
	}

	&:hover .button-inner {
		background-color: color-mix(in srgb, var(--color-selected) 90%, #fff);
	}
}

.control-button .size-s {
	font-size: 24px;
}

.control-button .size-m {
	font-size: 36px;
}

.control-button .size-l {
	font-size: 48px;
}

.favorite-button span {
	font-variation-settings:'FILL'0,'wght'400,'GRAD'0,'opsz'24;
	margin-top: 2px;
}

.favorite-button span.liked {
	font-variation-settings:'FILL'1,'wght'400,'GRAD'0,'opsz'24;
}
</style>
