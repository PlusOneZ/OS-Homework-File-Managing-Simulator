<template>
  <div class="home">
    <el-container class="frame">
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
              :user="user_name"
          ></EntireDirectory>
        </el-aside>
        <el-main>
          <FolderView
            :directory="currentDir"
            :disk="disk"
            :view_mode="viewMode"
            @create-directory-request="createDir"
            @create-document-request="createDoc"
            @dir-change-request="changeDir"
            @view-change="changeView"
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

var day = (new Date()).format("yyyy/MM/dd hh:mm:ss")

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
        children: [ ],
        creator: 'system',
        size: 0,
        date: day
      }],
      currentDir: undefined,
      disk: {
        block_size: 512, // kb as unit
        block_amount: 512,
        bit_table: [ ],
        storage: [ ],
        available: null
      },
      user_name: 'root',
      viewMode: true
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
    },
    changeView() {
      this.viewMode = !this.viewMode
    }
  },
  created() {
    this.currentDir = this.fileData[0]
    for (let i = 0; i < this.disk.block_amount; i++) {
      this.disk.bit_table[i] = false
      this.disk.storage[i] = null
    }
    this.disk.available = this.disk.block_amount
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

.home {
  max-height: 100%;
  min-height: 100%;
  height: 100%;
}

.frame {
  max-height: 100%;
  min-height: 100%;
  height: 100%;
}

</style>