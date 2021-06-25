<template>
  <div class="inline-flex">
    <el-popover
        placement="right"
        :width="160"
        :visible="contextmenuVisible"
    >
      <div>
        <el-button @click="doubleClick" class="contextmenu"> 打开 </el-button>
        <el-button v-if="data.type !== 0" @click="doubleClick(1)" class="contextmenu"> 编辑 </el-button>
        <el-button @click="deleteFileOrDirectory" class="contextmenu"> 删除 </el-button>
      </div>
      <template #reference>
        <el-button
            class="hover:shadow-lg"
            @dblclick="doubleClick"

            @blur="contextmenuVisible = false"
            @contextmenu.prevent="contextmenuVisible = true"
        >
          <div class="inline-flex h-20 w-20 focus:overflow-ellipsis">
            <svg
                v-if="data.type === 0"
                class="icon" viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg"
                width="100" height="80">
              <path
                  d="M853.333333 256H469.333333l-85.333333-85.333333H170.666667c-46.933333 0-85.333333 38.4-85.333334 85.333333v170.666667h853.333334v-85.333334c0-46.933333-38.4-85.333333-85.333334-85.333333z"
                  fill="#FFA000"></path>
              <path
                  d="M853.333333 256H170.666667c-46.933333 0-85.333333 38.4-85.333334 85.333333v426.666667c0 46.933333 38.4 85.333333 85.333334 85.333333h682.666666c46.933333 0 85.333333-38.4 85.333334-85.333333V341.333333c0-46.933333-38.4-85.333333-85.333334-85.333333z"
                  fill="#FFCA28"></path>
            </svg>

            <svg
                v-if="data.type === 1"
                class="icon" viewBox="0 0 1024 1024"
                xmlns="http://www.w3.org/2000/svg" width="100" height="80"
            >
              <path d="M853.333333 960H170.666667V64h469.333333l213.333333 213.333333z" fill="#90CAF9"></path>
              <path d="M821.333333 298.666667H618.666667V96z" fill="#E1F5FE"></path>
              <path
                  d="M341.333333 448h362.666667v42.666667H341.333333zM341.333333 533.333333h277.333334v42.666667H341.333333zM341.333333 618.666667h362.666667v42.666666H341.333333zM341.333333 704h277.333334v42.666667H341.333333z"
                  fill="#1976D2"></path>
            </svg>
          </div>

          <div class="break-all truncate w-24 overflow-hidden">
            {{ data.label }}
          </div>
        </el-button>
      </template>
    </el-popover>
  </div>
</template>

<script>

const DOC = 1
const DIRECTORY = 0

export default {
  name: "File",
  props: {
    data: Object,
  },
  data() {
    return {
      contextmenuVisible: false,
    }
  },
  methods: {
    doubleClick(mode) {
      if (this.data.type === DOC) {
        if (mode === 1) {
          this.$emit("docEditRequest", this.data)
        } else {
          this.$emit("docReadRequest", this.data)
        }
      } else if (this.data.type === DIRECTORY) {
        this.$emit("dirChangeRequest", this.data.key)
      }
    },
    deleteFileOrDirectory() {
      if (this.data.type === DOC) {
        this.$emit("removeDocRequest", this.data.key)
      } else {
        this.$emit("removeDirRequest", this.data.key)
      }
    }
  }
}
</script>

<style scoped>
.el-button {
  margin: 0;
}

.contextmenu {
  width: 125px;
}

.el-popover {
  max-width: 120px;
}


</style>