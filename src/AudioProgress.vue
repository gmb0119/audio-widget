<script setup lang="ts">
import { ref } from 'vue';

const emit = defineEmits<{
	( e: 'seek', time: number ): void;
}>();

const props = defineProps<{
	currentTime: number;
	duration: number;
	buffered: number;
}>();

const progressBar = ref<HTMLElement>();
const dragging = ref( false );

const formatTime = ( time: number ) => {
	const minutes = Math.floor( time / 60 );
	const seconds = Math.floor( time % 60 );
	return `${minutes}:${seconds.toString().padStart( 2, '0' )}`;
};

const timeAsPct = ( time: number ) => {
	return `${ time / props.duration * 100}%`;
};

const handlePointerDown = ( e: PointerEvent ) => {
	e.preventDefault();
	setTime( e );
	dragging.value = true;
	window.addEventListener( 'pointermove', setTime );
	window.addEventListener( 'pointerup', handlePointerUp );
};

const handlePointerUp = ( e: PointerEvent ) => {
	e.preventDefault();
	dragging.value = false;
	window.removeEventListener( 'pointermove', setTime );
	window.removeEventListener( 'pointerup', handlePointerUp );
};

const setTime = ( e: MouseEvent | PointerEvent ) => {
	if ( !progressBar.value ) return;
	e.preventDefault();
	const bbox = progressBar.value.getBoundingClientRect();
	const x = e.clientX - bbox.left;
	const pct = x / bbox.width;
	emit( 'seek', pct * props.duration );
};
</script>

<template>
	<div class="audio-progress">
		<div
			ref="progressBar"
			class="progress-bar"
			:class="{ dragging }"
			role="slider"
			aria-label="Seek"
			aria-valuemin="0"
			:aria-valuemax="duration"
			:aria-valuenow="currentTime"
			@pointerdown="handlePointerDown"
		>
			<div class="progress-bar-inner">
				<div class="progress-bar-buffered" :style="{ width: timeAsPct( buffered ) }"></div>
				<div class="progress-bar-played" :style="{ width: timeAsPct( currentTime ) }"></div>
				<div class="progress-bar-handle" :style="{ left: timeAsPct( currentTime ) }"></div>
			</div>
		</div>
		<div class="progress-time">
			<div class="progress-time-current">{{ formatTime( currentTime ) }}</div>
			<div class="progress-time-total">{{ formatTime( duration ) }}</div>
		</div>
	</div>
</template>

<style scoped>
.progress-bar {
  width: 100%;
  position: relative;
	cursor: pointer;
	height: 20px;
	&:hover, &.dragging {
		.progress-bar-handle {
			transform: translate(-50%, -50%) scale(1.4);
		}
		.progress-bar-played {
			height: 4.8px;
		}
	}
}

.progress-bar-inner {
	pointer-events: none;
	border-radius: 2px;
	height: 2.4px;
	position: absolute;
	top: 50%;
	transform: translateY(-50%);
  background-color: var(--color-bar2);
	width: 100%;
	> * {
		height: 100%;
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		transition: all 320ms var(--ease-out-elastic);
		transition-property: transform, height;
	}
}

.progress-bar-buffered {
  background-color: var(--color-bar);
}

.progress-bar-played {
  background-color: var(--color-foreground);
}

.progress-bar-handle {
	background-color: var(--color-foreground);
	width: 12px;
	height: 12px;
	border-radius: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
}

.progress-time {
  font-size: 12px;
  opacity: 0.7;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.audio-progress {
  display: flex;
  flex-direction: column;
}
</style>
