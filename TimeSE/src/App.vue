<script setup>
import { ref, computed } from 'vue'
import dayjs from 'dayjs'

const backward = ref('0');
const zeor_time = new Date("2025/1/1 00:00:00");
const time_start = ref();
const time_end = ref();
const time_point = ref();
const time_diff = ref();
const time_interval = computed(() => {
  if (!time_end.value || !time_start.value) {
    return ``;
  }
  else {
    let sec = Math.floor((time_end.value - time_start.value) / 1000);
    if (unit.value == 'sec') { return `${sec} s` }
    if (unit.value == 'min') {
      let min = Math.floor(sec / 60);
      sec -= min * 60;
      return `${min} min,${sec} s`;

      //Why NaN?  Because You Returned diff.val , which is not the real diff ms
      //Bad name diff
    }
    if (unit.value == 'hour') {
      let hour = Math.floor(sec / 3600);
      sec -= hour * 3600;
      let min = Math.floor(sec / 60);
      sec -= min * 60;
      return `${hour} hour, ${min} min,${sec} s`;
    }
  };
});

const new_time_point = computed(() => {
  if (!time_point.value || !time_diff.value) {
    return '';
  } else {
    const baseTime = dayjs(time_point.value);
    const diffTime = dayjs(time_diff.value);

    const hours = diffTime.hour();
    const minutes = diffTime.minute();
    const seconds = diffTime.second();

    if (backward.value == 0) {
      return baseTime
        .add(hours, 'hour')
        .add(minutes, 'minute')
        .add(seconds, 'second')
        .format('HH:mm:ss');
    }
    else {
      return baseTime
        .subtract(hours, 'hour')
        .subtract(minutes, 'minute')
        .subtract(seconds, 'second')
        .format('HH:mm:ss');
    }
  }
});
const unit_options = [
  {
    value: 'sec',
    label: 'sec',
  },
  {
    value: 'min',
    label: 'min',
  },
  {
    value: 'hour',
    label: 'hour',
  },
]

const unit = ref('min');
</script>
<template>
  <div>
    <div class="time-picker">
      <el-time-picker v-model="time_start" placeholder="Start Time" />
      <el-time-picker v-model="time_end" placeholder="End Time" />
    </div>
    <div class="time-output">
      <el-select v-model="unit" placeholder="Unit" size="large" style="width: 90px">
        <el-option v-for="item in unit_options" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
      <a>Time Interval:{{ time_interval.valueOf() }}</a>
    </div>
  </div>

  <div>
    <el-time-picker v-model="time_point" placeholder="Time Point" />
    <el-time-picker v-model="time_diff" placeholder="Arbitrary time" :default-value="zeor_time" />
    <div class="mb-2 ml-4">
      <el-radio-group v-model="backward">
        <el-radio-button value="0" size="large">向后</el-radio-button>
        <el-radio-button value="1" size="large">向前</el-radio-button>
      </el-radio-group>
    </div>
    <a>{{ new_time_point }}</a>
  </div>
</template>

<!-- TODO: 优化界面布局 -->
<style scoped>
.time-picker {
  margin: 8px;
}
</style>
