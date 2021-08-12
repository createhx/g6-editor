<template>
  <div>
    <div class="super-flow-demo1">
      <div class="node-container">
        <span class="node-item"
              v-for="item in nodeItemList"
              :key="item.label"
              @mousedown="evt => nodeItemMouseDown(evt, item.value)">
          {{ item.label }}
        </span>
      </div>
      <div class="flow-container"
           ref="flowContainer">
        <super-flow ref="superFlow"
                    :node-list="nodeList"
                    :link-list="linkList"
                    :origin="origin"
                    :node-menu="nodeMenu"
                    :link-menu="linkMenu"
                    :link-base-style="linkBaseStyle"
                    :link-style="linkStyle"
                    :link-desc="linkDesc">
          <template v-slot:node="{meta}">
            <div @mouseup="nodeMouseUp"
                 @click="nodeClick(meta)"
                 class="flow-node ellipsis">
              {{ meta.name }}
            </div>
          </template>
        </super-flow>
      </div>
      <div class="message-container">
        <el-tabs v-model="activeName">
          <el-tab-pane label="用户管理"
                       name="first">
            <div :title="drawerConf.title"
                 :close-on-click-modal="false"
                 width="200px">
              <el-form @keyup.native.enter="settingSubmit"
                       @submit.native.prevent
                       ref="nodeSetting"
                       :model="nodeSetting">
                <el-form-item label="节点名称"
                              prop="name">
                  <el-input v-model="nodeSetting.name"
                            placeholder="请输入节点名称"
                            maxlength="30">
                  </el-input>
                </el-form-item>
                <el-form-item label="节点描述"
                              prop="desc">
                  <el-input v-model="nodeSetting.desc"
                            placeholder="请输入节点描述"
                            maxlength="30">
                  </el-input>
                </el-form-item>
              </el-form>
              <span slot="footer"
                    class="dialog-footer">
                <el-button @click="drawerConf.cancel">
                  取 消
                </el-button>
                <el-button type="primary"
                           @click="settingSubmit">
                  确 定
                </el-button>
              </span>
            </div>
          </el-tab-pane>
          <el-tab-pane label="配置管理"
                       name="second">配置管理</el-tab-pane>
        </el-tabs>
      </div>
    </div>

  </div>
</template>

<script>
import SuperFlow from 'vue-super-flow'
import 'vue-super-flow/lib/index.css'
const drawerType = {
  node: 0,
  link: 1
}

