<template>
  <div class="text-lg pt-5 max-h-full">
    <el-button @click="quickAdd">
      Quick Add
    </el-button>
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
            {{ node.label }}
          </span>
        </span>
      </template>
    </el-tree>
  </div>
</template>

<script>

import {ElMessage} from "element-plus";

const DOC = 1
const DIRECTORY = 0

Date.prototype.format = function (fmt) {
  var o = {
    "M+": this.getMonth() + 1, //月份
    "d+": this.getDate(), //日
    "h+": this.getHours(), //小时
    "m+": this.getMinutes(), //分
    "s+": this.getSeconds(), //秒
    "q+": Math.floor((this.getMonth() + 3) / 3), //季度
    "S": this.getMilliseconds() //毫秒
  };
  if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
  for (var k in o)
    if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
  return fmt;
}

function getParentDir(path) {
  var i = path.lastIndexOf('/')
  return path.substring(0, i + 1)
}

export default {
  name: "EntireDirectory",
  props: {
    data: Object,
    user: String,
    current: String,
    disk: Object
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

    calculateSize(text) {
      let count = text.length;
      for (let i = 0; i < text.length; i++) {
        if (text.charCodeAt(i) > 255) {
          count++;
        }
      }
      // console.log('in calculate: ', count, count /1024)
      return (count / 1024);
    },

    addFile(label, type, nodeKey, docContent) {
      console.log(nodeKey)

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
        key += '.txt'
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
        'creator': this.user,
        'date': (new Date()).format('yyyy/MM/dd hh:mm:ss')
      }

      if (type === DIRECTORY) {
        data['children'] = []
        data['size'] = 0
      } else {
        data['content'] = docContent
        data['size'] = this.calculateSize(docContent)
        data['blocks'] = []
        if (data.size === 0) {
          data.size = this.calculateSize('1')
        }
      }

      // console.log(data['size'])

      let blockNeeded = Math.ceil(data.size / this.disk.block_size)
      // console.log("blocks:" , blockNeeded, data.size / this.disk.block_amount)

      if (nodeKey.substr(-1, 1) !== '/') {
        if (((data.type === DOC) && this.allocateBlocks(blockNeeded, data)) || data.type === DIRECTORY) {
          this.insertSameLevel(data, nodeKey)
          ElMessage({
            showClose: true,
            message: '创建成功！',
            type: 'success'
          })
        }
      } else {
        if (((data.type === DOC) && this.allocateBlocks(blockNeeded, data)) || data.type === DIRECTORY) {
          this.append(data, nodeKey)
          ElMessage({
            showClose: true,
            message: '创建成功！',
            type: 'success'
          })
        }
      }
      setTimeout(() => {
        if (data.size > 0) {
          this.$emit('changeSizeOfPath', data.key, data.size)
        }
      }, 500)
    },

    allocateBlocks(num, data) {
      console.log(num)
      if (num > this.disk.available) {
        ElMessage({
          showClose: true,
          message: '空间不足',
          type: 'error'
        })
        return false
      }
      for (let i = 0; i < num; i++) {
        console.log("free_pointer ", this.disk.free_pointer)
        data.blocks.push(this.disk.free_pointer)
        this.disk.bit_table[this.disk.free_pointer] = true
        this.disk.storage[this.disk.free_pointer] = data.key
        this.disk.available--
        this.findNextDiskFreeSpace()
      }
      return true
    },

    findNextDiskFreeSpace() {
      for (let i = 0; i < this.disk.block_amount; i++) {
        if (this.disk.bit_table[this.disk.free_pointer] === false) {
          return
        }
        this.disk.free_pointer = (this.disk.free_pointer + 1) % this.disk.block_amount
      }
    },

    quickAdd() {
      let type = Math.floor(Math.random() * 2)
      if (type === DIRECTORY) {
        this.addFile((new Date()).format("h-m-s.S M-d".substr(0, 21)), type, this.current)
      } else {
        this.addFile((new Date()).format("h-m-s.S M-d".substr(0, 21)), type, this.current, '')
      }
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
.el-tree {
  max-height: 90%;
  overflow: auto;
}


</style>