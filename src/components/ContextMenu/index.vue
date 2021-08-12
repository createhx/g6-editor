<template>
  <div>
    <ul class="el-scrollbar__view el-select-dropdown__list context-menu"
        ref="contextMenu">
      <li class="el-select-dropdown__item"
          v-for="menu in menus"
          :key="menu.key"
          @click="handleDelete(menu)">{{menu.name}}</li>
    </ul>
  </div>
</template>

<script>
import eventBus from "@/utils/eventBus";
export default {
  data () {
    return {
      menus: [{ key: 1, name: "删除" }],
      selectedItem: null,
      command: null,
      graph: {},
    };
  },
  created () {
    this.init();
    this.bindEvent();
  },
  methods: {
    init () {
      const { editor, command } = this.$parent;
      this.editor = editor;
      this.command = command;
    },
    bindEvent () {
      let self = this;
      eventBus.$on("afterAddPage", page => {
        self.page = page;
        self.graph = self.page.graph;
      });
      eventBus.$on("contextmenuClick", e => {
        const menu = this.$refs.contextMenu;
        menu.style.left = e.clientX + "px";
        menu.style.top = e.clientY + "px";
        menu.style.display = "block";
      });
      eventBus.$on("mousedown", () => {
        const menu = this.$refs.contextMenu;
        menu.style.display = "none";
      });
      eventBus.$on("nodeselectchange", () => {
        this.selectedItem = this.graph.findAllByState("node", "selected");
        this.selectedItem = this.selectedItem.concat(
          ...this.graph.findAllByState("edge", "selected")
        );
      });
      eventBus.$on("deleteItem", () => {
        this.handleDelete();
      });
    },
    handleDelete (item) {
      // alert(item.name);
      const menu = this.$refs.contextMenu;
      menu.style.display = "none";
      if (this.selectedItem.length > 0) {
        this.command.executeCommand("delete", this.selectedItem);
        this.selectedItem = null;
      }
    }
  }
};
</script>

<style>
.context-menu {
  position: absolute;
  border: 1px solid #e4e7ed;
  border-radius: 4px;
  background-color: #fff;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  box-sizing: border-box;
  margin: 5px 0;
  z-index: 1;
  display: none;
}

.context-menu li {
  cursor: pointer;
  font-size: 12px;
  height: 28px;
  line-height: 28px;
}
.context-menu li:hover {
  background-color: #f5f7fa;
}
</style>