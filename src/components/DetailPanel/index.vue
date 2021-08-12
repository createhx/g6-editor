<template>
  <div class="detailpannel">
    <div>
      <div v-if="status=='node-selected'"
           class="pannel"
           id="node_detailpannel">
        <el-tabs v-model="activeName">
          <el-tab-pane label="用户管理"
                       name="first">
            <div class="block-container">
              <el-row :gutter="10">
                <el-col :span="8">节点名称</el-col>
                <el-col :span="16">
                  <el-input v-model="node.name"
                            disabled />
                </el-col>
                <el-col :span="8">标题</el-col>
                <el-col :span="16">
                  <el-input v-model="node.label"
                            @change="handleChangeName" />
                </el-col>
                <el-col :span="8">文本信息</el-col>
                <el-col :span="16">
                  <el-input v-model="node.props.dec" />
                </el-col>
                <el-col :span="8">语音信息</el-col>
                <el-col :span="16">
                  <el-input v-model="node.props.voice" />
                </el-col>
                <el-col :span="8">标签</el-col>
                <el-col :span="16">
                  <el-input v-model="node.label" />
                </el-col>
              </el-row>
            </div>
          </el-tab-pane>
          <el-tab-pane label="配置管理"
                       name="second">全局设置</el-tab-pane>
        </el-tabs>
      </div>
      <div v-if="status==='canvas-selected'"
           class="pannel"
           id="canvas_detailpannel">
        <div class="pannel-title">画布</div>
        <div class="block-container">
          <el-checkbox v-model="showGrid"
                       @change="changeGridState">网格对齐</el-checkbox>
        </div>
      </div>
      <!-- 连线 -->
      <div v-if="status === 'edge-selected'"
           id="edge_detailpannel"
           class="pannel">
        <div class="pannel-title">连线</div>
        <div class="block-container">
          <el-col :span="8">内容</el-col>
          <el-col :span="16">
            <el-input v-model="edge.label"
                      @change="handleChange" />
          </el-col>
          <el-col :span="8">文字颜色</el-col>
          <el-col :span="16">
            <el-color-picker v-model="textColor"
                             @change="handleChangeColor" />
          </el-col>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import eventBus from "@/utils/eventBus";
import Grid from "@antv/g6/build/grid";
export default {
  data () {
    return {
      status: "canvas-selected",
      showGrid: false,
      activeName: 'first',
      page: {},
      graph: {},
      item: {},
      node: {},
      edge: {},
      grid: null,
      textColor: 'rgba(19, 206, 102, 0.8)'
    };
  },
  created () {
    this.init();
    this.bindEvent();
  },
  methods: {
    init () { },
    bindEvent () {
      let self = this;
      eventBus.$on("afterAddPage", page => {
        self.page = page;
        self.graph = self.page.graph;
        eventBus.$on("nodeselectchange", item => {
          if (item.select === true && item.target.getType() === "node") {
            self.status = "node-selected";
            self.item = item.target;
            self.node = item.target.getModel();//获取节点
          } else if (item.select === true && item.target.getType() === "edge") {
            self.status = "edge-selected";
            self.item = item.target;
            self.edge = item.target.getModel();
          } else {
            self.status = "canvas-selected";
            self.item = null;
            self.node = null;
          }
        });
      });
    },
    handleChangeName (e) {
      const model = {
        label: e
      };

      this.graph.update(this.item, model);
    },
    changeGridState (value) {
      if (value) {
        this.grid = new Grid();//网格
        this.graph.addPlugin(this.grid);
      } else {
        this.graph.removePlugin(this.grid);
      }
    },
    // add 
    handleChange (e) {
      const model = {
        label: e
      };
      console.log(model)
      this.graph.update(this.item, model);
    },
    handleChangeColor (e) {
      const model = {
        textColor: e
      };
      this.graph.update(this.item, model);
    }
  }
};
</script>

<style scoped>
.detailpannel {
  height: 100%;
  position: absolute;
  right: 0px;
  z-index: 2;
  background: #f7f9fb;
  width: 200px;
  border-left: 1px solid #e6e9ed;
}
.detailpannel .block-container {
  padding: 16px 8px;
}
.block-container .el-col {
  height: 28px;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.pannel-title {
  height: 32px;
  border-top: 1px solid #dce3e8;
  border-bottom: 1px solid #dce3e8;
  background: #ebeef2;
  color: #000;
  line-height: 28px;
  padding-left: 12px;
}
</style>
