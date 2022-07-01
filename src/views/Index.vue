<script lang="ts" setup>
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import { useIntervalFn } from "@vueuse/core";
import * as echarts from "echarts";
import type { EChartsType } from "echarts";

const dataArr = [135, 147, 150, 218, 224, 230, 260];
const everyTime = 1;
const max = 5;
let dataNum = 0;
const chartRef = ref<HTMLDivElement>();
let chart: EChartsType | null = null;
const options: any = {
  animation: false,
  tooltip: {
    trigger: "axis",
    showContent: false,
    axisPointer: {
      type: "cross",
      label: { backgroundColor: "#6a7985" }
    }
  },
  dataZoom: [
    {
      type: "inside",
      filterMode: "empty",
      xAxisIndex: [0],
      maxValueSpan: max
    },
    {
      type: "slider",
      filterMode: "empty",
      xAxisIndex: [0],
      maxValueSpan: max
    }
  ],
  xAxis: {
    type: "category",
    data: [],
    interval: 0,
    scale: true,
    axisLabel: {
      showMaxLabel: true,
    }
  },
  yAxis: { type: "value" },
  series: {
    data: [],
    type: "line",
    symbol: "none"
  }
};

const { pause: stopAddData, resume: addData } = useIntervalFn(() => {
  for (let i = 0; i < everyTime; i++) {
    const random = dataArr[Math.floor(Math.random() * dataArr.length)];
    options.xAxis.data.push(`第${++dataNum}条`);
    options.series.data.push(random);
  }
  chart?.setOption({
    xAxis: options.xAxis,
    series: options.series
  });
}, 500);
const toEnd = () => {
  chart?.dispatchAction({
    type: "dataZoom",
    end: 80
  });
};

onMounted(async () => {
  await nextTick();
  chart = echarts.init(chartRef.value!, undefined, {
    renderer: "canvas",
    useDirtyRect: true
  });
  chart.setOption(options);
  addData();
});
onBeforeUnmount(() => {
  stopAddData();
  chart?.dispose();
});
</script>

<template>
  <div>
    <div ref="chartRef" class="charts-box" />
    <span class="btn-primary" @click="toEnd">移动</span>
  </div>
</template>

<style lang="scss" scoped>
.charts-box {
  width: 80vw;
  height: 80vh;
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  border: 5px dashed #ccc;
}
.btn-primary {
  background-color: #66ccff;
  color: #fff;
  font-size: 12px;
  padding: 10px 20px;
  border-radius: 5px;
  display: inline-block;
  cursor: pointer;
  user-select: none;
}
</style>
