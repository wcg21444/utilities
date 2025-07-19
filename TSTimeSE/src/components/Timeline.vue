<template>
    <div class="time-line" @contextmenu="(e: MouseEvent) => { e.preventDefault(); }">
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
        <div ref="clickableArea" class="clickable-area" @click="(e: MouseEvent) => {
            let top = clickableArea?.getBoundingClientRect().y || 0;
            timeSpans.push(new TimeSpan(e.clientY - top, e.clientY - top + 10));
        }">
        </div>
        <!-- 渲染多个可拖动箭头 -->
        <div v-for="(timeSpan, index) in timeSpans" :key="index" class="time-span" :style="(timeSpan.spanStyle as any)"
            @contextmenu="(e: MouseEvent) => { e.preventDefault(); timeSpans.splice(index, 1); }">
            <div class="arrows-pair" :style="(timeSpan.arrowPairStyle as any)">
                <Vue3DraggableResizable :disabled-x="true" :disabled-y="timeSpan.disabledStartY" :resizable="false"
                    :style="{ border: '0px', transition: '' }" v-model:y="(timeSpan.startY as any)" @dragging="timeSpan.startPushEnd()
                        " @drag-start="timeSpan.startPushEnd()" @drag-end="timeSpan.startPushEnd()">
                    <div class="arrow"></div>
                    <a class="text">Start:{{ timeSpan.yToTime(timeSpan.startY as any) }}</a>

                    <p class="duration-text">Duration:{{ timeSpan.computedTime }}</p>
                </Vue3DraggableResizable>

                <Vue3DraggableResizable :disabled-x="true" :disabled-y="timeSpan.disabledEndY" :resizable="false"
                    :style="{ border: '0px', transition: '' }" v-model:y="(timeSpan.endY as any)"
                    @dragging="timeSpan.endPushStart()" @drag-start="timeSpan.endPushStart()"
                    @drag-end="timeSpan.endPushStart()">
                    <div class="arrow"></div>
                    <a class="text">End:{{ timeSpan.yToTime(timeSpan.endY as any) }}</a>
                </Vue3DraggableResizable>
            </div>
        </div>
        <!-- <div>
            y:{{ timeSpans[0].startY }}<button @click=" timeSpans[0].startY +=10">+</button><button
                            @click="timeSpans[0].startY -= 10">-</button>
            </div> -->
    </div>

    <!-- TODO: 子组件x,y变化 引起父组件变量变化,变化同步 -->
    <!-- <DraagableArrow>
        <a>{{ timeSpans[0].startY }}</a>
    </DraagableArrow> -->

</template>

<script setup lang="ts">
import { ref, type CSSProperties, computed, type Ref, type ComputedRef } from 'vue'
import DraggableBall from './DraggableBall.vue';
import DraagableArrow from './DraagableArrow.vue'
import Vue3DraggableResizable from 'vue3-draggable-resizable'
import 'vue3-draggable-resizable/dist/Vue3DraggableResizable.css'


interface TimeTick {
    time: string
}

//TODO:start , endY 钳制
//通用drag 方法
//样式计算方法

