<template>
  <div id="annotator">
    <div class="q-pa-md">
      <!-- Loop through the pest images to display them for annotating -->
      <div v-for="(image, i) in pest_images" :key="'pest_image_' + i">
        <q-card flat bordered class="col">
          <!-- Responsivly load image to fit in screen size -->
          <q-responsive :ratio="16 / 9">
            <div
              id="image-wrapper"
              :style="{ backgroundImage: `url(${image.src})` }"
              @mousedown="startDrawingBox"
              @mousemove="changeBox"
              @mouseup="stopDrawingBox"
            >
              <!-- Active box label -->
              <BoxLabel
                v-if="drawingBox.active"
                :b-label="drawingBox.label"
                :b-width="drawingBox.width"
                :b-height="drawingBox.height"
                :b-top="drawingBox.top"
                :b-left="drawingBox.left"
              />
              <!-- Loop through all inactive box labels to display -->
              <BoxLabel
                v-for="(box, j) in boxes"
                :key="'pest_image_' + i + '_box_annotation_' + j"
                :b-top="box.top"
                :b-left="box.left"
                :b-label="box.label"
                :b-width="box.width"
                :b-height="box.height"
                :b-active="j === activeBoxIndex"
                :on-select="makeBoxActive"
                :b-index="j"
                :on-delete="removeBox"
              />
            </div>
          </q-responsive>
          <q-separator />

          <h5>{{ image.title }}</h5>
        </q-card>
      </div>
      <q-card flat bordered class="col" style="margin: 50px">
        <h5>All Labeled Pests:</h5>

        <q-separator />

        <div
          style="
            display: flex;
            align-items: center;
            flex-direction: row;
            flex-wrap: wrap;
            width: 500px;
            justify-content: space-between;
          "
        >
          <!-- Display all pest annotations for easy deletion-->
          <q-item
            v-for="(box, i) in boxes"
            :key="i"
            v-bind:class="{ active: i === activeBoxIndex }"
            style="width: 400px"
          >
            <!-- Input for the label -->
            <q-input
              v-model="box.label"
              v-on:click="makeBoxActive(i)"
              outlined
            />
            <!-- Deletes a single box label -->
            <q-btn
              round
              style="width: 30px; height: 30px; margin: 10px"
              color="red"
              @click="removeBox(i)"
              >X</q-btn
            >
          </q-item>
        </div>
        <!-- Deletes all box labels -->
        <q-btn
          color="red"
          style="margin: 20px"
          label="Delete All Labels"
          @click="removeAllBoxes()"
        />
      </q-card>

      <!-- A button to scroll to the bottom -->
      <q-page-scroller
        reverse
        position="bottom"
        :scroll-offset="20"
        :offset="[0, 18]"
      >
        <q-btn fab icon="keyboard_arrow_down" color="primary" />
      </q-page-scroller>
      <!-- A button to scroll to the top -->
      <q-page-scroller position="top" :scroll-offset="20" :offset="[0, 18]">
        <q-btn fab icon="keyboard_arrow_up" color="primary" />
      </q-page-scroller>
    </div>
  </div>
</template>

<script>
// Components
import BoxLabel from "./BoxLabel";

// Packages
import { pick } from "lodash";
import iconSet from "quasar/icon-set/ionicons-v4";
import "@quasar/extras/ionicons-v4/ionicons-v4.css";

// Gets the left and top position of the cursor on click
const getCoursorLeft = (e) => {
  return e.pageX - 20;
};

const getCoursorTop = (e) => {
  return e.pageY - 20;
};

