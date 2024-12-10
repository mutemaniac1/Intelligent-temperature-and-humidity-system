<template>
  <div ref="chart" style="width: 100%; height: 300px;"></div>
</template>

<script>
import * as echarts from "echarts";

export default {
  props: {
    humidityData: Array, // 湿度数据
    secondaryHumidityData: Array, // 全量数据对应的湿度
    timeStamps: Array, // 时间戳
    ids: String // 设备ID
  },
  data() {
    return {
      chart: null
    };
  },
  methods: {
    initChart() {
      if (!this.chart) {
        this.chart = echarts.init(this.$refs.chart);
      }

      // 获取实时湿度数据
      const realTimeData = this.humidityData;

      // 获取时间戳，确保长度与湿度数据一致
      const realTimeTimestamps = this.timeStamps.slice(-realTimeData.length); // 最后N个时间戳

      // 合并实时湿度数据和全量湿度数据（根据需求，您可以选择是否需要全量数据）
      const combinedData = realTimeData.concat(this.secondaryHumidityData); // 合并实时和全量数据

      // 包装数据，添加标识以及对应时间戳
      const wrappedData = combinedData.map((item, index) => {
        return {
          value: item,
          type: index < 300 ? '实时数据' : '预测数据', // 标识数据类型
          timestamp: realTimeTimestamps[index] // 对应时间戳
        };
      });

      // 配置图表选项
      const options = {
        title: {
          text: `[${this.ids}] 湿度趋势图`,
          left: "center"
        },
        tooltip: {
          trigger: "axis",
          formatter: function (params) {
            let tip = '';
            params.forEach((param) => {
              tip += `${param.seriesName} - ${param.data.value}% (${param.data.type}) 时间: ${param.data.timestamp}<br>`;
            });
            return tip;
          }
        },
        xAxis: {
          type: "category",
          data: realTimeTimestamps, // 使用实时数据的时间戳
          axisLabel: {
            rotate: 45,
            fontSize: 10
          }
        },
        yAxis: {
          type: "value",
          axisLabel: {
            formatter: "{value}%" // 显示湿度百分比
          }
        },
        series: [
          {
            name: "湿度",
            type: "line",
            data: wrappedData,
            smooth: true,
            lineStyle: {
              color: "#1E90FF" // 湿度折线颜色
            },
            areaStyle: {
              opacity: 0.2,
              color: "#1E90FF"
            },
            markLine: {
              symbol: 'circle', // 标记点样式
              symbolSize: 10, // 标记点大小
              data: [
                { 
                  xAxis: 300, // 在第300个点的位置添加标记
                  label: {
                    show: true,
                    position: 'middle',
                    formatter: '预测' ,// 显示的名称
                    rotate: 0, // 确保文字方向不旋转
       
                  }
                }
              ]
            }
          }
        ]
      };

      this.chart.setOption(options);
    }
  },
  watch: {
    humidityData() {
      this.initChart();
    },
    secondaryHumidityData() {
      this.initChart();
    },
    timeStamps() {
      this.initChart();
    }
  },
  mounted() {
    this.initChart();
  }
};
</script>
