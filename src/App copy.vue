<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";
import { useCardStore } from "./stores/card";

const cardStore = useCardStore();
const canvas = ref<HTMLCanvasElement | null>(null);
const ctx = ref<CanvasRenderingContext2D | null>(null);

onMounted(() => {
    cardStore.setRandomCard();
    
    if (canvas.value) {
        canvas.value.width = window.innerWidth;
        canvas.value.height = window.innerHeight;
        
        const context = canvas.value.getContext("2d");
        ctx.value = context;
        if (context) {
            context.fillStyle = "rgba(0,0,0,0.5)";
            context.fillRect(0, 0, canvas.value.width, canvas.value.height);

            // Initialize drawing context
            context.strokeStyle = "red"; // Set drawing color to red
            context.lineWidth = 30; // Set line width
            context.lineCap = "round"; // Smooth edges
        }
    }

    // Prevent sleep
    // preventSleep();

    if (document.documentElement.requestFullscreen) {
        document.documentElement.requestFullscreen();
    }

    // Add touch event listeners
    const drawStart = (event: TouchEvent) => {
        if (ctx.value) {
            const touch = event.touches[0];
            ctx.value.beginPath();
            ctx.value.moveTo(touch.clientX, touch.clientY);
        }
    };

    const drawMove = (event: TouchEvent) => {
        if (ctx.value) {
            const touch = event.touches[0];
            ctx.value.lineTo(touch.clientX, touch.clientY);
            ctx.value.stroke();
        }
        event.preventDefault(); // Prevent scrolling
    };

    const drawEnd = (event: TouchEvent) => {
        if (ctx.value) {
            ctx.value.closePath();
        }
    };

    // Attach event listeners
    canvas.value?.addEventListener("touchstart", drawStart);
    canvas.value?.addEventListener("touchmove", drawMove);
    canvas.value?.addEventListener("touchend", drawEnd);
});

onUnmounted(() => {
    if (document.exitFullscreen) {
        document.exitFullscreen();
    }

    // Remove event listeners
    canvas.value?.removeEventListener("touchstart", drawStart);
    canvas.value?.removeEventListener("touchmove", drawMove);
    canvas.value?.removeEventListener("touchend", drawEnd);
});
</script>

<template>
    <div id="app">
        <div class="background" :style="{ backgroundImage: `url(${cardStore.cardImage})` }"></div>
        <canvas ref="canvas" class="foreground"></canvas>
    </div>
</template>

<style>
body {
    margin: 0;
    padding: 0;
    overflow: hidden;
}
#app {
    height: 100%;
    width: 100%;
}
.background {
    position: fixed;
    top: 50%;
    left: 50%;
    width: 90vw;  /* 90% of viewport width */
    height: 90vh;  /* 90% of viewport height */
    background-repeat: no-repeat;
    background-size: contain; /* Ensures the entire image is visible */
    background-position: center; /* Centers the image */
    transform: translate(-50%, -50%); /* Centers the element */
}
.foreground {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    touch-action: none; /* Ensures touch events are not blocked */
}
</style>