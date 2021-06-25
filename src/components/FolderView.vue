<template>
  <el-container class="frame">
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
    <el-main class="folder-view">
      <div class=" pt-4" v-if="view_mode">
        <File
            v-for="f in directory.children"
            :key="f.key"
            :data="f"
            @dir-change-request="dirChange"
            @doc-read-request="openDoc"
            @doc-edit-request="editDoc"
            @remove-doc-request="removeDoc"
            @remove-dir-request="removeDir"
        >
        </File>
      </div>
      <FileRecord
          v-if="!view_mode"
          :directory="directory"
          @dir-change-request="dirChange"
          @doc-read-request="openDoc"
          @doc-edit-request="editDoc"
          @remove-doc-request="removeDoc"
          @remove-dir-request="removeDir"
      >

      </FileRecord>

      <DiskView
          v-if="!view_mode"
          :disk="disk"
      >

      </DiskView>


<!--      New directory -->
      <el-dialog
          title="新建文件夹"
          v-model="showNewFolderWindow"
          width="30%"
      >
        <el-form label-position="left" label-width="80px" @submit.prevent="addNewDir();showNewFolderWindow = false; fileNameInput = ''">
          <el-form-item label="文件名">
            <el-input v-model="fileNameInput"  :placeholder="hint" ref="fileName"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="showNewFolderWindow = false; fileNameInput = ''" style="margin-right: 10px">取 消</el-button>
            <el-button type="primary" @click="addNewDir();showNewFolderWindow = false; fileNameInput = ''">确 定</el-button>
          </el-form-item>
        </el-form>
<!--        <template #footer>-->
<!--          <span class="dialog-footer">-->
<!--            <el-button @click="showNewFolderWindow = false; fileNameInput = ''" style="margin-right: 10px">取 消</el-button>-->
<!--            <el-button type="primary" @click="addNewDir();showNewFolderWindow = false; fileNameInput = ''">确 定</el-button>-->
<!--          </span>-->
<!--        </template>-->
      </el-dialog>

<!--      New document-->
      <el-dialog
          title="新建文本文件"
          v-model="showNewDocWindow"
          width="65%"
      >
        <el-form label-position="left" label-width="100px" @submit.prevent="addNewDoc();showNewDocWindow = false; fileNameInput = ''; docContent=''">
          <el-form-item label="文件名">
            <el-input v-model="fileNameInput"  :placeholder="hint" ref="fileName2"></el-input>
          </el-form-item>
          <el-form-item label="文件内容：">
            <el-input
                type="textarea"
                :autosize="{ minRows: 20, maxRows: 30 }"
                placeholder="请输入内容"
                v-model="docContent">
            </el-input>
          </el-form-item>
        </el-form>
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
          <div class="border-2 bg-blue-50 rounded-sm border-blue-500 p-2">
            <p class="font-medium font-sans overflow-clip">{{ viewDocContent }}</p>
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

<!--      Document editor -->
      <el-dialog
          :title="editDocName"
          v-model="showEditWindow"
          width="50%"
      >
        <div>
          <p> 编辑内容： </p>
          <el-input
              type="textarea"
              :autosize="{ minRows: 20, maxRows: 30}"
              placeholder="请输入内容"
              v-model="editDocContent">
          </el-input>
        </div>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="showEditWindow = false;">放弃修改</el-button>
            <el-button type="primary" @click="saveEditDoc(); showEditWindow = false;">保 存</el-button>
          </span>
        </template>
      </el-dialog>

<!--      At bottom: Button Group -->
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
                  @click="newFolderWindow"
              > 文件夹
              </el-button>

              <el-button
                  icon="el-icon-document-add" class="in-pop"
                  @click="newDocWindow"
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
          <el-button icon="el-icon-info" circle type="primary" class=" shadow-lg"
                     @click="viewChange"
          >
          </el-button>
        </div>
      </div>

    </el-main>
  </el-container>
</template>

<script>
import File from "@/components/File";
import Path from "@/components/Path";
import FileRecord from "@/components/FileRecord";
import DiskView from "@/components/DiskView";

import { ref } from 'vue'
import { ElMessage } from 'element-plus'

Array.prototype.remove = function(val) {
  let index = this.indexOf(val);
  if (index > -1) {
    this.splice(index, 1);
  }
};

