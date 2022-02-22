<template>
  <div class="warp">
    <div id="map" ref="rootmap"></div>
    <div class="drawselect">
      <el-button
        v-for="(item, i) in drawtool"
        :key="i"
        :type="item.value == 'None' ? 'danger' : 'primary'"
        @click="drawStart(item.value)"
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
import Draw from "ol/interaction/Draw";
import mapconfig from "../config/mapconfig.js";
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
    };
  },
  mounted() {
    this.initMap();
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
    // 初始化工具图层
    loopLayer() {
      // 将图形的数据层包上一层图层放入地图
      // 点层样式
      this.drawLayer.Point = new VectorLayer({
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
    // 开始绘制
    drawStart(type) {
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