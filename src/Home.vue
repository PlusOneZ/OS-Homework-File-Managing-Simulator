<template>
  <div class="home">
    <el-container class="frame">
      <el-header>

        <div class="left-top-button">
          <el-button @click="basicInfoVisible = true" :circle="true" icon="el-icon-user" type="success"></el-button>
        </div>

        <p class="text-xl bold hd">
          在线文件管理模拟器
        </p>

        <div class="button-right">
          <el-button @click="exportFile" :circle="true" icon="el-icon-download" type="primary"></el-button>
        </div>

        <el-upload ref="upload" :limit="1" :on-change="changeUploadFile" :file-list="undefined"
                   :auto-upload="false" :multiple="false" action="#" accept=".json">
          <div class="button-left">
            <el-button :circle="true" icon="el-icon-upload2" type="primary"></el-button>
          </div>
        </el-upload>

      </el-header>
      <el-container class="frame">
        <el-aside>
          <EntireDirectory
              class="pl-4"
              @dir-changed="treeDirChanged"
              @change-size-of-path="updateSize"
              :data="fileData"
              :current="currentDir.key"
              :disk="disk"
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
              @change-size-of-path="updateSize"
          >

          </FolderView>


          <el-dialog
              title="基本信息"
              v-model="basicInfoVisible"
              width="40%"
          >
            <el-form label-position="left" label-width="100px" @submit.prevent="basicInfo(); basicInfoVisible = false">
              <el-form-item label="用户名">
                <el-input v-model="user_name" ref="userName"></el-input>
              </el-form-item>
              <el-form-item label="磁盘块数量">
                <el-input v-model="block_amount">
                </el-input>
              </el-form-item>
              <el-form-item label="磁盘块大小(kb)">
                <el-input v-model="block_size">
                </el-input>
              </el-form-item>
            </el-form>
            <template #footer>
          <span class="dialog-footer">
            <el-button @click="basicInfoVisible = false" style="margin-right: 10px">取 消</el-button>
            <el-button type="primary" @click="basicInfo(); basicInfoVisible = false">确定并重置</el-button>
          </span>
            </template>
          </el-dialog>

        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import EntireDirectory from "@/components/EntireDirectory";
import FolderView from "@/components/FolderView";
import {ElMessage} from "element-plus";
import {ref} from "vue";

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

let RootOnly = {
  key: 'root/',
  label: 'root',
  type: DIRECTORY,
  isLeaf: false,
  children: [],
  creator: 'system',
  size: 0,
  date: day
}

export default {
  name: "Home",
  components: {
    EntireDirectory,
    FolderView
  },
  data() {
    return {
      fileData: [RootOnly],
      currentDir: undefined,
      disk: {
        block_size: 4, // kb as unit
        block_amount: 512,
        bit_table: [],
        storage: [],
        available: null,
        free_pointer: 0
      },
      user_name: ref('root'),
      viewMode: true,
      basicInfoVisible: true,
      block_amount: ref('512'),
      block_size: ref('4')
    }
  },
  methods: {

    basicInfo() {
      console.log(Number(this.block_amount))
      if (Number(this.block_amount)) {
        this.disk.block_amount = Number(this.block_amount)
        console.log("haha", this.disk.block_amount)
      }
      this.block_amount = String(this.disk.block_amount)
      if (Number(this.block_size)) {
        this.disk.block_size = Number(this.block_size)
      }
      this.block_size = String(this.disk.block_size)
      this.resetDisk()
    },

    updateSize(path, change) {
      console.log("updateSize: path: ", path)
      this.startUpdate(this.fileData[0], path, change)
    },

    startUpdate(data, key, change) {
      if (data.key === key) {
        return data
      }
      console.log(data, 'in changeSize')
      if (!data.children) return false
      for (var i = 0; i < data.children.length; i++) {
        let t = this.startUpdate(data.children[i], key, change)
        if (t) {
          data.size += change
          return t
        }
      }
    },

    changeUploadFile(file) {
      if (file.status === 'ready') {
        this.handleUpload(file)
      } else if (file.status === 'fail') {
        ElMessage({
          showClose: true,
          message: '上传出错',
          type: 'error'
        })
      }

    },

    handleUpload(file) {
      let reader = new FileReader()
      reader.readAsText(file.raw);
      let _this = this
      reader.onload = function (evt) {
        // console.log(evt.target.result)
        let data = JSON.parse(evt.target.result)

        if (data) {
          console.log("parse OK")
          _this.fileData = data.fileData
          _this.disk = data.disk
          setTimeout(() => {
            _this.$refs.directory.changeDir('root/')
          }, 500)
        }
      }
    },

    exportFile() {
      let data = new Blob([JSON.stringify({
        'fileData': this.fileData,
        'disk': this.disk
      })], {type: 'application/json'})
      let a = document.createElement('a');
      a.download = "ArchivedFiles"
      a.href = URL.createObjectURL(data)
      a.style.visibility = 'none'
      a.click()
      ElMessage({
        showClose: true,
        message: '成功导出，请查看下载文件夹',
        type: 'success'
      })
    },

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
    },
    resetDisk() {
      this.disk.bit_table = []
      this.disk.storage = []
      this.fileData = [RootOnly]
      for (let i = 0; i < this.disk.block_amount; i++) {
        this.disk.bit_table[i] = false
        this.disk.storage[i] = null
      }
      this.disk.available = this.disk.block_amount
      setTimeout(() => {this.changeDir('root/')}, 500)
    }
  },

  created() {
    this.currentDir = this.fileData[0]
    this.resetDisk()
  },
}
</script>

<style scoped>

.el-main {
  text-align: left;
  padding: 0;
}

.el-header {
  background: #baccd9;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  text-align: center;
  max-height: 90%;
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

.hd {
  line-height: 60px;
}

.button-left {
  line-height: normal;
  position: fixed;
  right: 75px !important;
  top: 8px !important;
}

.button-right {
  position: fixed;
  top: 8px;
  right: 10px;
  line-height: normal;
}

.left-top-button {
  position: fixed;
  top: 8px;
  left: 15px;
  line-height: normal;
}
</style>