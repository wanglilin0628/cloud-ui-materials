<template>
  <div :class="$style.root">
    <div ref="myChart" :style="formattedSize" :class="$style.canvasWrap"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts/core'
import {processEchartData} from "@/tools";

export default {
  name: "echartLine",
  props: {
    sourceData: [Array, Object],
    size: [Object],
    axisData: [Object],
  },
  data() {
    return {
      lineData: {},
      lineOption: {},
    }
  },
  mounted() {
    this.createMyChart();
  },
  computed: {
    changedObj() {
      let {size, axisData, sourceData} = this;
      return {size, axisData, sourceData};
    },
    formattedSize() {
      let width = this.size.width.replace("px", "") || 400;
      let height = this.size.height.replace("px", "") || 300;
      return {
        width: `${width}px`,
        height: `${height}px`,
      }
    }
  },
  watch: {
    changedObj: {
      handler() {
        this.$refs.myChart.removeAttribute('_echarts_instance_');
        const thisChart = echarts.init(this.$refs.myChart, this.axisData.theme);
        thisChart.dispose();
        this.createMyChart();
      },
      deep: true,
    },
  },
  beforeDestroy() {
    let thisChart = echarts.init(this.$refs.myChart, this.axisData.theme);
    thisChart.dispose();
    thisChart = null;
  },
  methods: {
    createMyChart() {
      const myChart = this.$refs.myChart;
      this.processLineData(this.sourceData);
      this.initChart(myChart, this.lineOption);
    },
    initChart(chart, config) {
      if (chart) {
        this.$refs.myChart.removeAttribute('_echarts_instance_');
        const thisChart = echarts.init(chart, this.axisData.theme);
        thisChart.setOption(config);
        this.$nextTick(() => {
          thisChart.resize();
        });
      }
    },
    processLineData(data) {
      if (!data) {
        return;
      }
      const [attrDict, xAxisList, yAxisList] = processEchartData(data);
      let multiAxisList = [];
      this.axisData.yAxis = this.axisData.yAxis.replace(/，/g, ",");
      if (this.$env.VUE_APP_DESIGNER) {
        multiAxisList = ['指标1', '指标2', '指标3'];
        this.axisData.xAxis = 'fakeXAxis';
      } else {
        multiAxisList = this.axisData.yAxis.replace(/\s+/g, '').split(',') || [];
        if (multiAxisList.length === 1) {
          const temp = multiAxisList[0];
          multiAxisList = [temp.split('.')[temp.split('.').length - 1]];
        }
      }
      this.axisData.xAxis = this.axisData.xAxis.split('.')[this.axisData.xAxis.split('.').length - 1] || '';
      let legendData = multiAxisList.length > 1 ? multiAxisList : [];
      legendData = this.$env.VUE_APP_DESIGNER ? ['指标1', '指标2', '指标3'] : legendData;
      const seriesData = [];
      multiAxisList.forEach((item) => {
        seriesData.push({
          name: item,
          type: 'line',
          data: attrDict[item],
          showBackground: true,
          label: {
            show: this.axisData.allowShowLabel,
          },
        })
      })
      // 发布部署后，如果字段不合法，加载默认图片
      if (!this.$env.VUE_APP_DESIGNER) {
        if (!xAxisList.includes(this.axisData.xAxis)) {
          this.$emit("startLoading");
          return;
        }
        for (let axis of multiAxisList) {
          if (!yAxisList.includes(axis)) {
            this.$emit("startLoading");
            return;
          }
        }
      }
      this.lineOption = {
        toolbox: {
          show: this.axisData.allowDownload,
          feature: {
            saveAsImage: {},
          }
        },
        legend: {
          show: this.axisData.allowShowLegend,
          top: '5%',
          left: 'center',
          data: legendData,
        },
        tooltip: {
          show: this.axisData.allowShowHint,
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        emphasis: {
          focus: 'series',
        },
        title: {
          text: this.axisData.title,
          textStyle: {
            fontSize: this.axisData.titleFontSize,
            fontStyle: this.axisData.titleFontStyle,
          }
        },
        xAxis: {
          data: attrDict[this.axisData.xAxis],
          name: this.axisData.xAxisTitle || this.axisData.xAxis || '',
          nameLocation: 'end',
          axisLine: {
            show: this.axisData.showXAxisLine,
          },
          axisLabel: {
            show: this.axisData.showXAxisLabel,
            rotate: Number(this.axisData.xAxisLabelRotate)
          },
        },
        yAxis: {
          type: 'value',
          name: this.axisData.yAxisTitle || this.axisData.yAxis || '',
          axisLine: {
            show: this.axisData.showYAxisLine,
          },
          axisLabel: {
            show: this.axisData.showYAxisLabel,
          },
        },
        series: seriesData,
      };
    },
  },

};
</script>

<style module>
.root {
}
.canvasWrap {
  width: 100%;
  height: 100%;
}
</style>