export default {
  name: "PestAnnotator",
  config: {
    iconSet,
  },
  components: { BoxLabel },
  data: function () {
    return {
      drawingBox: {
        active: false,
        top: 0,
        left: 0,
        height: 0,
        width: 0,
        label: "",
      },
      boxes: [],
      pest_images: [
        {
          src: "images/tomato.jpg",
          title: "Tomato Plant - 05/05/2023",
        },
      ],
      activeBoxIndex: null,
    };
  },
  methods: {
    startDrawingBox(e) {
      // Get the cursor coordinates
      const TOP = getCoursorTop(e);
      const LEFT = getCoursorLeft(e);

      // Find any existing boxes
      const EXISTING_BOX = _.findIndex(this.boxes, function (box) {
        return (
          box.top <= TOP &&
          box.left <= LEFT &&
          box.top + box.height >= TOP &&
          box.left + box.width >= LEFT
        );
      });

      if (EXISTING_BOX < 0 && !this.drawingBox.active) {
        // Start drawing a new box because no boxes overlap
        this.drawingBox = {
          resize: false,
          width: 20,
          height: 20,
          top: TOP,
          left: LEFT,
          active: true,
          label: "",
        };
      } else {
        // Expand existing box
        this.drawingBox = {
          resize: true,
          width: this.boxes[EXISTING_BOX].width,
          height: this.boxes[EXISTING_BOX].height,
          top: this.boxes[EXISTING_BOX].top,
          left: this.boxes[EXISTING_BOX].left,
          label: this.boxes[EXISTING_BOX].label,
          active: true,
        };
      }
    },
    changeBox(e) {
      // Get cursor positions
      const LEFT = getCoursorLeft(e);
      const TOP = getCoursorTop(e);
      const OLD_LEFT = this.drawingBox.left;
      const OLD_TOP = this.drawingBox.top;
      const OLD_WIDTH = this.drawingBox.width;
      const OLD_HEIGHT = this.drawingBox.height;

      if (this.drawingBox.active) {
        if (this.drawingBox.resize) {
          // New values for the box
          let newHeight = OLD_HEIGHT;
          let newWidth = OLD_WIDTH;
          let newTop = OLD_TOP;
          let newLeft = OLD_LEFT;

          // We are resizing an existing box, check for direction and update accordingly
          if (LEFT < OLD_LEFT && LEFT > 0 && OLD_LEFT - LEFT + OLD_WIDTH > 0) {
            // Expand left side
            newLeft = LEFT;
            newWidth = OLD_LEFT - LEFT + OLD_WIDTH;
          } else if (
            LEFT > OLD_LEFT &&
            LEFT < OLD_LEFT + OLD_WIDTH &&
            OLD_WIDTH - (LEFT - OLD_LEFT) > 0 &&
            LEFT < OLD_LEFT + 10
          ) {
            // Minimize left side
            newLeft = LEFT;
            newWidth = OLD_WIDTH - (LEFT - OLD_LEFT);
          } else if (LEFT > OLD_LEFT && LEFT > OLD_LEFT + OLD_WIDTH) {
            // Expand right side
            newWidth = LEFT - OLD_LEFT;
          } else if (LEFT > OLD_LEFT && LEFT < OLD_LEFT + OLD_WIDTH) {
            // Minimize right size
            newWidth = LEFT - OLD_LEFT;
          }

          if (TOP < OLD_TOP && OLD_HEIGHT + (OLD_TOP - TOP) > 0) {
            // Expand top side
            newTop = TOP;
            newHeight = OLD_HEIGHT + (OLD_TOP - TOP);
          } else if (TOP < OLD_TOP + 10 && OLD_HEIGHT - (TOP - OLD_TOP) > 0) {
            // Minimize top size
            newTop = TOP;
            newHeight = OLD_HEIGHT - (TOP - OLD_TOP);
          } else if (TOP > OLD_TOP + OLD_HEIGHT) {
            // Expand bottom side
            newHeight = TOP - OLD_TOP;
          } else if (TOP > OLD_TOP && TOP < OLD_TOP + OLD_HEIGHT) {
            // Minimize bottom side
            newHeight = TOP - OLD_TOP;
          }

          // Resizing drawing box
          this.drawingBox = {
            ...this.drawingBox,
            left: newLeft,
            top: newTop,
            width: newWidth,
            height: newHeight,
          };

          // Update existing box size based on new values
          let newBoxes = [];
          const UPDATED_BOXES = _.map(this.boxes, (box) => {
            if (
              box.width === OLD_WIDTH &&
              box.height === OLD_HEIGHT &&
              box.top === OLD_TOP &&
              box.left === OLD_LEFT
            ) {
              box = {
                left: newLeft,
                top: newTop,
                width: newWidth,
                height: newHeight,
                label: box.label,
              };
            }
            newBoxes.push({
              ...pick(box, ["width", "height", "top", "left", "label"]),
            });
            return box;
          });
          this.boxes = newBoxes;
        } else {
          // This is a new box, draw accordingly
          this.drawingBox = {
            ...this.drawingBox,
            width: LEFT - OLD_LEFT,
            height: TOP - OLD_TOP,
          };
        }
      }
    },
    stopDrawingBox() {
      // Make sure box is active
      if (this.drawingBox.active) {
        // Make sure the box width is over 20px
        if (this.drawingBox.width > 20) {
          if (!this.drawingBox.resize) {
            // Only add if the box doesn't exist
            this.boxes.push({
              ...pick(this.drawingBox, [
                "width",
                "height",
                "top",
                "left",
                "label",
              ]),
            });
          }
        }

        // Reset drawing box
        this.drawingBox = {
          active: false,
          resize: false,
          top: 0,
          left: 0,
          height: 0,
          width: 0,
          label: "",
        };
      }
    },
    makeBoxActive(i) {
      // When you click on a box, make it active
      this.activeBoxIndex = i;
    },
    removeAllBoxes() {
      // Deletes all boxes from the array
      this.boxes = [];
      this.activeBoxIndex = null;
    },
    removeBox(i) {
      // Removes a box at a specific index
      this.boxes = this.boxes.filter((elem, index) => {
        return index !== i;
      });
      this.activeBoxIndex = null;
    },
  },
};
</script>

<style lang="scss" scoped>
#annotator {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  position: relative;

  #image-wrapper {
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center center;
    cursor: cell;
    width: 100%;
  }

  #label-bar {
    float: right;
    margin-right: 50px;
    width: 220px;

    ul {
      padding: 0;

      li {
        list-style-type: none;
        padding: 8px 16px;

        &.active {
          background-color: lightblue;
        }

        a {
          cursor: pointer;
          display: inline-block;
          margin-left: 4px;
          font-weight: bold;
          color: red;
        }
      }
    }
  }
}
</style>
