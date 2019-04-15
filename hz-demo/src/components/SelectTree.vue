<template>
  <el-popover placement="bottom-start"  @hide="popoverHide" trigger="click" v-model="isShowSelect">
    <el-tree
      v-bind:style="{width: (width*1-25)+'px'}"
      style="height:300px;overflow-y: scroll;"
      v-show="isShowSelect"
      :data="treeData"
      :check-strictly="true"
      :node-key="nodeKey"
      :show-checkbox="multiple"
      :expand-on-click-node="multiple"
      :default-expanded-keys="defaultExpandedKeys"
      :default-checked-keys="defaultCheckedKeys"
      :default-expand-all="false"
      highlight-current
      :filter-node-method="filterNode"
      @node-click="handleNodeClick"
      @check="getKeys"
      ref="tree"
      :props="selectTreeProps"
    ></el-tree>
    <el-select  v-bind:style="{width: width+'px'}" popper-class="selectOptionDiv"
               slot="reference" :filter-method="filterElementValue" filterable ref="select" v-model="key" size="small"
               :multiple="multiple" :placeholder="tipText" @visible-change="changeFunc" @click="isShowSelect = !isShowSelect">
      <el-option v-for="item in options" :key="item.value" :label="item.text" :value="item.value"></el-option>
    </el-select>
  </el-popover>
</template>
<script>
  import { validatenull } from '@/utils/validate'
  export default {
    name: 'SelectTree',
    props: {
      treeData: { type: Array, required: true },
      defaultExpandAll: {
        type: Boolean,
        default: false
      },
      selectTreeProps: {
        type: Object,
        default: function() {
          return {
            children: 'children',
            label: 'text',
            disabled: function(data, node) {
              return data.type === '1'
            }
          }
        }
      },
      popoverHide: {
        type: Function,
        default: function() { }
      },
      multiple: {
        type: Boolean,
        default: false
      },
      width: { type: Number, default: 300 },
      id: [String, Array],
      nodeKey: { type: String, default: 'id' },
      tipText: { type: String, default: '请选择' }
    },
    created() {
      console.log(this.treeData)
    },
    data() {
      return {
        // 是否显示树状选择器
        isShowSelect: false,
        options: [],
        key: [],
        nodeInfos: [],
        showValueTmp: '',
        defaultExpandedKeys: [],
        defaultCheckedKeys: []
      }
    },
    watch: {
      isShowSelect(val) {
        // 隐藏select自带的下拉框
        // this.$refs.select.blur()
        if (val) {
          this.$nextTick(() => {
            const divs = document.getElementsByClassName('selectOptionDiv')
            for (let i = 0; i < divs.length; i++) {
              divs[i].style.display = 'none'
            }
          })
          // 下拉面板展开-选中节点-展开节点
          if (this.multiple) {
            this.setTreeCheckNode(this.key)
            this.defaultCheckedKeys = this.key
            this.defaultExpandedKeys = this.key
          }
        }
      },
      key(val) {
        this.$emit('update:id', this.key)
      },
      id(val) {
        this.key = this.id
        if (this.multiple && Array.isArray(this.id)) {
          this.setTreeCheckNode(this.key)
        } else {
          this.setTreeCheckNode([this.key])
        }
      }
    },
    mounted() {
      // 把传进来的参数转成数组处理
      this.key = this.id
      if (this.multiple && Array.isArray(this.id)) {
        this.setTreeCheckNode(this.key)
      } else {
        this.setTreeCheckNode([this.key])
      }
    },
    methods: {
      hideOptions() {
        // const parent = document.getElementsByTagName('body')[0]
        // const child = document.getElementsByClassName('selectOptionDiv')[0]
        // parent.removeChild(child)
      },
      changeFunc() {
        this.$nextTick(() => {
          const divs = document.getElementsByClassName('selectOptionDiv')
          for (let i = 0; i < divs.length; i++) {
            divs[i].style.display = 'none'
          }
        })
      },
      handleNodeClick(data, datax) {
        if (!this.multiple) { // 单选进来
          if (this.selectTreeProps.disabled(data)) { // 根据设置 无视点击disabled的节点的事件
            return
          }
          const tmpMap = {}
          tmpMap.value = data.id
          tmpMap.text = data.text
          this.key = data.id
          this.options = []
          this.options.push(tmpMap)
          this.isShowSelect = !this.isShowSelect
          this.$emit('treeSelect', datax)
        }
      },
      getKeys(data, checked) {
        const tmp = []
        checked.checkedNodes.forEach(node => {
          const tmpMap = {}
          tmpMap.value = node.id
          tmpMap.text = node.text
          tmp.push(tmpMap)
        })
        this.options = tmp
        this.key = checked.checkedKeys
        // debugger
      },
      setTreeCheckNode(ids) {
        const tmp = []
        if (this.multiple) {
          this.nodeInfos = []
          ids.forEach(id => {
            this.findTreeNode(this.treeData, id)
            tmp.push(this.showValueTmp)
          })
          this.options = this.nodeInfos
          this.key = this.id
          this.$refs.tree.setCheckedKeys(this.key)
        } else {
          this.nodeInfos = []
          ids.forEach(id => {
            this.findTreeNode(this.treeData, id)
            tmp.push(this.showValueTmp)
          })
          this.options = this.nodeInfos
          this.key = this.id
        }
      },
      // 递归查询树形节点
      findTreeNode(tree, id) {
        if ((!validatenull(tree)) && (!validatenull(id))) {
          for (var i = 0; i < tree.length; i++) {
            if (tree[i].id === id) {
              this.nodeInfos.push({ value: id, text: tree[i].text })
              this.showValueTmp = tree[i].text
            } else if (tree[i].children != null && tree[i].children.length > 0) {
              this.findTreeNode(tree[i].children, id)
            }
          }
        }
      },
      filterElementValue(data) {
        if (this.$refs.tree) {
          this.$refs.tree.filter(data)
        }
      },
      filterNode(value, data) {
        if (!value) return true
        return data.text.indexOf(value) !== -1 || data.pyCode.indexOf(value.toUpperCase()) !== -1
      }
    }
  }
</script>

