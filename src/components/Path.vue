<template>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item v-for="(dir, count) in pathList" :key="count" @click="goto(count)">
        <a href="#">
          {{ dir }}
        </a>
      </el-breadcrumb-item>
    </el-breadcrumb>
</template>

<script>
export default {
  name: "Path",
  props: {
    fullPath: String,
  },
  computed: {
    pathList() {
      let list = this.fullPath.substring(0, this.fullPath.length - 1).split('/')
      return list
    }
  },
  methods: {
    goto(i) {
      var key = ""
      for (let j = 0; j <= i; j++) {
        key += this.pathList[j] + '/'
      }
      // console.log(key)
      this.$emit("dirChangeRequest", key)
    }
  }
}
</script>

<style scoped>
.el-breadcrumb {
  padding-top: 5px;
  line-height: 45px;
  padding-left: 2rem;
  @apply text-lg;
}
</style>