class TimeSpan {
    startY: Ref<number>;
    endY: Ref<number>;
    disabledStartY: boolean;
    disabledEndY: boolean;
    spanStyle: ComputedRef<CSSProperties>;
    arrowPairStyle: ComputedRef<CSSProperties>;
    computedTime: ComputedRef<string> = computed(() => {
        let rect = clickableArea.value?.getBoundingClientRect();
        let top = rect?.y || 0;
        let bottom = rect?.bottom || 0;
        let timePartial = (this.endY.value - this.startY.value) / (bottom - top); //partial; transform into min , then hour 
        let hour = Math.floor(timePartial * 24);
        let min = Math.floor((timePartial * 24 - hour) * 60);
        return `${hour.toString().padStart(2, '0')}:${min.toString().padStart(2, '0')}`;
    });
    constructor(
        startY: number,
        endY: number,
        disabledStartY: boolean = false,
        disabledEndY: boolean = false
    ) {
        this.startY = ref(startY);
        this.endY = ref(endY);
        this.disabledStartY = disabledStartY;
        this.disabledEndY = disabledEndY;
        this.spanStyle = computed(() => {
            //startY发生改变,但是没有触发计算量更新  // 改成引用, template不绑定.value 才能触发更新  绑定.value 导致值无法正常绑定  !!模板里面不应该.value!!
            return {//这里引用要用value
                transform: `translateY(${this.startY.value}px)`,
                height: `${this.endY.value - this.startY.value}px`,
            };
        })
        this.arrowPairStyle = computed(() => {
            const offset = -5;
            return {
                transform: `translateY(${offset - this.startY.value}px)`,
            };
        })
    }
    startPushEnd() {
        //这里引用不用.value
        if (this.startY > this.endY) { this.endY = this.startY }
    }
    endPushStart() {
        if (this.startY > this.endY) { this.startY = this.endY }
    }
    yToTime(y: number): string {
        let rect = clickableArea.value?.getBoundingClientRect();
        let top = rect?.y || 0;
        let bottom = rect?.bottom || 0;
        let timePartial = y / (bottom - top); //partial; transform into min , then hour 
        let hour = Math.floor(timePartial * 24);
        let min = Math.floor((timePartial * 24 - hour) * 60);
        return `${hour.toString().padStart(2, '0')}:${min.toString().padStart(2, '0')}`;
    }

}


// 先定义空数组
const timeSpans: Ref<TimeSpan[]> = ref([]);
// 然后 push 进去对象（此时 timeSpans 已有类型，computed 能正确识别）
timeSpans.value.push(new TimeSpan(0, 10));
timeSpans.value.push(new TimeSpan(20, 40));

const clickableArea = ref<HTMLElement | null>(null);

// const timeTicks: TimeTick[] = [
//     { time: "23:00" },
//     { time: "23:10" },
//     { time: "23:20" },
//     { time: "23:30" },
//     { time: "23:40" },
//     { time: "23:50" },
//     { time: "00:00" },
// ]
const timeTicks: Ref<TimeTick[]> = ref([
]);
for (let hour = 0; hour < 24; hour++) {
    const time = `${hour.toString().padStart(2, '0')}:00`;
    timeTicks.value.push({ time });
}
</script>

<style scoped>
.clickable-area {
    position: absolute;
    top: 0;
    left: 200px;
    width: 20%;
    height: 100%;
    background-color: rgba(216, 191, 216, 0.31);
    z-index: 2;
}

.arrows-pair {
    position: relative;
    z-index: 1;

}

.arrow {
    width: 50px;
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


.time-span {
    position: absolute;
    left: 50%;
    width: 40px;
    top: 0%;
    z-index: 2;
}

.time-span::before {
    content: '';
    background-color: #3e71c84d;
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 0;
}

.time-line {
    position: relative;
    max-width: 500px;
    max-height: max-content;
    margin: 0 auto;
    padding: 20px 0;
    user-select: none;
}

.timeline-item {
    background-color: #ffffff00;
}

.time-line::after {
    content: '';
    position: absolute;
    top: 0;
    bottom: 0;
    left: 50%;
    width: 3%;
    background: var(--line-color, #9d9d9d);
    transform: translateX(-50%);
}

.tick {
    margin-top: 0%;
    position: relative;
    left: 50%;
    width: 20%;
    z-index: 1;
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
    color: rgb(198, 139, 139);
    margin-left: 150%;
    font-size: 10px;
    z-index: 2;
}

.tick-marker {
    position: absolute;
    left: 6%;
    top: 35%;
    width: 10px;
    height: 10px;
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

.text {
    color: #bf7878;
}

.duration-text {
    color: #787878;
    position: relative;
    left: 0;
}

/* ::v-deep {
  .vdr-container {
    border: 0px solid transparent;
  }
} */
</style>
