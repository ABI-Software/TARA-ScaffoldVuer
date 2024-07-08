<template>
  <div class="scaffold-container">
    <div class="settings-panels">
      <el-row :gutter="20" justify="center" align="middle">
        <el-col :span="auto">
          <el-button
            size="small"
            :icon="ElIconFolderOpened"
            @click="exportLocalAnnotations()">
            Export Annotations
          </el-button>
        </el-col>
        <el-col :span="auto">
            <el-button size="small" :icon="ElIconFolderOpened">
              <label for="annotations-upload">Import Annotations</label>
              <input
                id="annotations-upload"
                type="file"
                accept="application/json"
                @change="importLocalAnnotations" 
              />
            </el-button>
        </el-col>
      </el-row>
      <el-row :gutter="20" justify="center" align="middle">
        <el-col :span="12">
          Quick Edit
        </el-col>
        <el-col :span="6">
          <el-switch
            v-model="quickEditOn"
            :active-action-icon="ElIconEditPen"
            :inactive-action-icon="ElIconEditPen"
          />
        </el-col>
      </el-row>
    </div>
    <ScaffoldVuer
      v-if="url"
      ref="scaffold"
      class="vuer"
      :display-u-i="displayUI"
      :url="url"
      :help-mode="helpMode"
      :helpModeDialog="useHelpModeDialog"
      :helpModeActiveItem="helpModeActiveItem"
      @help-mode-last-item="onHelpModeLastItem"
      :display-latest-changes="false"
      :display-minimap="false"
      :display-markers="false"
      :enableOpenMapUI="false"
      :enableLocalAnnotations="true"
      :marker-cluster="false"
      :show-colour-picker="showColourPicker"
      :render="true"
      @on-ready="onReady"
      @scaffold-selected="onSelected"
      @user-primitives-updated="userPrimitivesUpdated"
      @zinc-object-added="objectAdded"
      @vue:mounted="viewerMounted"
    />
  </div>
</template>

<script>
/* eslint-disable no-alert, no-console */
import { shallowRef } from 'vue';
import { ScaffoldVuer } from "@abi-software/scaffoldvuer";
import "@abi-software/scaffoldvuer/dist/style.css";
import {
  EditPen as ElIconEditPen,
  FolderOpened as ElIconFolderOpened,
} from '@element-plus/icons-vue';
import {
  ElButton as Button,
  ElCol as Col,
  ElIcon as Icon,
  ElInput as Input,
  ElInputNumber as InputNumber,
  ElPopover as Popover,
  ElRow as Row,
  ElSwitch as Switch,
} from "element-plus";

const writeTextFile = (filename, data) => {
  let dataStr =
    "data:text/json;charset=utf-8," +
    encodeURIComponent(JSON.stringify(data));
  let hrefElement = document.createElement("a");
  document.body.append(hrefElement);
  hrefElement.download = filename;
  hrefElement.href = dataStr;
  hrefElement.click();
  hrefElement.remove();
} 

