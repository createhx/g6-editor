<template>
  <!-- 画布 -->
  <div class="page">
    <div :id="pageId"
         class="graph-container"
         style="position: relative;"></div>
  </div>
</template>


<script>
import G6 from "@antv/g6/build/g6";
import { initBehavors } from "@/behavior";
export default {
  data () {
    return {
      pageId: "graph-container",
      graph: null
    };
  },
  props: {
    height: {
      type: Number,
      default: 0
    },
    width: {
      type: Number,
      default: 0
    },
    data: {
      type: Object,
      default: () => { }
    }
  },
  created () {
    initBehavors();
  },
  mounted () {
    this.$nextTick(() => {
      this.init();
    });
  },
  methods: {
    init () {
      const height = this.height - 42
      const width = this.width - 400

      this.graph = new G6.Graph({
        container: "graph-container",
        height: height,
        width: width,
        modes: {
          // 支持的 behavior
          default: [
            "drag-canvas",
            "zoom-canvas",
            "hover-node",
            "select-node",
            "hover-edge",
            "keyboard",
            "customer-events",
            "add-menu"
          ],
          mulitSelect: ["mulit-select"],
          addEdge: ["add-edge"],
          moveNode: ["drag-item"]
        },
        // layout: {
        //   type: 'radial',
        //   unitRadius: 50,
        //   center: [500, 300]
        // }

      });
      const { editor, command } = this.$parent;
      editor.emit("afterAddPage", { graph: this.graph, command });

      this.readData();
    },
    readData () {
      let data = this.data;
      data = JSON.parse(localStorage.getItem("data"))
      if (data) {
        // 渲染节点
        this.graph.read(data);
      }
    }
  }
};
</script>

<style scoped>
.page {
  margin-left: 200px;
  margin-right: 200px;
}
</style>