export default {
  components: { SuperFlow },
  data () {
    return {
      origin: [300, 300],
      nodeList: [],
      linkList: [],
      input: "",
      activeName: 'first',
      drawerType,
      drawerConf: {
        title: '',
        visible: false,
        type: null,
        info: null,
        open: (type, info) => {
          const conf = this.drawerConf
          conf.visible = true
          conf.type = type
          conf.info = info
          if (conf.type === drawerType.node) {
            conf.title = '节点'
            if (this.$refs.nodeSetting) this.$refs.nodeSetting.resetFields()
            this.$set(this.nodeSetting, 'name', info.meta.name)
            this.$set(this.nodeSetting, 'desc', info.meta.desc)
          } else {
            conf.title = '连线'
            if (this.$refs.linkSetting) this.$refs.linkSetting.resetFields()
            this.$set(this.linkSetting, 'desc', info.meta ? info.meta.desc : '')
          }
        },
        cancel: () => {
          this.drawerConf.visible = false
          if (this.drawerConf.type === drawerType.node) {
            this.$refs.nodeSetting.clearValidate()
          } else {
            this.$refs.linkSetting.clearValidate()
          }
        }
      },
      linkSetting: {
        desc: ''
      },
      nodeSetting: {
        name: '',
        desc: ''
      },
      // 记录拖拽信息
      dragConf: {
        isDown: false,
        isMove: false,
        offsetTop: 0,
        offsetLeft: 0,
        clientX: 0,
        clientY: 0,
        ele: null,
        info: null
      },
      // 菜单
      nodeItemList: [
        {
          label: '开始',
          value: () => ({
            width: 120,
            height: 40,
            meta: {
              label: '1',
              name: '开始',
              title: ""
            }
          })
        },
        {
          label: '结束',
          value: () => ({
            width: 120,
            height: 40,
            meta: {
              label: '2',
              name: '结束',
              title: ""
            }
          })
        },
        {
          label: '选择输入项',
          value: () => ({
            width: 120,
            height: 40,
            meta: {
              label: '3',
              name: '选择输入项',
              title: ""
            }
          })
        },
        {
          label: '拍照',
          value: () => ({
            width: 120,
            height: 40,
            meta: {
              label: '4',
              name: '拍照',
              title: ""
            }
          })
        }
      ],

      nodeMenu: [
        [
          {
            label: '删除',
            selected: node => {
              node.remove()
            }
          },
          {
            label: '编辑',
            selected: node => {
              this.drawerConf.open(drawerType.node, node)
            }
          }
        ]
      ],
      linkMenu: [
        [
          {
            label: '删除',
            selected: link => {
              link.remove()
            }
          },
          {
            label: '编辑',
            selected: link => {
              this.drawerConf.open(drawerType.link, link)
            }
          }
        ]
      ],

      linkBaseStyle: {
        color: '#666666',           // line 颜色
        hover: '#FF0000',           // line hover 的颜色
        textColor: '#666666',       // line 描述文字颜色
        textHover: '#FF0000',       // line 描述文字 hover 颜色
        font: '14px Arial',         // line 描述文字 字体设置 参考 canvas font
        dotted: false,              // 是否是虚线
        lineDash: [4, 4],           // 虚线时生效
        background: 'rgba(255,255,255,0.6)'    // 描述文字背景色
      },
      fontList: [
        '14px Arial',
        'italic small-caps bold 12px arial'
      ]
    }
  },
  created () {
    // const nodeList = [
    //   {
    //     id: 'nodeS3WgFnzCI15X58Qw',
    //     width: 120,
    //     height: 40,
    //     coordinate: [10, 28],
    //     meta: {
    //       label: '1',
    //       name: '开始',
    //       title: "12"
    //     }
    //   },
    // ]
    const nodeList = JSON.parse(window.localStorage.getItem("nodeList"))
    const linkList = JSON.parse(window.localStorage.getItem("linkList"))
    if (nodeList) {
      this.nodeList = nodeList
    } else (
      this.nodeList = []
    )
    if (linkList) {
      this.linkList = linkList
    } else (
      this.linkList = []
    )
  },
  mounted () {
    document.addEventListener('mousemove', this.docMousemove)
    document.addEventListener('mouseup', this.docMouseup)
    // this.$once('hook:beforeDestroy', () => {
    //   document.removeEventListener('mousemove', this.docMousemove)
    //   document.removeEventListener('mouseup', this.docMouseup)
    // })
  },
  methods: {
    linkStyle (link) {
      if (link.meta && link.meta.desc === '1') {
        return {
          color: 'red',
          hover: '#FF00FF',
          dotted: true
        }
      } else {
        return {}
      }
    },
    linkDesc (link) {
      return link.meta ? link.meta.desc : ''
    },
    settingSubmit () {
      console.log("****settingSubmit***")
      // console.log("this.drawerConf", this.drawerConf)
      console.log("***superFlow.graph", this.$refs.superFlow.graph)
      const { nodeList, linkList } = this.$refs.superFlow.graph
      this.nodeList = nodeList.map(item => {
        return Object.assign({}, {
          id: item.id,
          width: item.width,
          height: item.height,
          coordinate: item.coordinate,
          meta: item.meta
        })
      })

      this.linkList = linkList.map(item => {
        return Object.assign({}, {
          id: item.id,
          startId: item.start.id,
          endId: item.end.id,
          startAt: item.startAt,
          endAt: item.endAt,
          meta: item.meta
        })
      })
      // 模拟后端数据
      window.localStorage.setItem("nodeList", JSON.stringify(this.nodeList))
      window.localStorage.setItem("linkList", JSON.stringify(this.linkList))
      // 保存之后传递给后端
      console.log(this.nodeList)
      console.log(this.linkList)
      // node setting
      // const conf = this.drawerConf
      // if (!conf.info.meta) conf.info.meta = {}
      // Object.keys(this.nodeSetting).forEach(key => {
      //   this.$set(conf.info.meta, key, this.nodeSetting[key])
      // })
      // this.$refs.nodeSetting.resetFields()
      // conf.visible = false
    },
    nodeMouseUp (evt) {
      evt.preventDefault()
    },
    nodeClick (meta) {
      // console.log("****nodeClick 触发")
      // console.log("***superFlow.graph", this.$refs.superFlow.graph)
      // console.log("点击节点", this.$refs.nodeName)
      this.nodeSetting.name = meta.name
    },
    docMousemove ({ clientX, clientY }) {
      const conf = this.dragConf

      if (conf.isMove) {

        conf.ele.style.top = clientY - conf.offsetTop + 'px'
        conf.ele.style.left = clientX - conf.offsetLeft + 'px'

      } else if (conf.isDown) {

        // 鼠标移动量大于 5 时 移动状态生效
        conf.isMove =
          Math.abs(clientX - conf.clientX) > 5
          || Math.abs(clientY - conf.clientY) > 5

      }
    },
    docMouseup ({ clientX, clientY }) {
      // console.log("docMouseup 触发")
      const conf = this.dragConf
      conf.isDown = false

      if (conf.isMove) {
        const {
          top,
          right,
          bottom,
          left
        } = this.$refs.flowContainer.getBoundingClientRect()

        // 判断鼠标是否进入 flow container
        if (
          clientX > left
          && clientX < right
          && clientY > top
          && clientY < bottom
        ) {

          // 获取拖动元素左上角相对 super flow 区域原点坐标
          const coordinate = this.$refs.superFlow.getMouseCoordinate(
            clientX - conf.offsetLeft,
            clientY - conf.offsetTop
          )

          // 添加节点
          this.$refs.superFlow.addNode({
            coordinate,
            ...conf.info
          })
          // this.nodeList.concat([coordinate,
          //   ...conf.info])
        }

        conf.isMove = false
      }

      if (conf.ele) {
        conf.ele.remove()
        conf.ele = null
      }
    },
    nodeItemMouseDown (evt, infoFun) {
      console.log("nodeClick 触发")
      // console.log(evt)
      // console.log(infoFun)
      // const info = infoFun()
      const {
        clientX,
        clientY,
        currentTarget
      } = evt
      // console.log("evt.currentTarget.getBoundingClientRect()-----")
      // console.log(evt.currentTarget.getBoundingClientRect())
      const {
        top,
        left
      } = evt.currentTarget.getBoundingClientRect()

      const conf = this.dragConf
      const ele = currentTarget.cloneNode(true)

      Object.assign(this.dragConf, {
        offsetLeft: clientX - left,
        offsetTop: clientY - top,
        clientX: clientX,
        clientY: clientY,
        info: infoFun(),
        ele,
        isDown: true
      })
      console.log("this.dragConf", this.dragConf)
      ele.style.position = 'fixed'
      ele.style.margin = '0'
      ele.style.top = clientY - conf.offsetTop + 'px'
      ele.style.left = clientX - conf.offsetLeft + 'px'
      this.$el.appendChild(this.dragConf.ele)
    }
  }
}
</script>

