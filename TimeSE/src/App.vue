<script setup>
import { ref, computed } from 'vue'

const time_start = ref(); //ref() const ??
const time_end = ref();
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
const options = [
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
  <div class="time-picker">
    <el-time-picker v-model="time_start" placeholder="Start Time" />
    <el-time-picker v-model="time_end" placeholder="End Time" />
  </div>
  <div class="time-output">
    <el-select v-model="unit" placeholder="Unit" size="large" style="width: 90px">
      <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
    </el-select>
    <a>Time Interval:{{ time_interval.valueOf() }}</a>
  </div>
</template>

<style scoped>
.time-output {
  float: left;
}

.time-picker {
  margin: 8px;
}
</style>
