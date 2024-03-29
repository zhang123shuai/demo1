<template>
  <div class="warp">
    <div id="map" ref="rootmap">
      <!-- 坐标展示位置div -->
      <div id="mouse_position"></div>
    </div>
    <div class="drawselect">
      <el-button
        v-for="(item, i) in drawtool"
        :key="i"
        :type="item.value == 'None' ? 'danger' : 'primary'"
        @click="drawingStart(item.value)"
        round
        >{{ item.label }}</el-button
      >
    </div>
  </div>
</template>
 
<script>
import "ol/ol.css";
import { Map, View } from "ol";
import TileLayer from "ol/layer/Tile";
import VectorLayer from "ol/layer/Vector";
import { OSM, TileWMS, Vector } from "ol/source";
import GeoJSON from "ol/format/GeoJSON";
import { Fill, Style, Stroke } from "ol/style";
import { DoubleClickZoom, Draw } from "ol/interaction";
import mapconfig from "../config/mapconfig.js";
import MousePosition from "ol/control/MousePosition";
import { createStringXY } from "ol/coordinate";
export default {
  data() {
    return {
      drawtool: [
        {
          value: "Point",
          label: "点",
        },
        {
          value: "LineString",
          label: "线",
        },
        {
          value: "Polygon",
          label: "多边形",
        },
        {
          value: "Circle",
          label: "圆",
        },
        {
          value: "None",
          label: "清除",
        },
      ],
      map: null,
      drawLayer: {},
      draw: null,
      dblClickInteraction: null,
    };
  },
  mounted() {
    this.initMap();
    this.initMouse();
    this.loopLayer();
  },
  methods: {
    // 初始化地图
    initMap() {
      var mapcontainer = this.$refs.rootmap;
      this.map = new Map({
        target: mapcontainer,
        layers: mapconfig.streetmap(),
        view: new View({
          projection: "EPSG:4326", //使用这个坐标系
          center: [mapconfig.x, mapconfig.y], //深圳
          zoom: mapconfig.zoom,
        }),
      });
    },
    //加入鼠标当前坐标事件
    initMouse() {
      var mousePositionControl = new MousePosition({
        coordinateFormat: createStringXY(6), //获取位置
        projection: "EPSG:4326",
        className: "custom-mouse-position",
        target: document.getElementById("mouse_position"), //将位置数据放到那里
        undefinedHTML: "&nbsp",
      });
      this.map.addControl(mousePositionControl);
    },
    // 初始化工具图层
    loopLayer() {
      // 将图形的数据层包上一层图层放入地图
      // 点层样式
      this.drawLayer.Point = new VectorLayer({
        source: new Vector({ wrapX: false }),
        zIndex: 9,
        // style: new Style({
        //   // 设置线颜色\宽度
        //   stroke: new Stroke({
        //     width: 4,
        //     color: "#119aff",
        //   }),
        // }),
      });
      // 线层 样式
      this.drawLayer.LineString = new VectorLayer({
        source: new Vector({ wrapX: false }),
        zIndex: 9,
        style: new Style({
          // 设置线颜色\宽度
          stroke: new Stroke({
            width: 4,
            color: "#119aff",
          }),
        }),
      });
      // 图形层 样式
      this.drawLayer.Polygon = new VectorLayer({
        source: new Vector({ wrapX: false }),
        zIndex: 9,
        style: new Style({
          // 设置线颜色\宽度
          stroke: new Stroke({
            width: 4,
            color: "#119aff",
          }),
          // 图形区域内颜色
          fill: new Fill({
            color: "rgba(57,160,255,0.5)",
          }),
        }),
      });
      // 圆 样式
      this.drawLayer.Circle = new VectorLayer({
        source: new Vector({ wrapX: false }),
        zIndex: 9,
        style: new Style({
          // 设置线颜色\宽度
          stroke: new Stroke({
            width: 4,
            color: "#119aff",
          }),
          // 图形区域内颜色
          fill: new Fill({
            color: "rgba(57,160,255,0.5)",
          }),
        }),
      });
      // 点线面图层放入地图盒子
      for (let k in this.drawLayer) {
        this.map.addLayer(this.drawLayer[k]);
      }
    },
    // 删除默认的双击事件
    deleteDoubleclick() {
      this.dblClickInteraction = this.map
        .getInteractions()
        .getArray()
        .find((interaction) => {
          return interaction instanceof DoubleClickZoom;
        });
      this.map.removeInteraction(this.dblClickInteraction);
    },
    // 开始绘制
    drawingStart(type) {
      this.deleteDoubleclick();
      if (type == "None") {
        this.drawtool.forEach((item, i) => {
          if (item.value != "None") {
            this.drawLayer[item.value].getSource().clear();
          }
        });
      } else {
        let that = this;
        this.draw = new Draw({
          source: this.drawLayer[type].getSource(),
          type: type,
          // style: new Style({
          //   stroke: new Stroke({
          //     color: "blue",
          //     width: 3,
          //   }),
          // }),
        });
        this.map.addInteraction(this.draw);
        this.draw.on("drawend", (evt) => {
          that.drawingEnd(evt);
        });
      }
    },
    // 绘制结束
    drawingEnd(evt) {
      let that = this;
      const geo = evt.feature.getGeometry();
      const t = geo.getType();
      console.log(t, "所画形状");
      // 获取坐标点
      const points = geo.getCoordinates();
      this.map.removeInteraction(this.draw); // 移除画笔绘制
      // 绘制后添加 利用时间差来避免绘制结束后双击放大地图
      setTimeout(() => {
        this.map.addInteraction(this.dblClickInteraction);
      }, 100);
      console.warn(points, "绘制结束，点坐标");
    },
  },
};
</script>
 
<style lang='scss' scoped>
.warp {
  height: 100%;
  #map {
    height: 100%;
    #mouse_position {
      color: #fff;
      position: absolute;
      bottom: 10px;
      right: 10px;
      z-index: 10000000;
      width: 200px;
      line-height: 30px;
      background: rgba(0, 0, 0, 0.5);
    }
  }
  .drawselect {
    position: fixed;
    top: 0;
    right: 0;
  }
}
/*隐藏ol的一些自带元素*/
.ol-attribution,
.ol-zoom {
  display: none;
}
</style>