<style lang="less" scoped>
.ellipsis {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
  word-wrap: break-word;
}

.link-base-style-form {
  .el-form-item {
    margin-bottom: 12px;
  }

  padding-bottom: 20px;
  border-bottom: 1px solid #dcdcdc;
}

.super-flow-demo1 {
  margin-top: 20px;
  width: 100%;
  height: 800px;
  background-color: #f5f5f5;
  @list-width: 200px;

  > .node-container {
    width: @list-width;
    float: left;
    height: 100%;
    text-align: center;
    background-color: #ffffff;
  }

  > .flow-container {
    width: calc(100% - @list-width - @list-width);
    float: left;
    height: 100%;
    overflow: hidden;
  }
  > .message-container {
    width: @list-width;
    float: left;
    height: 100%;
    background-color: #ffffff;
  }
  .super-flow__node {
    .flow-node {
      box-sizing: border-box;
      width: 100%;
      height: 100%;
      line-height: 40px;
      padding: 0 6px;
      font-size: 12px;
    }
  }
}

.node-item {
  @node-item-height: 30px;

  font-size: 14px;
  display: inline-block;
  height: @node-item-height;
  width: 120px;
  margin-top: 20px;
  background-color: #ffffff;
  line-height: @node-item-height;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  user-select: none;
  text-align: center;
  z-index: 6;

  &:hover {
    box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.4);
  }
}
</style>