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
      <el-row :gutter="20" justify="center" align="middle">
        <el-popover placement="bottom" trigger="manual" :visible="infoVisible" width="550" popper-class="table-popover" :teleported="false">
          <template #default>
            <NeedlesTable :needlesInfo="needlesInfo" />
          </template>
          <template #reference>
            <el-button
              size="small"
              class="needles-button"
              @click="infoVisible = !infoVisible"
              :icon="ElIconDataAnalysis"
            >
              {{ infoVisible ? "Hide Needles Info" : "Display Needles Info"}}
            </el-button>
          </template>
        </el-popover>
      </el-row>
    </div>
    <ScaffoldVuer
      v-if="url"
      ref="scaffold"
      class="vuer"
      :display-u-i="displayUI"
      :url="url"
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
import NeedlesTable from "./NeedlesTable.vue";
import { ScaffoldVuer } from "@abi-software/scaffoldvuer";
import "@abi-software/scaffoldvuer/dist/style.css";
import {
  EditPen as ElIconEditPen,
  FolderOpened as ElIconFolderOpened,
  DataAnalysis as ElIconDataAnalysis,
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
import {
  THREE
} from "zincjs";

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

const getIntersectedObjects = (intersects) => {
  const primitiveInfos = [];
  intersects.forEach((intersect) => {
    const zincObject = intersect.object.userData;
    if (zincObject) {
      const groupName = zincObject?.groupName;
      const distance = intersect.distance.toFixed(2);
      const x = intersect.point.x.toFixed(2);
      const y = intersect.point.y.toFixed(2);
      const z = intersect.point.z.toFixed(2);
      primitiveInfos.push({groupName, distance, x, y, z});
    }
  });
  return primitiveInfos;
}

const findNearbyPoints = (data, tolerance) => {
  if (data[0].data.zincObject?.isPointset) {
    return data[0].data.zincObject;
  } else {
    let distance = data[0].extraData.intersected.distance + tolerance;
    const intersects = data[0].extraData.intersects;
    for (let i = 0; i < intersects.length; i++) {
      if (distance > intersects[i].distance) {
       if (intersects[i].object.userData?.isPointset) {
        return intersects[i].object.userData;
       }
      } else {
        return;
      }
    }
  }
}

const v1 = new THREE.Vector3();
const v2 = new THREE.Vector3();

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
    NeedlesTable,
    ScaffoldVuer,
  },
  data: function () {
    return {
      quickEditOn: false,
      displayUI: true,
      ElIconEditPen: shallowRef(ElIconEditPen),
      ElIconFolderOpened: shallowRef(ElIconFolderOpened),
      ElIconDataAnalysis: shallowRef(ElIconDataAnalysis),
      coordinatesClicked: [],
      needlesInfo: {},
      infoVisible: false,
      importing: false,
    };
  },
  props: {
    consoleOn: {
      type: Boolean,
      default: false,
    },    
    url: {
      type: String,
      default: "https://mapcore-bucket1.s3.us-west-2.amazonaws.com/texture/arm1/arm_metadata.json",
    },
    pointTolerance: {
      type: Number,
      default: 20,
    }
  },
  watch: {
    helpMode: function (newVal) {
      if (!newVal) {
        this.helpModeActiveItem = 0;
      }
    },
    quickEditOn: function(value) {
      if (value) {
        this.$refs.scaffold.$module.ignorePreviousSelected = true;
        this.$refs.scaffold.viewingMode = "Exploration";
      } else {
        this.$refs.scaffold.$module.ignorePreviousSelected = false;
      }
    },
  },
  mounted: function () {
    this._createLinesLength = 100;
    const Zinc = this.$refs.scaffold.$module.Zinc;
    this._pickableObjects = [];
    const scene  = this.$refs.scaffold.$module.scene;
    this._rayCaster = new Zinc.RayCaster(
      scene,
      scene,
      undefined,
      undefined,
    );
  },
  methods: {
    exportLocalAnnotations: function() {
      const annotations = this.$refs.scaffold.getLocalAnnotations();
      const filename = 'scaffoldAnnotations' + JSON.stringify(new Date()) + '.json';
      writeTextFile(filename, annotations);
    },
    onReaderLoad: function(event) {
      const annotationsList = JSON.parse(event.target.result);
      this.importing = true;
      this.$refs.scaffold.importLocalAnnotations(annotationsList);
      this.importing = false;
    },
    importLocalAnnotations: function() {
      const selectedFile = document.getElementById("annotations-upload").files[0];
      const reader = new FileReader();
      reader.onload = this.onReaderLoad;
      reader.readAsText(selectedFile);
    },
    objectAdded: function (zincObject) {
      if (!zincObject.isLines2) {
        this._pickableObjects.push(zincObject);
      } else {
        this.userPrimitivesUpdated({zincObject});
      }
    },
    screenCapture: function () {
      this.$refs.scaffold.captureScreenshot("capture.png");
    },
    onReady: function () {
      const viewer = this.$refs.scaffold;
      const bounds = viewer.$module.scene.getBoundingBox();
      const d = bounds.max.distanceTo( bounds.min );
      this._createLinesLength = d / 6.0;
      if (this.consoleOn) console.log("Lines length", this._createLinesLength);
    },
    addLinesWithNormal: function (data, coord, normal) {
      const myViewer = this.$refs.scaffold;
      if (this.consoleOn) console.log(myViewer.createData);
      //changing shape like this will create a reactive issue.
      if (coord && normal) {
        myViewer.createData.shape = "LineString";
        this.$nextTick(() => {
          const newCoords = [
            coord[0] + normal.x * this._createLinesLength,
            coord[1] + normal.y * this._createLinesLength,
            coord[2] + normal.z * this._createLinesLength,
          ];
          myViewer.createData.toBeConfirmed = false;
          myViewer.createData.points.length = 0;
          myViewer.createData.points.push(newCoords);
          myViewer.createEditTemporaryLines(coord);
          myViewer.drawLine(coord, data);
        });
      }
    },
    addPoint: function (data, coord) {
      const myViewer = this.$refs.scaffold;
      if (this.consoleOn) {
        console.log(myViewer.createData);
        console.log("addPoints", data, coord);
      }
      if (coord) {
        myViewer.createData.shape = "Point";
        this.$nextTick(() => {
          myViewer.createData.toBeConfirmed = false;
          myViewer.createData.points.length = 0;
          myViewer.drawPoint(coord, data);
        });
      }
    },
    onSelected: function (data) {
      if (data && data.length > 0 && data[0].data.group) {
        if (this.consoleOn) {
          console.log(data[0].extraData.intersects);
          console.log(data[0], data[0].extraData.intersected);
        }
        if (this.quickEditOn && data[0].extraData.worldCoords) {
          //Try to look for point within tolerance
          const points = findNearbyPoints(data, this.pointTolerance);
             //Look for the surface underneath a point
          if (points && points?.isPointset) {
            const intersects = data[0].extraData.intersects;
            if (intersects) {
              let found = false;
              for (let i = 0; i < intersects.length && !found; i++) {
                if (intersects[i].face) {
                  found = true;
                  const coord = [intersects[i].point.x, intersects[i].point.y ,intersects[i].point.z];
                  this.addLinesWithNormal(data, coord, intersects[i].face.normal);
                }
              }
            }
          } else if (data[0].extraData.intersected?.face) {
            this.addPoint(data, data[0].extraData.worldCoords);
          }
        }
      }
    },
    userPrimitivesUpdated: function (payload) {
      if (this.consoleOn) console.log("userPrimitivesUpdated", payload);
      const zincObject = payload.zincObject;
      if ((zincObject.isEditable ||  this.importing) && zincObject.isLines2) {
        //Call the following to set the camera       
        const scene = this.$refs.scaffold.$module.scene;
        const camera = scene.getZincCameraControls();
        if (this._rayCaster) {
          this._rayCaster.getIntersectsObjectWithCamera(camera, 0, 0);
          for (let i = 0; i * 2 < zincObject.drawRange; i++) {
            const v = zincObject.getVerticesByFaceIndex(i);
            let d = [v[1][0] - v[0][0], v[1][1] - v[0][1], v[1][2] - v[0][2]];
            const mag = Math.sqrt(d[0] * d[0] + d[1] * d[1] + d[2] * d[2]);
            for (let l = 0; l < 3; l++) {
              v1.setComponent(l, v[0][l]);
              d[l] = d[l] / mag;
              v2.setComponent(l, d[l]);
            }
            this._rayCaster.setPickableObjects(this._pickableObjects);
            const objects = this._rayCaster.getIntersectsObjectWithOrigin(
              camera, v1, v2);
            const intersects = objects.filter((object) => object.distance < mag);
            const primitivesInfo = getIntersectedObjects(intersects);
            let needlesName = `Needle ${i + 1}`;
            if (zincObject.groupName) {
              needlesName = needlesName + ` of ${zincObject.groupName}`;
            }
            this.needlesInfo[needlesName] = primitivesInfo;
          }
        }
      }
    },
  },
};
</script>

<style scoped lang="scss">

:deep(.warning-icon) {
  display:none;
}
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
  background-color: rgba(255, 255, 255, 0.5);

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

.needles-button {
  z-index:10000;
  margin-top: 5px;
}

.vuer {
  :deep(svg.map-icon) {
    color: #8300BF;
  }
}
/* Component Styles */
</style>
