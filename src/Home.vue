<template>
  <div>
    <el-container>
      <el-header>
        <p class="text-red-300">
          haha
        </p>
      </el-header>
      <el-container>
        <el-aside>
          <EntireDirectory
              class="pl-4"
              @dir-changed="treeDirChanged"
              :data="fileData"
              ref="directory"
          ></EntireDirectory>
        </el-aside>
        <el-main>
          <FolderView
            :directory="currentDir"
            @create-directory-request="createDir"
            @create-document-request="createDoc"
            @dir-change-request="changeDir"
          >

          </FolderView>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import EntireDirectory from "@/components/EntireDirectory";
import FolderView from "@/components/FolderView";
const DOC = 1
const DIRECTORY = 0

export default {
  name: "Home",
  components: {
    EntireDirectory,
    FolderView
  },
  data() {
    return {
      fileData: [{
        key: 'root/',
        label: 'root',
        type: DIRECTORY,
        isLeaf: false,
        children: [
          {
            key: 'root/dic1',
            label: 'dic1',
            isLeaf: false,
            type: DIRECTORY,
            children: []
          },
          {
            key: 'root/file1',
            label: 'file1',
            isLeaf: true,
            type: DOC,
          }
        ],
      }],
      currentDir: undefined
    }
  },
  methods: {
    treeDirChanged(n) {
      if (n) {
        console.log('in treeDirChanged')
        console.log(n)
        this.currentDir = n
      }
    },
    changeDir(key) {
      console.log(key, "in changeDir of home")
      this.$refs.directory.changeDir(key)
    },
    createDir(name, key) {
      console.log(name, key, "in createDir of home")
      this.$refs.directory.addFile(name, DIRECTORY, key)
    },
    createDoc(name, key, content) {
      console.log(name, key, "in createDoc of home")
      this.$refs.directory.addFile(name, DOC, key, content)
    }
  },
  created() {
    this.currentDir = this.fileData[0]
  },
}
</script>

<style scoped>

.el-main {
  text-align: left;
  padding: 0;
}

.el-header {
  background: aquamarine;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  text-align: center;
  width: 30%;
}

</style>