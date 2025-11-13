<script setup lang="ts">
import { ref, watch } from 'vue';

const props = defineProps<{
	trigger: number;
}>();

const triggered = ref( false );

watch( () => props.trigger, () => {
	triggered.value = false;
	requestAnimationFrame( () => {
		triggered.value = true;
	} );
} );
</script>

<template>
	<div class="heart-animation" :class="{ triggered }">
		<span class="material-symbols-rounded">favorite</span>
	</div>
</template>

<style scoped>
.heart-animation {
	pointer-events: none;
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	color: rgb(255, 0, 55);
	font-size: 24px;
	opacity: 0;
	&.triggered {
		animation: float-up 0.8s var(--ease-out-elastic) both;
	}
}

.material-symbols-rounded {
	font-variation-settings: 'FILL' 1, 'wght' 400, 'GRAD' 0, 'opsz' 24;
	display: block;
}

@keyframes float-up {
	0% {
		opacity: 0;
		transform: translate(-50%, -50%) scale(0.5);
	}
	30% {
		opacity: 1;
	}
	70% {
		opacity: 1;
		transform: translate(-50%, -250%) rotateY(180deg) scale(1);
	}
	100% {
		opacity: 0;
		transform: translate(-50%, -250%) rotateY(180deg) scale(1);
	}
}

</style>
