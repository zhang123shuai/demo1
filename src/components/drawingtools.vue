<template>
  <el-select v-model="value" placeholder="请选择">
    <el-option
      v-for="item in drawtool"
      :key="item.value"
      :label="item.label"
      :value="item.value"
    >
      <span style="float: left">{{ item.label }}</span>
      <span style="float: right; color: #8492a6; font-size: 13px">{{
        item.value
      }}</span>
    </el-option>
  </el-select>
</template>

<script>
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
        }
      ],
      value: "",
    };
  },
};
</script>

<style>
</style>