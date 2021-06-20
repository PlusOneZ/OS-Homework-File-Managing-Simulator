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
    <el-button @click="haha">
      click
    </el-button>
  </div>
</template>

<script>

const DOC = 1
const DIRECTORY = 0

function getParentDir(path) {
  var i = path.lastIndexOf('/')
  return path.substring(0, i+1)
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
    addFile(label, type, nodeKey) {
      var node = undefined
      if (!nodeKey)
        node = this.$refs.tree.getCurrentNode()
      else
        node = this.$refs.tree.getNode(nodeKey)
      var key = node.data.key + label
      if (this.$refs.tree.getNode(key)) {
        console.log("duplicate")
        return false
      }
      if (node.type === DOC) {
        this.$refs.tree.insertAfter(
            {
              'label': label,
              'key': node.data.key + label,
              'type': type,
            },
            node.key
        )
      } else {
        this.$refs.tree.append(
            {
              'label': label,
              'key': node.data.key + label,
              'type': type,
            },
            node.key
        )
      }
    },
    haha() {
      console.log(this.$refs.tree.getNode('root/'))
      this.addFile('haha', DOC, 'root/')
    },
    doubleClick() {
      // console.log(this.$refs.tree.getCurrentNode())
      this.openNode = this.$refs.tree.getCurrentNode()
      console.log(this.openNode)
      if (this.openNode.type === DIRECTORY) {
        this.$emit("dirChanged", this.openNode)
      } else if (this.openNode.type === DOC) {
        let path = getParentDir(this.openNode.key)
        console.log(path)
        this.$emit("fileOpened", this.openNode)
        this.openNode = this.$refs.tree.getNode(path)
        this.$emit("dirChanged", this.openNode)
      }
    }
  },
  mounted() {
    this.resetData()
    console.log('reset OK')
  },
}
</script>

<style scoped>


</style>