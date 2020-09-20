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
        >{{ JSON.stringify(tables, null, '\t') }}</pre>

        <input v-model="imageInput" placeholder="Enter image name" />
        <button type="submit" v-on:click="load()">Load</button>
        <br />
        <br />
        <b v-on:click="saveTable()" style="color:blue; padding:5px">
          <u>Save Table</u>
        </b>
        <b v-on:click="saveData()" style="color:blue; padding:5px">
          <u>Export data</u>
        </b>
        <b v-on:click="upload()" style="color:blue; padding:5px">
          <u>Save data to S3</u>
        </b>
      </div>
      <embed
        style="padding-top:25px"
        :src="pdfName"
        width="80%"
        height="400px"
        type="application/pdf"
      />
    </div>
    <div
      id="image-wrapper"
      :style="{backgroundImage: 'url('+ imageName +')'}"
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

<script src="https://sdk.amazonaws.com/js/aws-sdk-2.756.0.min.js"></script>

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
      tables: [],
      imageInput: "paypal_0",
      imageName:
        "https://doc-upload-test-987.s3.ap-south-1.amazonaws.com/paypal_0.png",
      pdfName:
        "https://doc-upload-test-987.s3.ap-south-1.amazonaws.com/paypal.pdf",
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
    saveTable() {
      this.tables.push(this.boxes);
      this.boxes = [];
    },
    load() {
      this.pdfName =
        "https://doc-upload-test-987.s3.ap-south-1.amazonaws.com/" +
        this.imageInput.substring(0, this.imageInput.length - 2) +
        ".pdf";
      this.imageName =
        "https://doc-upload-test-987.s3.ap-south-1.amazonaws.com/" +
        this.imageInput +
        ".png";
    },
    saveData() {
      const data = JSON.stringify(this.tables);
      const blob = new Blob([data], { type: "text/plain" });
      const e = document.createEvent("MouseEvents"),
        a = document.createElement("a");
      a.download = "test.json";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ["text/json", a.download, a.href].join(":");
      e.initEvent(
        "click",
        true,
        false,
        window,
        0,
        0,
        0,
        0,
        0,
        false,
        false,
        false,
        false,
        0,
        null
      );
      a.dispatchEvent(e);
    },
    upload() {
      var albumBucketName = "doc-upload-test-987";
      var bucketRegion = "ap-south-1";
      var IdentityPoolId = "ap-south-1:5f660682-a77c-4d77-9456-7e4535bcb50c";

      AWS.config.update({
        region: bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: IdentityPoolId,
        }),
      });

      var s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: { Bucket: albumBucketName },
      });
      var file = new Blob([JSON.stringify(this.tables)], { type: "application/json" });
      var fileName = this.imageInput + ".json";
      // Use S3 ManagedUpload class as it supports multipart uploads
      var upload = new AWS.S3.ManagedUpload({
        params: {
          Bucket: "doc-upload-test-987",
          Key: fileName,
          Body: file,
          ACL: "public-read",
        },
      });

      var promise = upload.promise();

      promise.then(
        function (data) {
          alert("Successfully uploaded data.");
        },
        function (err) {
          console.log(err.message);
          return alert("There was an error uploading your data: ");
        }
      );
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