export default {
  name: "FolderView",
  props: {
    directory: Object,
    disk: Object,
    view_mode: Boolean
  },
  data() {
    return {
      // addWindowVisible: false,
      showNewFolderWindow: false,
      showNewDocWindow: false,
      showDocViewerWindow: false,
      showEditWindow: false,

      hint: "不包含 . / ' \" $ { } 符号",
      fileNameInput: ref(''),
      docContent: ref(''),
      editDocContent: ref(''),

      viewDocContent: ref(''),
      viewDocName: ref(''),
      editDocName: ref(''),

      docBeingEdited: null,
    }
  },
  components: {
    DiskView,
    FileRecord,
    File,
    Path
  },
  methods: {
    newFolderWindow(){
      this.showNewFolderWindow = true
      setTimeout(() => {this.$refs.fileName.focus()}, 300)
    },

    newDocWindow() {
      this.showNewDocWindow = true
      setTimeout(() => {this.$refs.fileName2.focus()}, 300)
    },

    viewChange() {
      this.$emit("viewChange")
    },

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
      this.editDocContent = ''
      this.docContent = ''
    },

    checkInputFile(name) {
      if (name === '') return false
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
    },

    editDoc(data) {
      this.docBeingEdited = data.key
      this.editDocName = data.label
      this.editDocContent = data.content
      this.showEditWindow = true
    },

    findByKey(key, data) {
      if (data.key === key) {
        return data
      }
      console.log(data, 'in findByKey of FolderView')
      if (!data.children) return false
      for (var i = 0; i < data.children.length; i++) {
        let t = this.findByKey(key, data.children[i])
        if (t) return t
      }
    },

    findParentByKey(key, data) {
      if (key === data.key) {
        return 1
      }
      console.log(data, 'in findParentByKey of FolderView')
      if (!data.children) return false
      for (var i = 0; i < data.children.length; i++) {
        let t = this.findParentByKey(key, data.children[i])
        if (t) return data
      }
    },

    saveEditDoc() {
      let node = this.findByKey(this.docBeingEdited, this.directory)
      let size = this.calculateSize(this.editDocContent)
      this.setBlockOf(node, size)
      this.updateDirectorySize(node.key, node.size - size)
      node.size = size
      node.content = this.editDocContent
      this.editDocContent = ''
      this.docBeingEdited = null
      this.editDocName = ''
      ElMessage({
        showClose: true,
        message: '保存成功！',
        type: 'success'
      })

    },

    setBlockOf(doc, size) {
      let oldBlockCount = doc.blocks.length
      let newBlockCount = Math.ceil(size /  this.disk.block_size)
      if (newBlockCount === 0) newBlockCount = 1
      if (oldBlockCount === newBlockCount) {
        return undefined
      } else if (oldBlockCount > newBlockCount) {
        this.retrieveSpaceOf(doc, oldBlockCount - newBlockCount)
      } else {
        this.allocateBlocks(newBlockCount - oldBlockCount, doc)
      }
    },

    removeByKey(nodeData, key) {
      for (const nodeDatum of nodeData) {
        if (nodeDatum.key === key) {
          nodeData.remove(nodeDatum)
          return true
        }
      }
      return false;
    },

    removeDoc(key) {
      let par = this.findParentByKey(key, this.directory)
      let data = this.findByKey(key, this.directory)
      this.updateDirectorySize(data.key, -data.size)
      this.retrieveSpaceOf(data)
      if (this.removeByKey(par.children, key)) {
        ElMessage({
          showClose: true,
          message: '已删除' + key,
          type: 'success'
        })
      } else {
        ElMessage({
          showClose: true,
          message: '删除错误',
          type: 'error'
        })
      }
    },

    removeDir(key) {
      let par = this.findParentByKey(key, this.directory)
      let cur = this.findByKey(key, this.directory)
      this.removeAllDocUnderDir(cur)
      if (this.removeByKey(par.children, key)) {
        ElMessage({
          showClose: true,
          message: '已删除' + key,
          type: 'success'
        })
      } else {
        ElMessage({
          showClose: true,
          message: '删除错误',
          type: 'error'
        })
      }
    },

    removeAllDocUnderDir(dir) {
      for (const child of dir.children) {
        if (child.type !== 0) {
          this.retrieveSpaceOf(child)
          this.updateDirectorySize(child.key, -child.size)
        } else {
          this.removeAllDocUnderDir(child)
        }
      }
    },

    allocateBlocks(num, data) {
      console.log(num)
      if (num > this.disk.available) {
        ElMessage({
          showClose: true,
          message: '空间不足',
          type: 'error'
        })
        return false
      }
      for (let i = 0; i < num; i++) {
        console.log("free_pointer ", this.disk.free_pointer)
        data.blocks.push(this.disk.free_pointer)
        this.disk.bit_table[this.disk.free_pointer] = true
        this.disk.storage[this.disk.free_pointer] = data.key
        this.findNextDiskFreeSpace()
      }
      return true
    },

    findNextDiskFreeSpace() {
      for (let i = 0; i < this.disk.block_amount; i++) {
        if (this.disk.bit_table[this.disk.free_pointer] === false) {
          return
        }
        this.disk.free_pointer = (this.disk.free_pointer + 1) % this.disk.block_amount
      }
    },

    retrieveSpaceOf(data, amount) {
      if (!data.blocks) return
      if (!amount) {
        amount = data.blocks.length
      }
      for (let i = 0; i < amount; i++) {
        this.disk.bit_table[data.blocks.pop()] = false
        this.disk.available++
      }
    },

    calculateSize(text) {
      let count = text.length;
      for (let i = 0; i < text.length; i++) {
        if (text.charCodeAt(i) > 255) {
          count++;
        }
      }
      // console.log('in calculate: ', count, count /1024)
      return (count / 1024);
    },

    updateDirectorySize(path, change) {
      this.$emit('changeSizeOfPath', path, change)
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

.frame {
  max-height: 100%;
  min-height: 100%;
  height: 100%;
}

.folder-view {
  max-height: 95%;
  min-height: 95%;
  height: 95%;
}

</style>