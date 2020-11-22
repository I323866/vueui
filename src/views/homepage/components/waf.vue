<template>
  <div class="chart-wrapper" ref="cityCount"></div>
</template>

<script>
import echarts from "echarts";
import { onMounted, onBeforeUnmount, ref, watch } from "vue";
import { debounce } from "@/utils/index.js";

import useResize from "@/componentApi/useResize.js";
import { selectCityData } from "@/api/chart";

export default {
  name: "cityCount",
  setup() {
    let { abcode, year, parentInfo, routerChange } = useResize();

    const cityCount = ref(null);
    let myChart = ref(null);

    const resizeHandler = debounce(() => {
      if (myChart) {
        myChart.resize();
      }
    }, 200);

    onMounted(() => {
      getChartData();
      window.addEventListener("resize", resizeHandler);
    });
    onBeforeUnmount(() => {
      window.removeEventListener("resize", resizeHandler);
    });

    //模拟接口，获取echarts数据
    const getChartData = async () => {
      const { data } = await selectCityData({
        abcode,
        year,
      });
      let xData = [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23];
      let pData = [0,0,0,0,0,16,30,28,24,25,24,23,23,22,24,22,23,20,22,20,21,22,23,1];
      let rData = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,0];
      // data.forEach((item) => {
      //   xData.push(item.name);
      //   yData.push(item.value);
      // });
      initEcharts(xData, pData, rData);
    };

    //渲染echarts图
    const initEcharts = (xData, pData ,rData) => {
      myChart = echarts.init(cityCount.value);
      myChart.setOption(
        {
          grid: {
            left: "13%",
            right: "8%",
            bottom: "25%",
            top: "10%",
          },
          title: {
            show: xData.length === 0,
            top: "center",
            left: "center",
            text: "暂无数据",
            textStyle: {
              color: "rgb(179, 239, 255)",
              fontSize: 12,
            },
          },
          tooltip: {
            trigger: "axis",
            axisPointer: {
              type: "shadow",
            },
            formatter: (params) => {
              return "计划架次：" + params[0].value + "<br>实际架次：" + params[1].value ;
            },
          },
          toolbox: {
            feature: {
              dataView: {
                show: true,
              },
              magicType: {
                show: true,
                type: ['line', 'bar', 'stack', 'tiled']
              },
              restore: {
                show: true,
              },
              saveAsImage: {
                show: true,
                name: "航班延误图",
                pixelRatio: 2,
              },
            },
            iconStyle: {
              normal: {
                borderColor: "#1990DA",
              },
            },
            top: 0,
            right: 5,
          },
          dataZoom: [
            {
              type: "inside",
              startValue: 0,
              endValue: 24,
            },
          ],
          xAxis: {
            type: "category",
            axisLine: {
              lineStyle: {
                color: "#397cbc",
              },
            },
            splitLine: {
              show: false,
            },
            axisTick: {
              show: false,
            },
            //轴线上的字
            axisLabel: {
              show: true,
              textStyle: {
                color: "#cecece",
                fontSize: 12,
              },
              rotate: 15,
            },
            data: xData,
          },
          yAxis: [
            {
              type: "value",
              axisTick: {
                show: false,
              },
              //轴线上的字
              axisLabel: {
                textStyle: {
                  fontSize: 12,
                  color: "#cecece",
                },
              },
              axisLine: {
                lineStyle: {
                  color: "#397cbc",
                },
              },
              //网格线
              splitLine: {
                lineStyle: {
                  color: "#11366e",
                },
              },
            },
          ],
          series: [
            {
              name: "plan",
              type: "bar",
              data: pData,
              barWidth: "35%",
              itemStyle: {
                color: new echarts.graphic.LinearGradient(
                  0,
                  0,
                  0,
                  1,
                  [
                    {
                      offset: 0,
                      color: "#01BDF4",
                    },
                    {
                      offset: 1,
                      color: "#083AF6 ",
                    },
                  ],
                  false
                ),
              },
            },
            {
              name: "reality",
              type: "bar",
              data: rData,
              barWidth: "35%",
              itemStyle: {
                color: new echarts.graphic.LinearGradient(
                  0,
                  0,
                  0,
                  1,
                  [
                    {
                      offset: 0,
                      color: "#2E8B57",
                    },
                    {
                      offset: 1,
                      color: "#2E8B57 ",
                    },
                  ],
                  false
                ),
              },
            },
          ],
        },
        true
      );

      myChart.getZr().off("click");
      myChart.getZr().on("click", (params) => {
        const pointInPixel = [params.offsetX, params.offsetY];
        if (myChart.containPixel("grid", pointInPixel) || xData.length === 0) {
          routerChange("/more");
        }
      });
    };

    watch(
      year,
      (nl, ol) => {
        getChartData();
      },
      { lazy: false }
    );
    watch(
      parentInfo,
      (nl, ol) => {
        getChartData();
      },
      { lazy: false, deep: true }
    );

    return {
      cityCount,
    };
  },
};
</script>
