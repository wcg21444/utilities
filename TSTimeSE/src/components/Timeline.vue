<template>
    <div class="time-line">
        <div v-for="(tick, index) in timeTicks" :key="index" class="timeline-item">
            <div class="tick">
                <div class="tick-content">
                    <slot name="tick" :item="tick" :index="index" class="">
                        <h3>{{ tick.time }}</h3>
                    </slot>
                </div>
                <div class="tick-marker"></div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface TimeTick {
    time: string
}

const timeTicks: TimeTick[] = [
    { time: "23:00" },
    { time: "23:10" },
    { time: "23:20" },
    { time: "23:30" },
    { time: "23:40" },
    { time: "23:50" },
    { time: "00:00" },
]
</script>

<style scoped>
.time-line {
    position: relative;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px 0;
}

.timeline-item {}


.time-line::after {
    content: '';
    position: absolute;
    top: 0;
    bottom: 0;
    left: 50%;
    width: 5%;
    background: var(--line-color, #9d9d9d);
    transform: translateX(-50%);
}

.tick {
    margin-top: 5%;
    position: relative;
    left: 50%;
    width: 20%;
}

/* tick line */
.tick::before {
    content: '';
    background-color: #9d9d9d;
    position: absolute;
    top: 50%;
    width: 50%;
    height: 3px;
    opacity: 50%;
    box-shadow: 1px 1px 2px 1px rgba(0, 0, 0, 0.2);
}

.tick-content {
    color: rgb(82, 82, 82);
    margin-left: 50%;
}

.tick-marker {
    position: absolute;
    left: 10%;
    top: 20%;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    transform: translateX(-50%);
    z-index: 1;
    background-color: #787878;
    box-shadow: -1px 1px 4px 2px rgba(0, 0, 0, 0.2);
}
</style>
