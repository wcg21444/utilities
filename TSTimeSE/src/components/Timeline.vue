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
        <!-- 渲染多个可拖动箭头 -->
        <!-- <Vue3DraggableResizable v-for="(arrow, index) in arrows" :disabled-x="true" :key="index" :resizable="false"
            :style="{ border: '0px', transition: '20ms' }">
            <div class="timeline-arrows">
                <div class="arrow"></div>
            </div>
        </Vue3DraggableResizable> -->

        <div class="time-span-fill" :style="unref(timeSpans[0].spanStyle)">
            <div class="arrows-pair" :style="unref(timeSpans[0].arrowPairStyle)">
                <Vue3DraggableResizable :disabled-x="true" :disabled-y="timeSpans[0].disabledStartY" :resizable="false"
                    :style="{ border: '0px', transition: '' }" v-model:y="timeSpans[0].startY" :maxTop="400" @dragging="() => {
                        if (timeSpans[0].startY > timeSpans[0].endY) { timeSpans[0].endY = timeSpans[0].startY }
                    }"
                    @drag-start="() => { if (timeSpans[0].startY > timeSpans[0].endY) { timeSpans[0].endY = timeSpans[0].startY } }"
                    @drag-end="() => { if (timeSpans[0].startY > timeSpans[0].endY) { timeSpans[0].endY = timeSpans[0].startY } }">
                    <div class="arrow"></div>
                    <a>startY</a>
                    <a>{{ timeSpans[0].startY }}</a>
                </Vue3DraggableResizable>

                <Vue3DraggableResizable :disabled-x="true" :disabled-y="timeSpans[0].disabledEndY" :resizable="false"
                    :style="{ border: '0px', transition: '' }" v-model:y="timeSpans[0].endY" @dragging="() => {
                        if (timeSpans[0].startY > timeSpans[0].endY) { timeSpans[0].startY = timeSpans[0].endY }
                    }"
                    @drag-start="() => { if (timeSpans[0].startY > timeSpans[0].endY) { timeSpans[0].startY = timeSpans[0].endY } }"
                    @drag-end="() => { if (timeSpans[0].startY > timeSpans[0].endY) { timeSpans[0].startY = timeSpans[0].endY } }">
                    <div class="arrow"></div>
                    <a>endY</a>
                    <a>{{ timeSpans[0].endY }}</a>
                </Vue3DraggableResizable>
            </div>

        </div>
        <!-- <div>
            y:{{ timeSpans[0].startY }}<button @click="timeSpans[0].startY += 10">+</button><button
                @click="timeSpans[0].startY -= 10">-</button>
        </div> -->
    </div>

    <!-- TODO: 子组件x,y变化 引起父组件变量变化,变化同步 -->
    <!-- <DraagableArrow>
        <a>{{ timeSpans[0].startY }}</a>
    </DraagableArrow> -->

</template>

<script setup lang="ts">
import { ref, type CSSProperties, computed, type Ref, type ComputedRef, unref } from 'vue'
import DraggableBall from './DraggableBall.vue';
import DraagableArrow from './DraagableArrow.vue'
import Vue3DraggableResizable from 'vue3-draggable-resizable'
import 'vue3-draggable-resizable/dist/Vue3DraggableResizable.css'

//TODO:start , endY 钳制
//通用drag 方法
interface TimeSpan {
    startY: number;
    endY: number;
    disabledStartY: boolean;
    disabledEndY: boolean;
    spanStyle: ComputedRef<CSSProperties>;
    arrowPairStyle: ComputedRef<CSSProperties>;

}

// 先定义空数组
const timeSpans: Ref<TimeSpan[]> = ref([]);
// 然后 push 进去对象（此时 timeSpans 已有类型，computed 能正确识别）
timeSpans.value.push({
    startY: 0,
    endY: 10,
    disabledStartY: false,
    disabledEndY: false,
    spanStyle: computed(() => {
        const currentSpan = timeSpans.value[0];
        return {
            transform: `translateY(${currentSpan.startY}px)`, // 替代 top
            height: `${currentSpan.endY - currentSpan.startY}px`,
        };
    }),
    arrowPairStyle: computed(() => {
        const currentSpan = timeSpans.value[0];
        const offset = -5;
        return {
            transform: `translateY(${offset - currentSpan.startY}px)`, // 替代 top
        };
    })
});



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
.arrows-pair {
    position: relative;
    z-index: 1;
}

.arrow {
    width: 40px;
    height: 10px;
    background-color: #3a81f2;
    rotate: 180deg;
    clip-path: polygon(0% 20%,
            00% 00%,
            80% 00%,
            /*/---*/
            100% 50%,
            80% 100%,
            /*\---*/
            50% 100%,
            0% 100%);
    opacity: 0.8;
    user-select: none;
    cursor: pointer;

}

.arrow:hover {
    box-shadow: -10px 1px 4px 2px rgba(0, 0, 0, 0.2);
    background-color: #5f9cff;
}


.time-span-fill {
    background-color: #5694ff4d;
    position: absolute;
    left: 50%;
    width: 40px;
}

.time-line {
    position: relative;
    max-width: 500px;
    max-height: max-content;
    margin: 0 auto;
    padding: 20px 0;
}

.timeline-item {
    background-color: #3a81f210;
}

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
    z-index: 1;
}

.tick-content {
    color: rgb(255, 0, 0);
    margin-left: 50%;
    z-index: 2;
}

.tick-marker {
    position: absolute;
    left: 10%;
    top: 20%;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    transform: translateX(-50%);
    z-index: 2;
    background-color: #787878;
    box-shadow: -1px 1px 4px 2px rgba(0, 0, 0, 0.2);
}




.point {
    position: absolute;
    width: 50px;
    height: 50px;
    background-color: #ff0000;
    border-radius: 50%;
    transition: 50ms;
    animation-delay: 1ms;
}

.timeline-arrows {
    content: '';
    position: relative;
    /* left: 0%; */
    z-index: 1;
}

/* ::v-deep {
  .vdr-container {
    border: 0px solid transparent;
  }
} */
</style>
