<template>
  <div class="text-lg pt-5">
    <el-tree
        :data="data"
        node-key="key"
        default-expand-all
        draggable
        accordion
        :allow-drop="allowDrop"
        :allow-drag="allowDrag"
        @node-drag-end="dragEnd"
        ref="tree"
        @dblclick="doubleClick(node)"
    >
      <template #default="{ node, data }">
        <span class="custom-tree-node">
          <span>
            <i class="el-icon-folder" v-if="(data.type === 0) && (!node.expanded || node.isLeaf)"></i>
            <i class="el-icon-folder-opened" v-if="(data.type === 0) && node.expanded && !node.isLeaf"></i>
            <i class="el-icon-document" v-if="(data.type === 1)"></i>
            {{ node.label + "   " + data.key }}
          </span>
        </span>
      </template>
    </el-tree>
    <div class="h-5"></div>
    <el-button @click="add">
      Add Directory / File
    </el-button>
  </div>
</template>

<script>

import {ElMessage} from "element-plus";

const DOC = 1
const DIRECTORY = 0

function getParentDir(path) {
  var i = path.lastIndexOf('/')
  return path.substring(0, i + 1)
}

export default {
  name: "EntireDirectory",
  props: {
    data: Object
  },
  data() {
    return {
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      openNode: 'root/'
    };
  },

  methods: {
    allowDrop(draggingNode, dropNode, type) {
      if (!dropNode) {
        return false
      }
      if (dropNode.data.key === 'root/') {
        return (type !== 'prev') && (type !== 'next') && (type !== 'before');
      } else return dropNode.data.type !== DOC;
    },
    allowDrag(draggingNode) {
      if (draggingNode.label === 'root') return false;
      return draggingNode.data.label.indexOf('三级 3-2-2') === -1;
    },
    dragEnd() {
      this.resetData()
      console.log(this.data)
    },
    resetData() {
      console.log('reset')
      console.log(this.data[0])
      this.setKeyOf(this.data[0], "")
    },
    setKeyOf(node, parentKey) {
      console.log("node" + node.label)
      node['key'] = parentKey + node.label + ((node.type === DIRECTORY) ? '/' : '')
      if (node.isLeaf || !node.children) return
      for (const child of node.children) {
        console.log('child' + child)
        this.setKeyOf(child, node.key)
      }
    },

    addFile(label, type, nodeKey, docContent) {
      console.log(nodeKey)
      // var treeNode = this.$refs.tree.getNode(nodeKey)
      // console.log(treeNode)
      // if (!treeNode) {
      //   treeNode = this.$refs.tree.getCurrentNode()
      //   treeNode = treeNode? treeNode : this.$refs.tree.getNode('root/')
      // }

      if (!nodeKey) {
        nodeKey = this.$refs.tree.getCurrentKey()
        if (!nodeKey) {
          nodeKey = 'root/'
        }
      }

      var key = ''

      if (nodeKey.substr(-1, 1) !== '/') {
        let basepath = nodeKey.substring(0, nodeKey.lastIndexOf('/') + 1)
        key = basepath + label
      } else {
        key = nodeKey + label
      }

      if (type === DIRECTORY) {
        key += '/'
      } else {
        label += '.txt'
      }

      if (this.$refs.tree.getNode(key)) {
        console.log("duplicate")
        ElMessage({
          showClose: true,
          message: '创建失败，已经存在同名文件。',
          type: 'error'
        })
        return false
      }

      var data = {
        'label': label,
        'key': key,
        'type': type,
      }

      if (type === DIRECTORY) {
        data['children'] = []
      } else {
        data['content'] = docContent
      }

      if (nodeKey.substr(-1, 1) !== '/') {
        this.insertSameLevel(data, nodeKey)
      } else {
        this.append(data, nodeKey)
      }
      ElMessage({
        showClose: true,
        message: '创建成功！',
        type: 'success'
      })
    },

    add() {
      console.log(this.$refs.tree.getNode('root/dic1'))
      this.addFile('haha', DOC, 'root/file1')
    },

    doubleClick() {
      // console.log(this.$refs.tree.getCurrentNode())
      this.openNode = this.$refs.tree.getCurrentNode()
      console.log(this.openNode)
      if (this.openNode.type === DIRECTORY) {
        this.$emit("dirChanged", this.openNode)
      } else if (this.openNode.type === DOC) {
        let path = getParentDir(this.openNode.key)
        this.$emit("fileOpened", this.openNode)
        this.openNode = this.$refs.tree.getNode(path)
        this.$emit("dirChanged", this.openNode)
      }
    },

    changeDir(key) {
      let node = null
      // if (key.substr(-1, 1) === '/' && key !== 'root/') {
      //   node = this.$refs.tree.getNode(key.substring(0, key.length - 1))
      // } else {
        node = this.$refs.tree.getNode(key)
      // }
      console.log(node, 'in changDir of ED')
      if (node) {
        // this.$refs.tree.setCurrentNode(node)
        this.openNode = node.data
        this.$emit("dirChanged", this.openNode)
      }
    },

    findChildrenOfData(key, data) {
      if (data.key === key) {
        return data
      }
      console.log(data, 'in findChildrenOfData')
      if (!data.children) return false
      for (var i = 0; i < data.children.length; i++) {
        let t = this.findChildrenOfData(key, data.children[i])
        if (t) return t
      }
    },

    findParentOfData(key, data) {
      if (key === data.key) {
        return 1
      }
      console.log(data, 'in findParentOfData')
      if (!data.children) return false
      for (var i = 0; i < data.children.length; i++) {
        let t = this.findParentOfData(key, data.children[i])
        if (t) return data
      }
    },

    append(data, toNodeKey) {
      let d = this.findChildrenOfData(toNodeKey, this.data[0])
      console.log(d, 'in append')
      if (!d) {
        console.log("Error: cannot find" + toNodeKey);
        return
      }
      var c = d.children
      c.push(data)
    },

    insertSameLevel(data, toNodeKey) {
      let d = this.findParentOfData(toNodeKey, this.data[0])
      console.log(d, 'in insertSameLevel')
      if (!d) {
        console.log("Error: cannot find" + toNodeKey);
        return
      }
      var c = d.children
      c.push(data)
    }

  },
  mounted() {
    this.resetData()
    console.log('reset OK')
  },
  computed: {}
}
</script>

<style scoped>


</style>