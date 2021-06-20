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
          ></EntireDirectory>
        </el-aside>
        <el-main>
          <FolderView
            :directory="currentDir"
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
      currentDirectory: 'root/',
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
      console.log(n.key)
      if (n.key){
        this.currentDirectory = n.key
      }
    }
  },
  created() {
    this.currentDir = this.fileData[0]
  }
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