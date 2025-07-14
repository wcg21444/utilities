<template>
    <div ref="pointContainer" class="point-container" :style="pointStyle">
        <div class="point"></div>
    </div>

</template>

<script setup lang="ts">
//TODO: 出边界也能拖拽
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

const pointContainer = ref<HTMLElement | null>(null);
onMounted(() => {
    if (pointContainer.value) {

        // console.log(`Initial position: X=${x_start.value}, Y=${y_start.value}`);
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
    if (!pointContainer.value) return;
    const target = event.target as HTMLElement;
    if (!(target.parentElement == pointContainer.value)) {
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
    console.log(isDragging.value);
    console.log(x.value, y.value);

    return {
        transform: `translate(${x.value}px, ${y.value}px)`, position: `absolute`
    }
});

</script>

<style scoped>
.point {
    position: absolute;
    width: 50px;
    height: 50px;
    background-color: rgba(255, 0, 0, 0.556);
    border-radius: 50%;
    user-select: none;
    box-shadow: 1px 50px 10px 2px rgba(0, 0, 0, 0.2);
}

.point-container {
    position: relative;
    cursor: pointer;
    background-color: #7350504f;
    transition: 10ms;
    /*Container animation delay */
}
</style>
