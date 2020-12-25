<!-- 热销商品图表 -->
<template>
  <div class="com-container">
    <div class="com-chart" ref="hot_ref"></div>
    <span class="iconfont arr-left" @click="toLeft" :style="comStyle"
      >&#xe6ef;</span
    >
    <span class="iconfont arr-right" @click="toRight" :style="comStyle"
      >&#xe6ed;</span
    >
    <span class="cat-name" :style="comStyle">{{ catName }}</span>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { getThemeValue } from "@/utils/theme_utils";
export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      currentIndex: 0, // 当前所展示出的一级分类数据
      titleFontSize: 0
    };
  },
  created() {
    // 在组件创建完成之后 进行回调函数的注册
    this.$socket.registerCallBack("hotData", this.getData);
  },
  computed: {
    catName() {
      if (!this.allData) {
        return "";
      } else {
        return this.allData[this.currentIndex].name;
      }
    },
    comStyle() {
      return {
        fontSize: this.titleFontSize + "px",
        color: getThemeValue(this.theme).titleColor
      };
    },
    ...mapState(["theme"])
  },
  mounted() {
    this.initChart();
    this.startTimer();
    // this.getData()
    this.$socket.send({
      action: "getData",
      socketType: "hotData",
      chartName: "hot",
      value: ""
    });
    window.addEventListener("resize", this.screenAdapter);
    this.screenAdapter();
  },
  destroyed() {
    window.removeEventListener("resize", this.screenAdapter);
    this.$socket.unRegisterCallBack("hotData");
  },
  methods: {
    initChart() {
      const color = [
        "#00ffff",
        "#00cfff",
        "#006ced",
        "#ffe000",
        "#ffa800",
        "#ff5b00",
        "#ff3000",
        "#00FF7F"
      ];

      this.chartInstance = this.$echarts.init(this.$refs.hot_ref, this.theme);
      const initOption = {
        color: color,
        title: {
          text: "▎ 热销商品的占比",
          left: 20,
          top: 20
        },
        // graphic: {
        //   elements: [
        //     {
        //       type: "image",
        //       z: 5,
        //       style: {
        //         // image: img,
        //         width: 178,
        //         height: 178
        //       },
        //       left: "center",
        //       top: "center",
        //       position: [100, 100]
        //     }
        //   ]
        // },

        series: [
          {
            type: "pie",
            center: ["50%", "50%"],
            label: {
              show: false
            },
            emphasis: {
              label: {
                show: true
              },
              labelLine: {
                show: false
              }
            }
          },
          {
            type: "pie",
            // radius: ['44%', '45%'],
            center: ["50%", "50%"],
            hoverAnimation: false,
            clockWise: false,
            itemStyle: {
              normal: {
                color: "#0C355E"
              }
            },
            label: {
              show: false
            },
            data: this._dashed()
          }
        ]
      };
      this.chartInstance.setOption(initOption);
    },
    getData(ret) {
      // 获取服务器的数据, 对this.allData进行赋值之后, 调用updateChart方法更新图表
      // const { data: ret } = await this.$http.get('hotproduct')
      this.allData = ret;
      console.log(this.allData);
      this.updateChart();
    },
    doing() {
      let option = this.chartInstance.getOption();
      option.series[1].startAngle = option.series[1].startAngle - 1;
      this.chartInstance.setOption(option);
    },
    startTimer() {
      setInterval(this.doing, 500);
    },
    _dashed() {
      let dataArr = [];
      for (var i = 0; i < 60; i++) {
        if (i % 2 === 0) {
          dataArr.push({
            name: (i + 1).toString(),
            value: 20,
            itemStyle: {
              normal: {
                color: "#23E5E5"
              }
            }
          });
        } else {
          dataArr.push({
            name: (i + 1).toString(),
            value: 20,
            itemStyle: {
              normal: {
                color: "rgb(0,0,0,0)",
                borderWidth: 1,
                borderColor: "#2E72BF"
              }
            }
          });
        }
      }
      return dataArr;
    },
    updateChart() {
      const color = [
        "#00ffff",
        "#00cfff",
        "#006ced",
        "#ffe000",
        "#ffa800",
        "#ff5b00",
        "#ff3000"
      ];
      // 处理图表需要的数据
      const legendData = this.allData[this.currentIndex].children.map(item => {
        return item.name;
      });
      const totalValue = this.allData[this.currentIndex].value;
      const seriesData = this.allData[this.currentIndex].children.map(item => {
        return {
          name: item.name,
          value: item.value,
          children: item.children // 新增加children的原因是为了在tooltip中的formatter的回调函数中,来拿到这个二级分类下的三级分类数据
        };
      });
      let data = [];
      for (var i = 0; i < seriesData.length; i++) {
        data.push(
          {
            value: seriesData[i].value,
            name: seriesData[i].name,
            itemStyle: {
              normal: {
                borderWidth: 5,
                shadowBlur: 20,
                borderColor: color[i],
                shadowColor: color[i]
              }
            }
          },
          {
            value: 10000,
            name: "",
            itemStyle: {
              normal: {
                label: {
                  show: false
                },
                labelLine: {
                  show: false
                },
                color: "rgba(0, 0, 0, 0)",
                borderColor: "rgba(0, 0, 0, 0)",
                borderWidth: 0
              }
            }
          }
        );
      }
      const dataOption = {
        // legend: {
        //   data: legendData
        // },
        series: [
          // { data: this._dashed() },
          {
            data: data,
            clockWise: false,
            hoverAnimation: false,
            itemStyle: {
              normal: {
                label: {
                  show: true,
                  position: "outside",
                  // color: "#D3D3D3",
                  formatter: function(params) {
                    let percent = 0;
                    let total = 0;
                    for (let i = 0; i < seriesData.length; i++) {
                      total += seriesData[i].value;
                    }
                    percent = ((params.value / total) * 100).toFixed(0);
                    if (params.name !== "") {
                      return params.name + "\n" + "\n" + percent + "%";
                    } else {
                      return "";
                    }
                  }
                }
              }
            }
            // radius: [55, 70]
          }
        ]
      };
      this.chartInstance.setOption(dataOption);
    },
    screenAdapter() {
      this.titleFontSize = (this.$refs.hot_ref.offsetWidth / 100) * 3.3;
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: this.titleFontSize
          }
        },

        series: [
          {
            radius: [4 * this.titleFontSize, 5 * this.titleFontSize],
            center: ["50%", "50%"],
            itemStyle: {
              normal: {
                label: {
                  fontSize: this.titleFontSize / 1.3
                },
                labelLine: {
                  show: true,
                  color: "#00ffff"
                }
              }
            }
          },
          {
            radius: [3.2 * this.titleFontSize, 3.5 * this.titleFontSize]
          }
        ]
      };
      this.chartInstance.setOption(adapterOption);
      this.chartInstance.resize();
      this.startTimer();
    },
    toLeft() {
      this.currentIndex--;
      if (this.currentIndex < 0) {
        this.currentIndex = this.allData.length - 1;
      }
      this.updateChart();
    },
    toRight() {
      this.currentIndex++;
      if (this.currentIndex > this.allData.length - 1) {
        this.currentIndex = 0;
      }
      this.updateChart();
    }
  },
  watch: {
    theme() {
      console.log("主题切换了");
      this.chartInstance.dispose(); // 销毁当前的图表
      this.initChart(); // 重新以最新的主题名称初始化图表对象
      this.screenAdapter(); // 完成屏幕的适配
      this.updateChart(); // 更新图表的展示
    }
  }
  // mounted() {
  //   this.startTimer();
  // }
};
</script>

<style lang="less" scoped>
.arr-left {
  position: absolute;
  left: 7%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color: white;
}
.arr-right {
  position: absolute;
  right: 7%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color: white;
}
.cat-name {
  position: absolute;
  left: 80%;
  bottom: 20px;
  color: white;
}
</style>
