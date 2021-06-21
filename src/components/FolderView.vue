<template>
  <el-container>
    <el-header>
      <div class="flex line-45 pt-1">
        <el-button
            icon="el-icon-arrow-left"
            circle type="primary"
            size="small"
            @click="goback"
            :disabled="(directory.key === 'root/')"
        ></el-button>
        <Path
            :full-path="directory.key"
            @dir-change-request="dirChange"
        ></Path>
      </div>
    </el-header>
    <el-main>
      <div class="grid-flow-row grid-cols-3 pt-4">
        <File
            v-for="f in directory.children"
            :key="f.key"
            :data="f"
            @dir-change-request="dirChange"
            @doc-read-request="openDoc"
            @doc-edit-request="editDoc"
        >
        </File>
      </div>

<!--      New directory -->
      <el-dialog
          title="新建文件夹"
          v-model="showNewFolderWindow"
          width="30%"
      >
        <div>
          文件名：
          <el-input v-model="fileNameInput" :placeholder="hint"></el-input>
        </div>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="showNewFolderWindow = false; fileNameInput = ''" style="margin-right: 10px">取 消</el-button>
            <el-button type="primary" @click="addNewDir();showNewFolderWindow = false; fileNameInput = ''">确 定</el-button>
          </span>
        </template>
      </el-dialog>

<!--      New document-->
      <el-dialog
          title="新建文本文件"
          v-model="showNewDocWindow"
          width="65%"
      >
        <div>
          <p> 文件名： </p>
          <el-input v-model="fileNameInput" :placeholder="hint"></el-input>
          <p> 文件内容： </p>
          <el-input
              type="textarea"
              :autosize="{ minRows: 20, maxRows: 30}"
              placeholder="请输入内容"
              v-model="docContent">
          </el-input>
        </div>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="showNewDocWindow = false; fileNameInput = ''" style="margin-right: 10px">取 消</el-button>
            <el-button type="primary" @click="addNewDoc(); showNewDocWindow = false; fileNameInput = ''">确 定</el-button>
          </span>
        </template>
      </el-dialog>

<!--      Document viewer -->
      <el-dialog
          :title="viewDocName"
          v-model="showDocViewerWindow"
          width="50%"
      >
        <div>
          <p> 文件内容： </p>
          <div class="border-2 bg-blue-50 rounded-sm border-blue-500 block p-2">
            <p class="font-medium font-sans">
              {{ viewDocContent }}
            </p>
            <p v-if="!viewDocContent" class="text-gray-300">
              该文档没有内容
            </p>
          </div>
        </div>
        <template #footer>
          <span class="dialog-footer">
            <el-button type="primary" @click="showDocViewerWindow = false;">关 闭</el-button>
          </span>
        </template>
      </el-dialog>

<!--      At bottom -->
      <div class="button-group">
        <div>
          <el-popover
              placement="left"
              title="选择类型"

              width="150px"
          >
            <div>
              <el-button
                  icon="el-icon-folder-add" class="in-pop"
                  @click="showNewFolderWindow = true"
              > 文件夹
              </el-button>

              <el-button
                  icon="el-icon-document-add" class="in-pop"
                  @click="showNewDocWindow = true"
              >
                文档
              </el-button>
            </div>
            <template #reference>
              <el-button
                  icon="el-icon-plus" circle type="success" class=" shadow-lg"
                  @click="addWindowVisible = true"
              ></el-button>
            </template>
          </el-popover>
        </div>
        <div class="h-5"></div>
        <div>
          <el-button icon="el-icon-info" circle type="primary" class=" shadow-lg"></el-button>
        </div>
      </div>

    </el-main>
  </el-container>
</template>

<script>
import File from "@/components/File";
import Path from "@/components/Path";
import { ref } from 'vue'
import { ElMessage } from 'element-plus'

export default {
  name: "FolderView",
  props: {
    directory: Object
  },
  data() {
    return {
      // addWindowVisible: false,
      showNewFolderWindow: false,
      showNewDocWindow: false,
      showDocViewerWindow: false,

      hint: "不包含 . / ' \" $ { } 符号",
      fileNameInput: ref(''),
      docContent: ref(''),

      viewDocContent: ref(''),
      viewDocName: ref('')
    }
  },
  components: {
    File,
    Path
  },
  methods: {
    dirChange(dirKey) {
      console.log(dirKey, 'in dirChange')
      this.$emit("dirChangeRequest", dirKey)
    },
    addDoc(docName, content) {
      console.log(docName, content)
    },
    addDir(dirName) {
      console.log(dirName)
    },
    goback() {
      let path = this.directory.key.substring(0, this.directory.key.length - 1)
      path = path.substring(0, path.lastIndexOf('/') + 1)
      this.$emit("dirChangeRequest", path)
    },

    addNewDir() {
      console.log("Adding file ", this.fileNameInput, " in dir ", this.directory.key)
      if (this.checkInputFile(this.fileNameInput)) {
        this.$emit('createDirectoryRequest', this.fileNameInput, this.directory.key)
      } else {
        ElMessage({
          showClose: true,
          message: '含有非法字符，建立失败',
          type: 'error'
        })
      }
    },

    addNewDoc() {
      console.log("Adding file ", this.fileNameInput, " in dir ", this.directory.key)
      if (this.checkInputFile(this.fileNameInput)) {
        this.$emit('createDocumentRequest', this.fileNameInput, this.directory.key, this.docContent)
      } else {
        ElMessage({
          showClose: true,
          message: '含有非法字符，建立失败',
          type: 'error'
        })
      }
    },

    checkInputFile(name) {
      let notAllowed = ['/', '.', "'", '"', '$', '{', '}']
      for (const char of notAllowed) {
        if (name.indexOf(char) > 0) {
          return false
        }
      }
      return true
    },

    openDoc(data) {
      this.viewDocName = data.label
      this.viewDocContent = data.content
      this.showDocViewerWindow = true
    }

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
}

.el-container {
  padding-top: 0;
}

.el-main {
  padding: 0;
}

.el-header {
  @apply bg-gray-100;
}

.line-45 {
  line-height: 45px;
}

.button-group {
  @apply flex-col;
  position: fixed;
  right: 3%;
  bottom: 5%
}

.el-popover {
  text-align: left;
  justify-content: left;
  margin-inside: none;
  margin: 0;
}

.el-button {
  margin: 0;
}

.in-pop {
  width: 120px;
  text-align: left;
}

</style>