<script setup lang="ts">
import { ref, computed } from 'vue'
import dayjs from 'dayjs'
import type { Dayjs } from 'dayjs'
import Timeline from './components/Timeline.vue'

interface UnitOption {
  value: 'sec' | 'min' | 'hour'
  label: string
}

const backward = ref<'0' | '1'>('0');
const zero_time = new Date("2025/1/1 00:00:00");
const time_start = ref<Dayjs>();
const time_end = ref<Dayjs>();
const time_point = ref<Dayjs>();
const time_diff = ref<Dayjs>();
const time_interval = computed(() => {
  if (!time_end.value || !time_start.value) {
    return ``;
  }
  else {
    let sec = Math.floor((time_end.value.valueOf() - time_start.value.valueOf()) / 1000);
    if (unit.value == 'sec') { return `${sec} s` }
    if (unit.value == 'min') {
      let min = Math.floor(sec / 60);
      sec -= min * 60;
      return `${min} min,${sec} s`;
    }
    if (unit.value == 'hour') {
      let hour = Math.floor(sec / 3600);
      sec -= hour * 3600;
      let min = Math.floor(sec / 60);
      sec -= min * 60;
      return `${hour} hour, ${min} min,${sec} s`;
    }
    return '';
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

    if (backward.value == '0') {
      return baseTime
        .add(hours, 'hour')
        .add(minutes, 'minute')
        .add(seconds, 'second')
        .format('HH:mm');
    }
    else {
      return baseTime
        .subtract(hours, 'hour')
        .subtract(minutes, 'minute')
        .subtract(seconds, 'second')
        .format('HH:mm');
    }
  }
});

const unit_options: UnitOption[] = [
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
];

const timelineItems = ref([
  {
    title: '项目启动',
    date: '2023-01-15',
    description: '召开项目启动会议，确定项目目标和范围',
    tags: ['规划', '会议']
  },
  {
    title: '需求分析完成',
    date: '2023-02-10',
    description: '完成用户需求收集和分析，输出需求文档',
    tags: ['需求', '文档']
  },
  {
    title: 'UI设计定稿',
    date: '2023-03-05',
    description: '完成所有页面的UI设计并获得客户确认',
    tags: ['设计', '评审']
  },
  {
    title: '开发阶段开始',
    date: '2023-03-20',
    description: '前端和后端开发工作正式开始',
    tags: ['开发', '里程碑']
  }
]);

const unit = ref<'sec' | 'min' | 'hour'>('min');
</script>

<template>
  <p>时间间隔计算</p>
  <div>
    <div class="time-picker">
      <el-time-picker v-model="time_start" placeholder="Start Time" />
      <el-time-picker v-model="time_end" placeholder="End Time" />
    </div>
    <div class="time-output">
      <el-select v-model="unit" placeholder="Unit" size="large" style="width: 90px">
        <el-option v-for="item in unit_options" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
      <p class="time-point">Time Interval:{{ time_interval }}</p>
    </div>
  </div>

  <div>
    <p>时间推算</p>
    <el-time-picker v-model="time_point" placeholder="Time Point" format="H:mm" />
    <el-time-picker v-model="time_diff" placeholder="Arbitrary time" :default-value="zero_time" format="H:mm" />
    <div class="select-button">
      <el-radio-group v-model="backward">
        <el-radio-button value="0" size="large">向后</el-radio-button>
        <el-radio-button value="1" size="large">向前</el-radio-button>
      </el-radio-group>
    </div>
    <p class="time-point">新的时间点:{{ new_time_point }}</p>
  </div>

  <div>
    <Timeline :items="timelineItems" marker-color="#4CAF50" line-color="#9E9E9E"></Timeline>
  </div>
</template>
<!-- TODO : Time Allocator  -->

<!--  
- 0:00      <--begin
- 1:00          description
- 2:00          3h
  - 2:15
  - 2:20
  - 2:25    <-- sub-begin
  - 2:30        5min
  - 2:35    <-- sub-end
  - 2:40         
  - 2:45    <-- sub-begin   5min
  - 2:50    <-- sub-end
  - 2:55
- 3:00      <--end
- 4:00
- 5:00
- 6:00      <--begin     descrpition
- 7:00      <--end       1hour
- 8:00
- 9:00
- 10:00
...

如何实现时间分配器?  可以放置时间游标,自动响应时间间隔
  如何添加时间刻度组件?
 -->
<style scoped>
.time-picker {
  margin: 8px;
}

.time-point {
  margin: 8px;
  font-size: 16px;
  color: #409eff;
}
</style>