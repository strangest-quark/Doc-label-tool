<template>
  <div id="app">
    <div id="label-bar">
      <div id="label-bar" style="height:250px; width: 35%; padding-right:100px">
        <h4>Edit labels</h4>
        <ul>
          <li v-for="(box, i) in boxes" :key="i" v-bind:class="{'active': i===activeBoxIndex}">
            <a style="float:right" @click="removeBox(i)">x</a>
            <br />
            <input v-model="box.label" placeholder="Cell text" v-on:click="makeBoxActive(i)" />
            <input v-model="box.row" placeholder="Cell row" v-on:click="makeBoxActive(i)" />
            <input v-model="box.col" placeholder="Cell column" v-on:click="makeBoxActive(i)" />
          </li>
        </ul>
      </div>
      <h4>Output</h4>
      <div>
      <pre
        style="padding-right:100px; height: 250px; overflow-y: scroll; scroll-snap-type: y proximity; "
      >{{ JSON.stringify(boxes, null, '\t') }}</pre>
      <download-csv style="color:blue; padding-top:20px"
    :data   = "boxes">
    <b><u>Export data</u></b>
</download-csv>
</div>
      <embed
        style="padding-top:40px"
        src="RedactedPayPal_sample3.pdf"
        width="80%"
        height="400px"
        type="application/pdf"
      />
    </div>
    <div
      id="image-wrapper"
      :style="{backgroundImage: `url(0.png)`}"
      @mousedown="startDrawingBox"
      @mousemove="changeBox"
      @mouseup="stopDrawingBox"
    >
      <Box
        v-if="drawingBox.active"
        :b-width="drawingBox.width"
        :b-height="drawingBox.height"
        :b-top="drawingBox.top"
        :b-left="drawingBox.left"
      />
      <Box
        v-for="(box, i) in boxes"
        :key="i"
        :b-top="box.top"
        :b-left="box.left"
        :b-label="box.label"
        :b-width="box.width"
        :b-height="box.height"
        :b-active="i===activeBoxIndex"
        :on-select="makeBoxActive"
        :b-index="i"
        :on-delete="removeBox"
      />
    </div>
  </div>
</template>

<script>
import Box from "./components/Box";
import { pick } from "lodash";


const getCoursorLeft = (e) => {
  return e.pageX - 10;
};

const getCoursorTop = (e) => {
  return e.pageY - 10;
};

export default {
  name: "app",
  components: { Box },
  data: function () {
    return {
      drawingBox: {
        active: false,
        top: 0,
        left: 0,
        height: 0,
        width: 0,
      },
      boxes: [],
      activeBoxIndex: null,
    };
  },
  methods: {
    startDrawingBox(e) {
      this.drawingBox = {
        width: 0,
        height: 0,
        top: getCoursorTop(e),
        left: getCoursorLeft(e),
        active: true,
      };
    },
    changeBox(e) {
      if (this.drawingBox.active) {
        this.drawingBox = {
          ...this.drawingBox,
          width: getCoursorLeft(e) - this.drawingBox.left,
          height: getCoursorTop(e) - this.drawingBox.top,
        };
      }
    },
    stopDrawingBox() {
      if (this.drawingBox.active) {
        if (this.drawingBox.width > 5) {
          this.boxes.push({
            ...pick(this.drawingBox, ["width", "height", "top", "left"]),
          });
        }
        this.drawingBox = {
          active: false,
          top: 0,
          left: 0,
          height: 0,
          width: 0,
        };
      }
    },
    makeBoxActive(i) {
      this.activeBoxIndex = i;
    },
    removeBox(i) {
      this.boxes = this.boxes.filter((elem, index) => {
        return index !== i;
      });
      this.activeBoxIndex = null;
    },
  },
};
</script>

<style lang="scss" scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  #image-wrapper {
    height: 1400px;
    width: 850px;
    background-repeat: no-repeat;
    position: relative;
  }

  #label-bar {
    float: right;
    width: 50%;

    ul {
      padding: 0;
      overflow-y: scroll;
      scroll-snap-type: y proximity;
      height: 250px;
      li {
        list-style-type: none;
        padding: 8px 16px;

        &.active {
          background-color: lightblue;
        }

        background-color: rgb(233, 233, 233);

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