<template>
  <div class="upload-container">
    <el-button :style="{background:color,borderColor:color}" @click=" dialogVisible=true" icon="el-icon-upload" size="mini"
               type="primary">上传图片
    </el-button>
    <el-dialog :visible.sync="dialogVisible">
      <el-upload
        :action="fileUrl"
        :before-upload="beforeUpload"
        :file-list="fileList"
        :headers="uploadHeaders"
        :multiple="true"
        :on-remove="handleRemove"
        :on-success="handleSuccess"
        :show-file-list="true"
        class="editor-slide-upload"
        list-type="picture-card">
        <el-button size="small" type="primary">点击上传</el-button>
      </el-upload>
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button @click="handleSubmit" type="primary">确 定</el-button>
    </el-dialog>
  </div>
</template>

<script>
  import { getApiUrl } from '@/utils/utils'
  import { getToken } from '@/utils/auth'
  export default {
    name: 'EditorSlideUpload',
    props: {
      color: {
        type: String,
        default: '#1890ff'
      }
    },
    data() {
      return {
        dialogVisible: false,
        listObj: {},
        fileList: [],
        fileUrl: '',
        uploadHeaders: {
          'Authorization': ''
        },
      }
    },
    created() {
      this.fileUrl = getApiUrl() + '/file'
      this.uploadHeaders['Authorization'] = getToken()
    },
    methods: {
      checkAllSuccess() {
        return Object.keys(this.listObj).every(item => this.listObj[item].hasSuccess)
      },
      handleSubmit() {
        const arr = Object.keys(this.listObj).map(v => this.listObj[v])
        if (!this.checkAllSuccess()) {
          this.$message('请等待所有图片上传成功 或 出现了网络问题，请刷新页面重新上传！')
          return
        }
        this.$emit('successCBK', arr)
        this.listObj = {}
        this.fileList = []
        this.dialogVisible = false
      },
      handleSuccess(response, file) {
        const uid = file.uid
        const objKeyArr = Object.keys(this.listObj)
        for (let i = 0, len = objKeyArr.length; i < len; i++) {
          if (this.listObj[objKeyArr[i]].uid === uid) {
            this.listObj[objKeyArr[i]].url = this.fileUrl + '/download?fileName=' + response.data.realFileName
            this.listObj[objKeyArr[i]].hasSuccess = true
            return
          }
        }
      },
      handleRemove(file) {
        const uid = file.uid
        const objKeyArr = Object.keys(this.listObj)
        for (let i = 0, len = objKeyArr.length; i < len; i++) {
          if (this.listObj[objKeyArr[i]].uid === uid) {
            delete this.listObj[objKeyArr[i]]
            return
          }
        }
      },
      beforeUpload(file) {
        const _self = this
        const _URL = window.URL || window.webkitURL
        const fileName = file.uid
        this.listObj[fileName] = {}
        return new Promise((resolve, reject) => {
          const img = new Image()
          img.src = _URL.createObjectURL(file)
          img.onload = function() {
            _self.listObj[fileName] = { hasSuccess: false, uid: file.uid, width: this.width, height: this.height }
          }
          resolve(true)
        })
      }
    }
  }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  .editor-slide-upload {
    margin-bottom: 20px;

    /deep/ .el-upload--picture-card {
      width: 100%;
    }
  }
</style>
