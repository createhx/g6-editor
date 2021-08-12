<template>
  <ul>
    <li v-for="(item,index) in list"
        :key="index"
        class="getItem"
        :data-shape="item.shape"
        :data-type="item.type"
        :data-size="item.size"
        draggable
        @dragstart="handleDragstart"
        @dragend="handleDragEnd($event,item)">
      <span class="pannel-type-icon"
            :style="{background:'url('+item.image+')'}"></span>
      {{item.name}}
    </li>
  </ul>
</template>

<script>
import eventBus from "@/utils/eventBus";
import okSvg from "@/assets/icons/ok.svg";
import bgImg from "@/assets/bg.jpg";
export default {
  data () {
    return {
      page: null,
      command: null,
      offsetX: 0,
      offsetY: 0,
      // 菜单栏
      list: [
        {
          name: "语音录入",
          label: "语音录入",//语音录入
          size: "170*34",
          type: "node",
          x: 0,
          y: 0,
          shape: "customNode",
          inPoints: [[0, 0.5]],
          outPoints: [[1, 0.5]],
          props: {//左列表格
            nodeType: 3,
            dec: "",
            voice: ""
          }
        },
        {
          name: "条件节点",
          label: "条件节点",//条件节点
          size: "170*34",
          type: "node",
          x: 0,
          y: 0,
          shape: "customNode",
          inPoints: [[0, 0.5]],
          outPoints: [[1, 0.4], [1, 0.6]],
          props: {//左列表格
            nodeType: 4,
            dec: "",
            voice: ""
          }
        },
        {
          name: "开始节点",
          label: "开始节点",//开始节点
          size: "170*34",
          type: "node",
          x: 0,
          y: 0,
          shape: "customNode",
          inPoints: [[0, 0.5]],
          outPoints: [[1, 0.5]],
          isDoingStart: true,
          props: {//左列表格
            nodeType: 1,
            dec: "",
            voice: ""
          }
        },
        {
          name: "结束节点",
          label: "结束节点",//结束节点
          size: "170*34",
          type: "node",
          x: 0,
          y: 0,
          shape: "customNode",
          inPoints: [[0, 0.5]],
          outPoints: [[1, 0.5]],
          isDoingEnd: true,
          props: {//左列表格
            nodeType: 2,
            dec: "",
            voice: ""
          }
        }
      ]
    };
  },
  created () {
    this.bindEvent();
  },
  methods: {
    handleDragstart (e) {
      this.offsetX = e.offsetX;
      this.offsetY = e.offsetY;
    },
    handleDragEnd (e, item) {
      let data = {};
      Object.assign(data, item);
      data.offsetX = this.offsetX;
      data.offsetY = this.offsetY;
      if (this.page) {
        const graph = this.page.graph;
        // const size = e.target.dataset.size.split("*");
        const xy = graph.getPointByClient(e.x, e.y);
        data.x = xy.x;
        data.y = xy.y;
        data.size = item.size.split("*");
        data.type = "node";
        this.command.executeCommand("add", [data]);
      }
    },
    bindEvent () {
      // 接收方通过 $on监听自定义事件的callback接收数据
      eventBus.$on("afterAddPage", page => {
        this.page = page;
        this.command = page.command;
      });
    }
  }
};
</script>

<style scoped>
.itempannel {
  height: 100%;
  position: absolute;
  left: 0px;
  z-index: 2;
  background: #f7f9fb;
  width: 200px;
  padding-top: 8px;
  border-right: 1px solid #e6e9ed;
}
.itempannel ul {
  padding: 0px;
  padding-left: 16px;
}
.itempannel li {
  color: rgba(0, 0, 0, 0.65);
  border-radius: 4px;
  width: 160px;
  height: 28px;
  line-height: 26px;
  padding-left: 8px;
  border: 1px solid rgba(0, 0, 0, 0);
  list-style-type: none;
}
.itempannel li:hover {
  background: white;
  border: 1px solid #ced4d9;
  cursor: move;
}

.itempannel .pannel-type-icon {
  width: 16px;
  height: 16px;
  display: inline-block;
  vertical-align: middle;
  margin-right: 8px;
}
</style>