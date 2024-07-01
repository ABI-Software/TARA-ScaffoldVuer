<template>
  <div class="scaffold-container">
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
      @shown-tooltip="onTooltipShown"
      @shown-map-tooltip="onMapTooltipShown"
      :display-latest-changes="false"
      :display-minimap="false"
      :display-markers="false"
      :enableOpenMapUI="false"
      :marker-cluster="false"
      :show-colour-picker="showColourPicker"
      :render="true"
      @on-ready="onReady"
      @scaffold-selected="onSelected"
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
  FolderOpened as ElIconFolderOpened,
  Setting as ElIconSetting,
} from '@element-plus/icons-vue';
import {
  ElAutocomplete as Autocomplete,
  ElButton as Button,
  ElCol as Col,
  ElIcon as Icon,
  ElInput as Input,
  ElInputNumber as InputNumber,
  ElPopover as Popover,
  ElRow as Row,
  ElSwitch as Switch,
} from "element-plus";
import { HelpModeDialog } from '@abi-software/map-utilities'
import '@abi-software/map-utilities/dist/style.css'

export default {
  name: "TaraScaffoldVuer",
  components: {
    Autocomplete,
    Button,
    Col,
    Icon,
    Input,
    InputNumber,
    Popover,
    Row,
    Switch,
    ElIconFolderOpened,
    ElIconSetting,
    ScaffoldVuer,
    HelpModeDialog,
  },
  data: function () {
    return {
      consoleOn: true,
      displayUI: true,
      helpMode: false,
      helpMode: false,
      helpModeActiveItem: 0,
      helpModeLastItem: false,
      useHelpModeDialog: true,
      ElIconSetting: shallowRef(ElIconSetting),
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
  },
  mounted: function () {
    this._objects = [];
  },
  unmounted: function () {
    this.$refs.dropzone.revokeURLs();
  },
  methods: {
    objectAdded: function (zincObject) {
      if (this.consoleOn) {
        console.log(zincObject)
        console.log(this.$refs.scaffold.$module.scene.getBoundingBox())
      }
      if (this._objects.length === 0) {
        zincObject.setMarkerMode("on");
      }
      this._objects.push(zincObject);
    },
    screenCapture: function () {
      this.$refs.scaffold.captureScreenshot("capture.png");
    },
    onReady: function () {
      if (this.consoleOn) console.log(this.$refs.scaffold)
    },
    addLines: function (coord) {
      if (this.coordinatesClicked.length === 1) {
        const returned = this.$refs.scaffold.$module.scene.createLines(
            "test",
            "lines",
            [this.coordinatesClicked[0], coord],
            0x00ee22,
          );
          this.coordinatesClicked.length = 0;
          if (this.consoleOn) console.log(returned);
      } else {
        this.coordinatesClicked.push(coord);
      }
    },
    onSelected: function (data) {
      if (data && data.length > 0 && data[0].data.group) {
        if (this.consoleOn) console.log(data[0]);
        if (this.createPoints && data[0].extraData.worldCoords) {
          const returned = this.$refs.scaffold.$module.scene.createPoints(
            "test",
            "points",
            [data[0].extraData.worldCoords],
            undefined,
            0x0022ee,
          );
        }
        this.$refs.scaffold.showRegionTooltipWithAnnotations(data, false, true);
      }
    },
    onHelpModeShowNext: function () {
      this.helpModeActiveItem += 1;
    },
    onHelpModeLastItem: function (isLastItem) {
      if (isLastItem) {
        this.helpModeLastItem = true;
      }
    },
    onFinishHelpMode: function () {
      this.helpMode = false;
      // reset help mode to default values
      this.helpModeActiveItem = 0;
      this.helpModeLastItem = false;
    },
    onTooltipShown: function () {
      if (this.$refs.scaffold && this.$refs.scaffoldHelp) {
        this.$refs.scaffoldHelp.toggleTooltipHighlight();
      }
    },
    onMapTooltipShown: function () {
      if (this.$refs.scaffold && this.$refs.scaffoldHelp) {
        this.$refs.scaffoldHelp.toggleTooltipPinHighlight();
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
/* Component Styles */
</style>
