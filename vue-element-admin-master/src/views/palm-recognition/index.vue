<template>
  <el-card class="box-card">
    <div slot="header" class="clearfix">
      <span>掌纹识别软件</span>
    </div>

    <!-- 标准图片上传 -->
    <el-divider>上传标准图片组</el-divider>
    <el-upload
      class="upload-demo"
      :before-upload="handlePreview('standard')"
      :file-list="standardFileList"
      accept="image/*"
      multiple
    >
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">点击或拖拽上传标准图片</div>
    </el-upload>
    <div v-if="standardPreviewUrls.length > 0" class="preview-container">
      <div
        v-for="(url, index) in standardPreviewUrls"
        :key="'standard-' + index"
        class="preview-item"
      >
        <el-image :src="url" fit="cover" style="width: 100px; height: 100px; margin: 5px;"></el-image>
        <el-button
          class="delete-btn"
          type="danger"
          icon="el-icon-delete"
          size="mini"
          @click="removeFile(index, 'standard')"
        ></el-button>
      </div>
    </div>

    <!-- 对比图片上传 -->
    <el-divider>上传对比图片组</el-divider>
    <el-upload
      class="upload-demo"
      :before-upload="handlePreview('comparison')"
      :file-list="comparisonFileList"
      accept="image/*"
      multiple
    >
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">点击或拖拽上传对比图片</div>
    </el-upload>
    <div v-if="comparisonPreviewUrls.length > 0" class="preview-container">
      <div
        v-for="(url, index) in comparisonPreviewUrls"
        :key="'comparison-' + index"
        class="preview-item"
      >
        <el-image :src="url" fit="cover" style="width: 100px; height: 100px; margin: 5px;"></el-image>
        <el-button
          class="delete-btn"
          type="danger"
          icon="el-icon-delete"
          size="mini"
          @click="removeFile(index, 'comparison')"
        ></el-button>
      </div>
    </div>

    <!-- 提交按钮 -->
    <el-button
      type="primary"
      style="margin-top: 20px;"
      :loading="isProcessing"
      @click="comparePalmprints"
    >
      提交并识别
    </el-button>

    <!-- 识别结果 -->
    <div v-if="result" class="result-container">
      <el-divider>识别结果</el-divider>
      <p>{{ result }}</p>
    </div>
  </el-card>
</template>

<script>
export default {
  data() {
    return {
      standardFileList: [],
      comparisonFileList: [],
      standardPreviewUrls: [],
      comparisonPreviewUrls: [],
      isProcessing: false,
      result: null
    }
  },
  beforeDestroy() {
    this.standardPreviewUrls.forEach((url) => URL.revokeObjectURL(url))
    this.comparisonPreviewUrls.forEach((url) => URL.revokeObjectURL(url))
  },
  methods: {
    // 图片预览处理
    handlePreview(type) {
      return (file) => {
        const previewUrl = URL.createObjectURL(file)
        if (type === 'standard') {
          this.standardPreviewUrls.push(previewUrl)
          this.standardFileList.push(file)
        } else if (type === 'comparison') {
          this.comparisonPreviewUrls.push(previewUrl)
          this.comparisonFileList.push(file)
        }
        return false// 禁止自动上传
      }
    },
    // 删除文件
    removeFile(index, type) {
      if (type === 'standard') {
        this.standardFileList.splice(index, 1)
        URL.revokeObjectURL(this.standardPreviewUrls[index])
        this.standardPreviewUrls.splice(index, 1)
      } else if (type === 'comparison') {
        this.comparisonFileList.splice(index, 1)
        URL.revokeObjectURL(this.comparisonPreviewUrls[index])
        this.comparisonPreviewUrls.splice(index, 1)
      }
    },
    // 调用大模型进行掌纹识别
    async comparePalmprints() {
      if (this.standardFileList.length === 0 || this.comparisonFileList.length === 0) {
        this.$message.error('请确保上传了标准图片组和对比图片组')
        return
      }

      this.isProcessing = true
      const formData = new FormData()
      this.standardFileList.forEach((file) => formData.append('standard_images', file))
      this.comparisonFileList.forEach((file) => formData.append('comparison_images', file))

      try {
        const response = await this.$axios.post('/api/palmprint/compare', formData, {
          headers: { 'Content-Type': 'multipart/form-data' }
        })
        this.result = response.data.result// 假设后端返回的字段为 result
        this.$message.success('识别完成！')
      } catch (error) {
        this.$message.error('识别失败，请重试')
        console.error(error)
      } finally {
        this.isProcessing = false
      }
    }
  }
}
</script>

<style scoped>
.preview-container {
  display: flex;
  flex-wrap: wrap;
}
.preview-item {
  position: relative;
}
.delete-btn {
  position: absolute;
  top: 5px;
  right: 5px;
}
.result-container {
  margin-top: 20px;
}
</style>