export default {
  name: "TaraScaffoldVuer",
  components: {
    Button,
    Col,
    Icon,
    Input,
    InputNumber,
    Popover,
    Row,
    Switch,
    ElIconEditPen,
    ElIconFolderOpened,
    ScaffoldVuer,
  },
  data: function () {
    return {
      quickEditOn: false,
      consoleOn: true,
      displayUI: true,
      helpMode: false,
      helpModeActiveItem: 0,
      helpModeLastItem: false,
      useHelpModeDialog: false,
      ElIconEditPen: shallowRef(ElIconEditPen),
      ElIconFolderOpened: shallowRef(ElIconFolderOpened),
      coordinatesClicked: [],
    };
  },
  props: {
    url: {
      type: String,
      default: "https://mapcore-bucket1.s3.us-west-2.amazonaws.com/texture/arm1/arm_metadata.json",
    },
  },
  watch: {
    helpMode: function (newVal) {
      if (!newVal) {
        this.helpModeActiveItem = 0;
      }
    },
    quickEditOn: function(value) {
      if (value) {
        this.$refs.scaffold.viewingMode = "Exploration";
      }
    },
  },
  mounted: function () {
    this._objects = [];
    this._createLinesLength = 100;
    const Zinc = this.$refs.scaffold.$module.Zinc;
    this._rayCaster = new Zinc.RayCaster(this.$refs.scaffold.$module.scene,
      this.$refs.scaffold.$module.scene, undefined, undefined);
  },
  methods: {
    exportLocalAnnotations: function() {
      const annotations = this.$refs.scaffold.getLocalAnnotations();
      const filename = 'scaffoldAnnotations' + JSON.stringify(new Date()) + '.json';
      writeTextFile(filename, annotations);
    },
    onReaderLoad: function(event) {
      const annotationsList = JSON.parse(event.target.result);
      this.$refs.scaffold.importLocalAnnotations(annotationsList);
    },
    importLocalAnnotations: function() {
      const selectedFile = document.getElementById("annotations-upload").files[0];
      const reader = new FileReader();
      reader.onload = this.onReaderLoad;
      reader.readAsText(selectedFile);
    },
    objectAdded: function (zincObject) {
      if (this.consoleOn) {
        console.log(zincObject)
        this._objects.push(zincObject);
      }
    },
    screenCapture: function () {
      this.$refs.scaffold.captureScreenshot("capture.png");
    },
    onReady: function () {
      const bounds = this.$refs.scaffold.$module.scene.getBoundingBox();
      const d = bounds.max.distanceTo( bounds.min );
      this._createLinesLength = d / 6.0;
      if (this.consoleOn) console.log("Lines length", this._createLinesLength);
      this.$refs.scaffold.changeActiveByName(
        undefined, undefined, false);
      this._rayCaster.setPickableObjects(this._objects);
    },
    addLinesWithNormal: function (data, coord, normal) {
      const myViewer = this.$refs.scaffold;
      if (this.consoleOn) console.log(myViewer.createData);
      if (coord && normal) {
        myViewer.tData.visible = true;
        const newCoords = [
          coord[0] + normal.x * this._createLinesLength,
          coord[1] + normal.y * this._createLinesLength,
          coord[2] + normal.z * this._createLinesLength,
        ];
        myViewer.createData.points.length = 0;
        myViewer.createData.points.push(newCoords);
        myViewer.createEditTemporaryLines(coord);
        myViewer.createData.shape = "LineString";
        myViewer.drawLine(
          coord,
          data,
        );
      }
    },
    onSelected: function (data) {
      if (data && data.length > 0 && data[0].data.group) {
        if (this.consoleOn) console.log(data[0], data[0].extraData.intersected);
        if (this.quickEditOn && data[0].extraData.worldCoords &&
          data[0].extraData.intersected?.face) {
          this.addLinesWithNormal(
            data,
            data[0].extraData.worldCoords,
            data[0].extraData.intersected.face.normal)
        }
      }
    },
    userPrimitivesUpdated: function (payload) {
      if (this.consoleOn) console.log("userPrimitivesUpdated", payload);
      const zincObject = payload.zincObject;
      if (zincObject.isEditable && zincObject.isLines2) {
        if (this._rayCaster) {
          const scene = this.$refs.scaffold.$module.scene;
          const camera = scene.getZincCameraControls();
          for (let i = 0; i * 2 < zincObject.drawRange; i++) {
            const v = zincObject.getVerticesByFaceIndex(i);
            console.log(v)
            const origin = v[0];
            let d = [v[1][0] - v[0][0], v[1][1] - v[0][1], v[1][2] - v[0][2]];
            const mag = Math.sqrt(d[0] * d[0] + d[1] * d[1] + d[2] * d[2]);
            for (let l = 0; l < 3; l++) {
              d[i] = d[i] / mag;
            }
            console.log(this._rayCaster.getIntersectsObjectWithOrigin(camera, origin, d));
          }
          
        }
      }
    },
  },
};
</script>

<style scoped lang="scss">
.scaffold-container {
  height: 100%;
  width: 100%;
  overflow: hidden;
  position: absolute;
}

input[type="file"] {
  display: none;
}

.settings-panels {
  z-index:10000;
  right:0px;
  position:absolute;
  text-align: center;

  .el-row {
    width:200px; 
    .el-col {
      &.is-guttered {
        padding-top: 5px;
        padding-bottom: 5px;
      }

      > p {
        font-size: 12px;
        margin: 0;
      }

      .el-input__inner,
      .el-switch {
        font-size: 12px;
        height: 20px;
      }
    }
  }
}

.vuer {
  :deep(svg.map-icon) {
    color: #8300BF;
  }
}
/* Component Styles */
</style>
