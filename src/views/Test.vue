<template>
  <div>
    vue2.7
    <p>{{ message }}</p>
    <button
      @click="downLoad"
      class="py-2 px-4 font-semibold rounded-lg shadow-md text-white bg-green-500 hover:bg-green-700 border-none cursor-pointer"
    >
      download
    </button>
    <button
      @click="cancelCtr"
      class="ml-3 py-2 px-4 font-semibold rounded-lg shadow-md text-white bg-green-500 hover:bg-green-700 border-none cursor-pointer"
    >
      cancel
    </button>
    <p>共 {{ bytesToSize(file.size) }}</p>
    <p>下载进度 {{ bytesToSize(file.downloadSize) }}</p>
    <div class="w-[50%]">
      <el-progress
        class="el-bg-inner-done"
        :text-inside="true"
        :stroke-width="24"
        :percentage="downloadProgress"
      ></el-progress>
    </div>
    <div class="w-[126px] mt-10">
      <el-progress
        type="circle"
        status="success"
        :percentage="downloadProgress"
      ></el-progress>
    </div>
    <div class="w-[160px] mt-10">
      <el-progress
        type="dashboard"
        :stroke-width="14"
        :percentage="downloadProgress"
        :width="160"
      >
      </el-progress>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { Message as ElMessage } from 'element-ui'

const message = 'Hello Vue!'
const customColors = [
  { color: '#f56c6c', percentage: 20 },
  { color: '#e6a23c', percentage: 40 },
  { color: '#5cb87a', percentage: 60 },
  { color: '#1989fa', percentage: 80 },
  { color: '#6f7ad3', percentage: 100 },
]
// 下载进度
const downloadProgress = ref(0)
const file = ref({
  size: 0,
  downloadSize: 0,
})
let cancel

//下载
const downLoad = function () {
  downloadProgress.value = 0
  let uniSign = Date.now() //可能会连续点击下载多个文件，这里用时间戳来区分每一次下载的文件
  //通过传递一个 executor 函数到 CancelToken 的构造函数来创建 cancel token：
  const CancelToken = axios.CancelToken

  axios({
    url: 'https://suqiqi.oss-cn-beijing.aliyuncs.com/test/video/1.mp4',
    responseType: 'blob',

    headers: { 'Content-Type': 'application/json; charset=utf-8' },
    //取消axios的函数
    cancelToken: new CancelToken(function executor(c) {
      // executor 函数接收一个 cancel 函数作为参数
      cancel = c
    }),

    onDownloadProgress: function (progress) {
      console.log(progress)
      const { loaded, total } = progress
      file.value.size = total
      if (progress.lengthComputable) {
        console.log(loaded, total)
        file.value.downloadSize = loaded
        downloadProgress.value = +((loaded / total) * 100.0).toFixed(1) //实时获取最新下载进度
      }
      // 读取进度并且覆盖
      // const percent = Math.floor(progress.loaded / progress.total * 100)
    },
  })
    .then((res) => {
      console.log(res)
      let link = document.createElement('a')
      link.href = URL.createObjectURL(res.data)
      link.download = '下载.mp4'
      document.body.appendChild(link)
      link.click()
      URL.revokeObjectURL(link.href)
    })
    .catch((e) => {
      if (JSON.stringify(e) == '{}') {
        ElMessage.error('Download Cancelled:' + downloadProgress.value)
      } else {
        ElMessage.error('Download Failed:' + downloadProgress.value)
      }
    })
}
// byte单位换算
const bytesToSize = function (bytes) {
  if (bytes === 0) return '0 B'
  let k = 1024, // or 1024
    sizes = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
    i = Math.floor(Math.log(bytes) / Math.log(k))
  return (bytes / Math.pow(k, i)).toPrecision(3) + ' ' + sizes[i]
}
// bytes转换为MB
const bytesToMB = function (bytes) {
  return (bytes / 1024 / 1024).toFixed(2)
}
// MB单位换算
const MBToSize = function (MB) {
  if (MB === 0) return '0 MB'
  let k = 1024, // or 1024
    sizes = ['MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
    i = Math.floor(Math.log(MB) / Math.log(k))
  return (MB / Math.pow(k, i)).toPrecision(3) + ' ' + sizes[i]
}
// 取消下载
const cancelCtr = function () {
  cancel()
}
</script>

<style scoped>
.el-progress {
  width: 100%;
}
.el-progress__text {
  color: #fff;
  font-size: 14px;
}
.el-progress-bar__outer {
  height: 12px !important;
  border: 1px solid #78335f;
  background-color: transparent;
}
/* 渐变进度条 */
:deep(.el-bg-inner-running .el-progress-bar__inner) {
  background-color: unset;
  background-image: linear-gradient(to right, #3587d8, #6855ff);
}
:deep(.el-bg-inner-error .el-progress-bar__inner) {
  background-image: linear-gradient(to right, #3587d8, #fb3a7e);
}
:deep(.el-bg-inner-done .el-progress-bar__inner) {
  background-image: linear-gradient(to right, #3587d8, #53ff54);
}
</style>
