<script lang="ts" setup>
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import { useIntervalFn } from "@vueuse/core";
import * as echarts from "echarts";
import type { EChartsType } from "echarts";

const dataArr = [135, 147, 150, 218, 224, 230, 260];
const everyTime = 1;
let dataNum = 0;
const chartRef = ref<HTMLDivElement>();
let chart: EChartsType | null = null;
let isInChart = false;
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
    { type: "inside", filterMode: "empty", xAxisIndex: [0] },
    { type: "slider", filterMode: "empty", xAxisIndex: [0] }
  ],
  xAxis: {
    type: "category",
    data: [],
    interval: 0,
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
  const { length } = options.series.data;
  const startValue = length > 6 ? length - 6 : 0;
  const newOption: any = {
    xAxis: options.xAxis,
    series: options.series
  };

  if (isInChart) {
    newOption.dataZoom = [
      { type: "inside", filterMode: "empty", xAxisIndex: [0] },
      { type: "slider", filterMode: "empty", xAxisIndex: [0] }
    ];
  } else {
    newOption.dataZoom = [
      {
        type: "inside",
        filterMode: "empty",
        xAxisIndex: [0],
        startValue,
        endValue: length
      },
      {
        type: "slider",
        filterMode: "empty",
        xAxisIndex: [0],
        startValue,
        endValue: length
      }
    ];
  }
  chart?.setOption(newOption);
}, 500);
const setIsInChart = (state: boolean) => {
  isInChart = state;
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
  <div ref="chartRef" class="charts-box" @mouseover="setIsInChart(true)" @mouseout="setIsInChart(false)" />
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
</style>
