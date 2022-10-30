<template>
  <div class="content">
    <!-- 设置容器ID，宽度1920PX，高度1080PX -->
    <div id="myEchart" style="width: 1920px; height: 1080px"></div>
    <div id="qrcode"></div>
  </div>
</template>

<script>
import $ from "jquery"; /* 引入JQUERY，此处不赘述，用于调用$.get函数。 */
var echarts = require("echarts"); /* 引入echarts模块，此处不赘述 */

export default {
  name: "WorldMap",
  data() {
    return {
      myChart: undefined /* 定义变量 */,
    };
  },
  mounted() {
    $.get("world.json", function (geoJson) {
      /* 请求世界地图的本地json数据 */ this.myChart = echarts.init(
        document.getElementById("myEchart")
      ); /* 初始画布 */
      echarts.registerMap("world", geoJson); /* 注册world地图 */
      this.myChart.setOption({
        /* 设置myChart配置项 */
        tooltip: {
          trigger: "item",
          formatter: "{b}",
        },
        series: [
          {
            type: "map",
            mapType: "world", // 自定义扩展图表类型
            itemStyle: {
              // normal: { label: { show: true } }, // 普通状态下，显示标签
              emphasis: { label: { show: true } }, // 高亮状态下，显示标签（默认）
            },
            roam: true, // 放大缩小、平移
          },
        ],
      });
    });

    let qrcode = new QRCode(document.getElementById("qrcode"), {
      // 显示二维码的元素或该元素的 ID
      text: "你好dfdfdfdee,ychidjckosdd89wuqeyiuhacsiuuqiu28y97e98uqiwdnbcaxhi第十九届2白沙笔记本的静安寺", // 内容可以是文字，链接，邮箱
      width: 150, //设置宽度
      height: 150, // 设置高度
      colorDark: "black", // 设置前景色
      colorLight: "white", // 设置背景色
      // correctLevel: QRCode.CorrectLevel.H, // 设置容错级别，上面有介绍
      correctLevel: 3,
    });
  },
};
</script>
