<template>
  <div class="box-wrapper">
    <div
      class="label"
      v-if="bLabel"
      :style="{
        top: bTop + 'px',
        left: bLeft + 'px',
        width: bWidth + 'px',
      }"
    >
      {{ bLabel }}
    </div>
    <a
      class="box-delete"
      v-on:click="removeMyself"
      v-if="bActive"
      :style="{
        top: bTop - 18 + 'px',
        left: bLeft + bWidth + 'px',
      }"
    >
      x
    </a>
    <div
      class="box"
      :style="{
        position: 'absolute',
        top: `${bTop}px`,
        left: `${bLeft}px`,
        width: `${bWidth}px`,
        height: `${bHeight}px`,
      }"
      v-bind:class="{ active: bActive }"
      @mousedown="selectBox"
    >
      <div class="box-line box-top-line"></div>
      <div class="box-line box-right-line"></div>
      <div class="box-line box-left-line"></div>
      <div class="box-line box-bottom-line"></div>

      <div class="box-corner box-top-left-corner"></div>
      <div class="box-corner box-top-right-corner"></div>
      <div class="box-corner box-bottom-right-corner"></div>
      <div class="box-corner box-bottom-left-corner"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BoxLabel",
  props: [
    "b-top",
    "b-left",
    "b-width",
    "b-height",
    "b-label",
    "on-select",
    "b-active",
    "b-index",
    "on-delete",
  ],
  methods: {
    selectBox() {
      this.onSelect(this.bIndex);
    },
    removeMyself() {
      this.onDelete(this.bIndex);
    },
  },
};
</script>

<style lang="scss" scoped>
.box {
  position: absolute;
  // border: 2px rgba(43, 55, 210, 0.8) solid;

  &:hover {
    // border: 2px rgba(43, 55, 210, 0.8) solid;
    background-color: rgba(88, 96, 213, 0.2);
  }
  &.active {
    // border: 2px rgba(105, 34, 176, 0.8) solid;
    background-color: rgba(207, 66, 242, 0.2);
  }

  z-index: 3;
}
.box .box-line {
  position: absolute;
  transition: all 0.25s;
}
.box:hover .box-line {
  background-color: rgba(88, 96, 213, 0.2);
}
.box .box-top-line {
  top: 0;
  left: 0;
  right: 0;
  height: 5px; /* 5px for the mouse cursor update size */
  border-top: 2px rgba(43, 55, 210, 0.8) solid;
  cursor: n-resize;
}
.box .box-bottom-line {
  bottom: 0;
  left: 0;
  right: 0;
  height: 5px; /* 5px for the mouse cursor update size */
  border-bottom: 2px rgba(43, 55, 210, 0.8) solid;
  cursor: s-resize;
}
.box .box-left-line {
  top: 0;
  left: 0;
  bottom: 0;
  width: 5px; /* 5px for the mouse cursor update size */
  border-left: 2px rgba(43, 55, 210, 0.8) solid;
  cursor: w-resize;
}
.box .box-right-line {
  top: 0;
  right: 0;
  bottom: 0;
  width: 5px;
  border-right: 2px rgba(43, 55, 210, 0.8) solid;
  cursor: e-resize;
}
.box .box-corner {
  position: absolute;
  width: 6px;
  height: 6px;
  border-radius: 2px;
  border: 1px solid #808080;
  opacity: 0;
  transition: all 0.25s;
}
.box .box-top-left-corner {
  top: -3px;
  left: -3px;
  cursor: nw-resize;
}
.box .box-top-right-corner {
  top: -3px;
  right: -3px;
  cursor: ne-resize;
}
.box .box-bottom-left-corner {
  bottom: -3px;
  left: -3px;
  cursor: sw-resize;
}
.box .box-bottom-right-corner {
  bottom: -3px;
  right: -3px;
  cursor: se-resize;
}
.box:hover .box-corner {
  opacity: 1;
}
.label {
  position: absolute;
  height: 22px;
  font-size: 16px;
  color: #000;
  font-weight: bold;
  background-color: rgba(105, 34, 176, 0.5);
  z-index: 4;
}
.box-delete {
  position: absolute;
  z-index: 6;
  font-weight: bold;
  color: rgb(146, 7, 7);
  cursor: pointer;
  font-size: 24px;
  font-weight: bold;
}
</style>
