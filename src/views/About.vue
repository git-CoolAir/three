<template>
  <div class="about">
    <div class="three" ref="three"></div>
  </div>
</template>

<script>
import "@/scss/About/index.scss";
// import * as THREE from "three";
export default {
  data() {
    return {
      Colors: {
        red: 0xf25346,

        white: 0xd8d0d1,

        brown: 0x59332e,

        pink: 0xf5986e,

        brownDark: 0x23190f,

        blue: 0x68c3c0,
      },
    };
  },
  methods: {
    init() {
      //创建绘画容器
      // let canvas = this.$refs.three;
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style lang="scss" scoped>
.three {
  position: absolute;

  width: 100%;

  height: 100%;

  overflow: hidden;

  background: linear-gradient(#e4e0ba, #f7d9aa);
}
</style>