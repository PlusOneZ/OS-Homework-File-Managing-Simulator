<template>
  <div class="grid-flow-row grid-cols-3">
    <el-container>
      <el-header>
        <el-button icon="el-icon-arrow-left" circle type="primary"> </el-button>

      </el-header>
      <el-main style="padding: 0">
        <File
            v-for="f in directory.children"
            :key="f.key"
            :data="f"
        >
        </File>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import File from "@/components/File";

export default {
  name: "FolderView",
  props: {
    directory: Object
  },
  components: {
    File
  },
  mounted() {

  }
}
</script>

<style scoped>
.el-header {
  padding-top: 0;
  max-height: 45px;
  line-height: 45px;
  background: aquamarine;
}
.el-container {
  padding-top: 0;
}
.el-main {
  padding: 0;
}
</style>