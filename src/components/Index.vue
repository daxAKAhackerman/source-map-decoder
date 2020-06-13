<template>
  <b-container>
    <b-row>
      <b-col>
        <h1>Source Map Decoder</h1>
        <hr />
        <b-alert variant="danger" :show="showAlert">{{ alertMessage }}</b-alert>
        <b-collapse v-model="textareaVisible" id="collapse-textarea">
          <b-form-textarea v-model="sourceMap" rows="20" max-rows="20"></b-form-textarea>
          <br />
        </b-collapse>
        <b-button @click="parseSourceMap" variant="outline-primary">Decode</b-button>
        <b-button @click="cleanUp" variant="outline-secondary">Reset</b-button>
        <br />
        <br />
        <ul>
          <li>Paste in the full content of the ".map" file and press decode.</li>
          <li>Click on the table row that corresponds to the file that you wish to view.</li>
          <li>
            If you want your own Source Map Decoder, the source code and build instructions can be found
            <b-link href="https://github.com/daxAKAhackerman/source-map-decoder">here</b-link>.
          </li>
        </ul>
        <hr />
        <b-table
          v-if="fileList.length != 0"
          striped
          hover
          :items="fileList"
          :fields="fields"
          sort-by.sync="name"
          sort-direction="asc"
          @row-clicked="item=>$set(item, '_showDetails', !item._showDetails)"
        >
          <template v-slot:row-details="row">
            <vue-code-highlight class="codeWrapper" :language="row.item.type">{{ row.item.code }}</vue-code-highlight>
          </template>
        </b-table>
      </b-col>
    </b-row>
  </b-container>
</template>
<script>
import { component as VueCodeHighlight } from "vue-code-highlight";
export default {
  name: "Index",
  components: {
    VueCodeHighlight
  },
  data() {
    return {
      fields: [
        {
          key: "name",
          sortable: true,
          label: "File name"
        }
      ],
      textareaVisible: true,
      alertMessage: "asdasdas",
      showAlert: false,
      sourceMap: "",
      sourceMapObject: {},
      fileList: []
    };
  },
  methods: {
    parseSourceMap() {
      this.fileList = [];
      this.textareaVisible = false;
      try {
        this.sourceMapObject = JSON.parse(this.sourceMap);
      } catch (err) {
        this.alertMessage = "Invalid format";
        this.showAlert = true;
        return;
      }
      this.showAlert = false;
      this.alertMessage = "";
      this.sourceMapObject["sources"].forEach(this.feedFileList);
    },
    feedFileList(item, index) {
      let fileObject = {};
      let extension = item.split(".").slice(-1)[0];
      extension = extension.split("?")[0];
      if (/^\w+$/.test(extension) !== true) {
        extension = "js";
      }
      if (extension == "vue") {
        extension = "html";
      }

      fileObject = {
        name: item,
        code: this.sourceMapObject["sourcesContent"][index],
        type: extension
      };
      this.fileList.push(fileObject);
    },
    cleanUp() {
      this.textareaVisible = true;
      this.sourceMap = "";
      this.fileList = [];
      this.sourceMapObject = {};
      this.showAlert = false;
      this.alertMessage = "";
    }
  }
};
</script>
<style>
.codeWrapper code {
  overflow-x: auto !important;
  white-space: pre-wrap !important;
  white-space: -moz-pre-wrap !important;
  white-space: -pre-wrap !important;
  white-space: -o-pre-wrap !important;
  word-wrap: break-word !important;
}
textarea {
  background-color: #edf6ff !important;
}
</style>