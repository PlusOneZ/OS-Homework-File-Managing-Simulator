<template>
  <div class="justify-center content-center half bg-gray-700">
    <div class="font-mono text-white" style="width: 100%">
      <table class="table-fixed pl-5" style="width: 100%">
        <thead>
        <tr class="text-left">
          <th style="width: 16.67%">Creator</th>
          <th style="width: 16.67%">Size</th>
          <th style="width: 16.67%">Date</th>
          <th style="width: 10%">Type</th>
          <th style="width: 40%">File Name</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="r in directory.children" :key="r.key"
            @dblclick="doubleClick(r, 0)"
            @contextmenu.prevent="contextmenuVisible = true"
        >
          <!--          <el-popover-->
          <!--              placement="top"-->
          <!--              :width="160"-->
          <!--              :visible="contextmenuVisible"-->
          <!--          >-->
          <!--            <div>-->
          <!--              <el-button @click="doubleClick(r)" class="contextmenu"> 打开</el-button>-->
          <!--              <el-button v-if="r.type !== 0" @click="doubleClick(r, 1)" class="contextmenu"> 编辑</el-button>-->
          <!--              <el-button @click="deleteFileOrDirectory(r)" class="contextmenu"> 删除</el-button>-->
          <!--            </div>-->
          <!--            <template #reference>-->
          <td> {{ r.creator }}</td>
          <td> {{ r.size }}</td>
          <td> {{ r.date }}</td>
          <td> {{ r.type === 0 ? 'DIR' : 'DOC' }}</td>
          <td> {{ r.label }}</td>
          <!--            </template>-->
          <!--          </el-popover>-->
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>

const DOC = 1
const DIRECTORY = 0

export default {
  name: "FileRecord",
  props: {
    directory: Object,
  },
  data() {
    return {
      contextmenuVisible: false,
    }
  },
  methods: {
    doubleClick(data, mode) {
      if (data.type === DOC) {
        if (mode === 1) {
          this.$emit("docEditRequest", data)
        } else {
          this.$emit("docReadRequest", data)
        }
      } else if (data.type === DIRECTORY) {
        this.$emit("dirChangeRequest", data.key)
      }
    },
    deleteFileOrDirectory(data) {
      if (data.type === DOC) {
        this.$emit("removeDocRequest", data.key)
      } else {
        this.$emit("removeDirRequest", data.key)
      }
    }
  }
}
</script>

<style scoped>

th {
  overflow: hidden;
  white-space: nowrap;
}

td {
  overflow: hidden;
  padding-right: 10px;
  text-overflow: clip;
}

tbody tr:hover {
  @apply bg-gray-500;
}

.half {
  max-height: 45%;
  min-height: 45%;
  height: 45%;
  width: 100%;
  overflow-y: scroll;
}

</style>