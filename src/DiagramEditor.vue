<template>
  <div id="editor">
    <VButton v-if="!editable" @click="editable = true">Edit</VButton>
    <span v-else>
      <VButton @click="openModal">New Node</VButton>
      <VButton @click="endEdit">End</VButton>
    </span>
    <VButton @click="openInputModal">import/export</VButton>
    <VSelect v-model="scale">
      <option value="0.5">Small</option>
      <option value="1" selected>Medium</option>
      <option value="2">Large</option>
    </VSelect>
    <EditNodeModal
      :node="{ content: {} }"
      :isActive="isModalActive"
      @ok="addNode"
      @cancel="cancel"
    />
    <EditNodeModal
      :node="tmpNode"
      :isActive="isEditModalActive"
      @ok="editNode"
      @cancel="cancel"
    />
    <EditLinkModal
      :link="tmpLink"
      :isActive="isEditLinkModalActive"
      @ok="editLink"
      @cancel="cancel"
    />
    <InputModal
      :text="json"
      :isActive="isInputModalActive"
      @ok="importData"
      @cancel="cancel"
    />
    <Diagram
      :width="graphData.width || 2000"
      :height="graphData.height || 1000"
      :scale="scale"
      :background="graphData.background || '#fafafa'"
      :nodes="graphData.nodes"
      :links="graphData.links"
      :editable="editable"
      :labels="
        graphData.labels || {
          edit: 'Edit',
          remove: 'Remove',
          link: 'New Link',
          select: 'Select'
        }
      "
      @editNode="openNodeEdit"
      @editLink="openLinkEdit"
      @nodeChanged="nodeChanged"
      @linkChanged="linkChanged"
    >
    </Diagram>
  </div>
</template>

<script>
import Diagram from "./Diagram";
import EditNodeModal from "@/lib/EditNodeModal";
import EditLinkModal from "@/lib/EditLinkModal";
import InputModal from "@/lib/InputModal";
export default {
  name: "DiagramEditor",
  components: {
    Diagram,
    EditNodeModal,
    EditLinkModal,
    InputModal
  },
  props: {
    value: {
      type: Object,
      default: () => {
        return {
          width: 2000,
          height: 1000,
          background: "#fafafa",
          labels: {
            edit: "Edit",
            remove: "Remove",
            link: "New Link",
            select: "Select"
          },
          nodes: [],
          links: []
        };
      }
    }
  },
  computed: {
    graphData: {
      get() {
        return this.value;
      },
      set(val) {
        this.$emit("input", val);
      }
    }
  },
  data() {
    return {
      name: "",
      url: "",
      color: "",
      json: "",
      scale: "1",
      isModalActive: false,
      isEditModalActive: false,
      isEditLinkModalActive: false,
      isInputModalActive: false,
      editable: false,
      tmpNode: {
        id: "",
        shape: "rectangle",
        width: 0,
        height: 0,
        content: {
          text: "",
          url: "",
          color: ""
        }
      },
      tmpLink: {
        id: "",
        content: {
          color: "",
          pattern: "solid"
        }
      }
    };
  },
  methods: {
    generateID() {
      return (
        new Date().getTime().toString(16) +
        Math.floor(Math.random() * 1000000).toString(16)
      );
    },
    openModal() {
      this.isModalActive = true;
    },
    cancel() {
      this.isModalActive = false;
      this.isEditModalActive = false;
      this.isEditLinkModalActive = false;
      this.isInputModalActive = false;
    },
    addNode(item) {
      this.graphData.nodes.push({
        id: this.generateID(),
        content: {
          text: item.content.text,
          url: item.content.url,
          color: item.content.color
        },
        width: parseInt(item.width) || 150,
        height: parseInt(item.height) || 60,
        shape: item.shape,
        point: {
          x: 10,
          y: 100 + Math.random() * 100
        }
      });
      this.isModalActive = false;
    },
    openNodeEdit(item) {
      this.tmpNode.id = item.id;
      this.tmpNode.content.text = item.content.text;
      this.tmpNode.content.url = item.content.url;
      this.tmpNode.content.color = item.content.color;
      this.tmpNode.shape = item.shape;
      this.tmpNode.width = item.width;
      this.tmpNode.height = item.height;
      this.isModalActive = false;
      this.isEditModalActive = true;
    },
    editNode(item) {
      let tmp = this.graphData.nodes.find(x => x.id === item.id);
      tmp.content.text = item.content.text;
      tmp.content.url = item.content.url;
      tmp.content.color = item.content.color;
      tmp.shape = item.shape;
      tmp.width = parseInt(item.width);
      tmp.height = parseInt(item.height);
      this.isEditModalActive = false;
    },
    openLinkEdit(item) {
      this.tmpLink.id = item.id;
      this.tmpLink.content.color = item.content.color;
      this.tmpLink.content.pattern = item.content.pattern;
      this.isEditLinkModalActive = true;
    },
    editLink(item) {
      let tmp = this.graphData.links.find(x => x.id === item.id);
      tmp.color = item.content.color;
      tmp.pattern = item.content.pattern;
      this.isEditLinkModalActive = false;
    },
    endEdit() {
      this.editable = false;
    },
    nodeChanged(obj) {
      this.graphData.nodes = obj.nodes;
    },
    linkChanged(obj) {
      this.graphData.links = obj.links;
    },
    openInputModal() {
      this.isInputModalActive = true;
      this.json = JSON.stringify(this.graphData);
    },
    importData(value) {
      const obj = JSON.parse(value.text);
      if (obj) {
        this.graphData = obj;
        this.isInputModalActive = false;
      }
    }
  }
};
</script>
