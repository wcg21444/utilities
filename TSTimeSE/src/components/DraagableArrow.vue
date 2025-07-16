<template>
    <div ref="dragContainer" class="drag-container" :style="pointStyle">
        <div class="arrow"></div>
        <slot></slot>

    </div>
</template>

<script setup lang="ts">
//出边界也能拖拽
//初始位置设置明确

import { computed, onMounted, ref } from 'vue'
import type { CSSProperties } from 'vue';
const isDragging = ref(false);
const x = ref(0);//位移x
const y = ref(0);//位移y
const down_x = ref(0);
const down_y = ref(0);
const last_x = ref(0);//上次位移
const last_y = ref(0);

const disabedX = ref(true);

const dragContainer = ref<HTMLElement | null>(null);
onMounted(() => {
    if (dragContainer.value) {

    }
});

//鼠标移动监听
document.addEventListener('mousemove', (event) => {
    if (!isDragging.value) return;
    x.value = last_x.value + event.pageX - down_x.value;
    y.value = last_y.value + event.pageY - down_y.value;
}
);
//鼠标点击监听
document.addEventListener('mousedown', (event) => {
    if (!dragContainer.value) return;
    const target = event.target as HTMLElement;
    if (!(target.parentElement == dragContainer.value)) {
        isDragging.value = false;
        return
    }
    down_x.value = event.pageX;
    down_y.value = event.pageY;
    isDragging.value = true;

})
document.addEventListener('mouseup', () => {
    if (!isDragging.value) return;
    isDragging.value = false;
    last_x.value = x.value;
    last_y.value = y.value;
})

const pointStyle = computed<CSSProperties>(() => {

    if (disabedX) {
        console.log(y.value)
        return {
            transform: `translate(${0}px, ${y.value}px)`, position: `absolute`
        }
    }
    return {
        transform: `translate(${x.value}px, ${y.value}px)`, position: `absolute`
    }
});

</script>

<style scoped>
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

.drag-container {
    position: absolute;
    cursor: pointer;
    background-color: #7350504f;
    transition: 10ms;
    /*Container animation delay */
}
</style>
