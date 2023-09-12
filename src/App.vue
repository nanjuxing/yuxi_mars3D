<template>
  <div id="popup"></div>
  <MarsMap :url="configUrl" map-key="test" @onload="marsOnload" />
</template>

<script setup lang="ts">
import * as mars3d from "mars3d";
import MarsMap from "./components/mars-work/mars-map.vue";

const configUrl = "config/config.json";

const Cesium = mars3d.Cesium;

const marsOnload = (map: any) => {
  const graphicLayer = new mars3d.layer.GraphicLayer();
  map.addLayer(graphicLayer, mars3d);
  var popup = document.getElementById("popup");
  document.addEventListener("mousemove", function (event) {
    var popup = document.getElementById("popup");

    // 获取鼠标的位置
    var mouseX = event.clientX;
    var mouseY = event.clientY;

    // 设置弹框的位置
    popup.style.left = mouseX + 10 + "px"; // 在鼠标右侧偏移10像素
    popup.style.top = mouseY + 10 + "px"; // 在鼠标下方偏移10像素
  });

  let selectGraphic: any[] = [];
  function updateSelect(drawGraphic: any) {
    graphicLayer.eachGraphic((graphic) => {
      const position = graphic.positionShow;

      const isInArea = drawGraphic.isInPoly(position);
      if (isInArea) {
        selectGraphic.push(graphic);
      }
    });

    // 2.在layer上绑定监听事件
    graphicLayer.on(mars3d.EventType.click, function (_event) {
      // console.log("监听layer，单击了矢量对象，在范围内", event);
    });
    graphicLayer.on(mars3d.EventType.mouseOver, function (_event) {
      // console.log("监听layer，鼠标移入了矢量对象", event);
      popup.style.display = "block";
      popup.textContent = "在范围内";
    });
    graphicLayer.on(mars3d.EventType.mouseOut, function (_event) {
      // console.log("监听layer，鼠标移出了矢量对象，超出了范围", event);
      popup.style.display = "block";
      popup.textContent = "超出范围了";
    });
  }
  graphicLayer.startDraw({
    type: "polygon",
    style: {
      color: "#9d364e",
      opacity: 0.6,
      clampToGround: true,
      highlight: {
        opacity: 0.4,
        color: "#22a878",
      },
    },
    success: function (graphic: any) {
      updateSelect(graphic);
    },
  });

  // 可在图层绑定右键菜单,对所有加到这个图层的矢量数据都生效
  graphicLayer.bindContextMenu([
    {
      text: "删除对象",
      callback: function (e: any) {
        const graphic = e.graphic;
        if (graphic) {
          graphicLayer.removeGraphic(graphic);
        }
      },
    },
  ]);
  // 在图层绑定弹窗
  // function bindTooltip(text: string) {
  //   graphicLayer.bindTooltip(function (event) {
  //     const attr = event.graphic.attr || {};
  //     attr["提示"] = text;

  //     return mars3d.Util.getTemplateHtml({
  //       title: "矢量图层",
  //       template: "all",
  //       attr: attr,
  //     });
  //   });
  // }
  // bindTooltip("在范围内");
};
</script>

<style scoped>
#popup {
  position: fixed;
  width: 100px;
  height: 40px;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius: 5%;
  text-align: center;
  z-index: 9999;
  display: none;
  color:aliceblue
}
</style>
