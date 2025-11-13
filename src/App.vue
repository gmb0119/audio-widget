<script setup lang="ts">
import { ref } from 'vue';

import type { SongInfo } from '@/types';

import COVER from '@/assets/cover.jpg?url';
import TRACK1 from '@/assets/track-1.mp3?url';
import TRACK2 from '@/assets/track-2.mp3?url';
import TRACK3 from '@/assets/track-3.mp3?url';
import AudioWidget from '@/AudioWidget.vue';

const songs = ref<SongInfo[]>( [
	{
		id: '1',
		title: 'Plantasia',
		artist: 'Mort Garson',
		audioPath: TRACK1,
		albumArtPath: COVER,
		liked: false,
	},
	{
		id: '2',
		title: 'Symphony for a Spider Plant',
		artist: 'Mort Garson',
		audioPath: TRACK2,
		albumArtPath: COVER,
		liked: false,
	},
	{
		id: '3',
		title: 'Baby\'s Tears Blues',
		artist: 'Mort Garson',
		audioPath: TRACK3,
		albumArtPath: COVER,
		liked: false,
	}
] );

const handleLike = ( songId: string ) => {
	const song = songs.value.find( song => song.id === songId );
	if ( song ) {
		song.liked = !song.liked;
	}
};
</script>

<template>
	<AudioWidget :songs="songs" @like="handleLike" />
</template>
