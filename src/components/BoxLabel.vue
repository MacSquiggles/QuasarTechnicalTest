<template>
  <div class="box-wrapper">
    <!-- The label for the box annotation -->
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
    <!-- A button to delete the box label -->
    <q-btn
      round
      class="box-delete"
      @click.prevent="removeMyself"
      v-if="bActive"
      :style="{
        top: bTop - 20 + 'px',
        left: bLeft + bWidth + 5 + 'px',
      }"
      color="red"
      >X
    </q-btn>
    <!-- Display the box -->
    <div
      class="box"
      :style="{
        position: `absolute`,
        top: `${bTop}px`,
        left: `${bLeft}px`,
        width: `${bWidth}px`,
        height: `${bHeight}px`,
      }"
      v-bind:class="{ active: bActive }"
      @mousedown.prevent="selectBox"
    >
      <!-- Display the box lines with proper cursors -->
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
      // Selects the clicked on box
      this.onSelect(this.bIndex);
    },
    removeMyself() {
      // Removes the box from the array
      this.onDelete(this.bIndex);
    },
  },
};
</script>

<style lang="scss" scoped>
.box {
  position: absolute !important;
  cursor: pointer;

  &:hover {
    background-color: rgba(88, 96, 213, 0.2);
  }
  &.active {
    background-color: rgba(207, 66, 242, 0.2);
  }

  z-index: 3;
}
.box .box-line {
  position: absolute;
  transition: all 0.25s;
}
.box .box-top-line {
  top: 0;
  left: 0;
  right: 0;
  height: 15px;
  border-top: 2px rgba(43, 55, 210, 0.8) solid;
  transition: all 0.3s;
  cursor: n-resize;
}
.box .box-bottom-line {
  bottom: 0;
  left: 0;
  right: 0;
  height: 15px;
  border-bottom: 2px rgba(43, 55, 210, 0.8) solid;
  transition: all 0.3s;
  cursor: s-resize;
}
.box .box-left-line {
  top: 0;
  left: 0;
  bottom: 0;
  width: 15px;
  border-left: 2px rgba(43, 55, 210, 0.8) solid;
  transition: all 0.3s;
  cursor: w-resize;
}
.box .box-right-line {
  top: 0;
  right: 0;
  bottom: 0;
  width: 15px;
  border-right: 2px rgba(43, 55, 210, 0.8) solid;
  transition: all 0.3s;
  cursor: e-resize;
}
.box .box-corner {
  position: absolute;
  width: 10px;
  height: 10px;
  opacity: 0;
  transition: all 0.3s;
}
.box .box-top-left-corner {
  top: 1px;
  left: 1px;
  cursor: nw-resize;
}
.box .box-top-right-corner {
  top: 1px;
  right: 1px;
  cursor: ne-resize;
}
.box .box-bottom-left-corner {
  bottom: 1px;
  left: 1px;
  cursor: sw-resize;
}
.box .box-bottom-right-corner {
  bottom: 1px;
  right: 1px;
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
  font-size: 16px;
  font-weight: bold;
  height: 15px;
  width: 15px;
}
.box-delete:hover {
  color: rgb(213, 12, 12);
  transition: all 0.3;
}
</style>
