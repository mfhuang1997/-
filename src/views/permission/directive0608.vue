<template>
  <div class="wrap">
    <!-- 录音笔 -->
    <MRecorder @handleStop="handelEndRecord" />
  </div>
</template>
<script>
// 引入recorder.js
import MRecorder from '@/components/MRecorder'
import recording from '@/utils/recorder.js'
export default {
  components: { MRecorder },
  data() {
    return {
      form: {
        time: '按住说话(60秒)',
        audioUrl: ''
      },
      num: 60, // 按住说话时间
      recorder: null,
      interval: '',
      audioFileList: [], // 上传语音列表
      startTime: '', // 语音开始时间
      endTime: '' // 语音结束
    }
  },
  methods: {

    // 录音处理结束事件
    handelEndRecord(param) {
      console.log(param)
      this.msource = param
    },

    // 清除定时器
    clearTimer() {
      if (this.interval) {
        this.num = 60
        clearInterval(this.interval)
      }
    },
    // 长按说话
    mouseStart() {
      this.clearTimer()
      this.startTime = new Date().getTime()
      recording.get((rec) => {
        // 当首次按下时，要获取浏览器的麦克风权限，所以这时要做一个判断处理
        if (rec) {
          // 首次按下，只调用一次
          if (this.flag) {
            this.mouseEnd()
            this.flag = false
          } else {
            this.recorder = rec
            this.interval = setInterval(() => {
              if (this.num <= 0) {
                this.recorder.stop()
                this.num = 60
                this.clearTimer()
              } else {
                this.num--
                this.time = '松开结束（' + this.num + '秒）'
                this.recorder.start()
              }
            }, 1000)
          }
        }
      })
    },
    // 松开时上传语音
    mouseEnd() {
      this.clearTimer()
      this.endTime = new Date().getTime()
      if (this.recorder) {
        this.recorder.stop()
        // 重置说话时间
        this.num = 60
        this.time = '按住说话（' + this.num + '秒）'
        // 获取语音二进制文件
        const bold = this.recorder.getBlob()
        // 将获取的二进制对象转为二进制文件流
        const files = new File([bold], 'test.mp3', { type: 'audio/mp3', lastModified: Date.now() })
        const fd = new FormData()
        fd.append('file', files)
        fd.append('tenantId', 3) // 额外参数，可根据选择填写
        // 这里是通过上传语音文件的接口，获取接口返回的路径作为语音路径
        this.uploadFile(fd)
      }
    }
  }
}
</script>
<style scoped>
</style>
