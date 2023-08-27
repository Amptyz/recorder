<template>
  <div class="BaseRecorder">
    <div class="BaseRecorder-record">
      <el-button @click="startRecorder()" :disabled="recorder_state !== '无'">开始录音</el-button>
      <el-button @click="stopRecorder()" :disabled="recorder_state === '无'">结束录音</el-button>
    </div>

    <div class="BaseRecorder-play">
      <el-button @click="playRecorder()" :disabled="play_state !== '无'">录音播放</el-button>
      <el-button @click="stopPlayRecorder()" :disabled="play_state === '无'"
      >停止录音播放</el-button
      >
    </div>

    <div class="BaseRecorder-destroy">
      <el-button type="error" @click="destroyRecorder()">销毁录音</el-button>
    </div>
    <div class="BaseRecorder-wave">
      <p class="recorder_state">{{ recorder_state }}</p>
      <canvas ref="record"></canvas>
      <p class="recorder_state">{{ play_state }}</p>
      <canvas ref="play"></canvas>
    </div>
  </div>
</template>

<script>
import Recorder from 'js-audio-recorder';


export default {

  data() {
    return {
      recorder: null,
      // 波浪图-录音
      drawRecordId: null,
      // 波浪图-播放
      drawPlayId: null,
      // 录音状态
      recorder_state: '无',
      // 播放状态
      play_state: '无',
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    // 初始化
    init() {
      this.recorder = new Recorder({
        // 采样位数，支持 8 或 16，默认是16
        sampleBits: 16,
        // 采样率，支持 11025、16000、22050、24000、44100、48000，根据浏览器默认值
        sampleRate: 16000,
        // 声道，支持 1 或 2， 默认是1
        numChannels: 1,
        // 是否边录边转换，默认是false
        compiling: false,
      });
    },
    // 开始录音
    startRecorder() {
      this.recorder.start().then(
          () => {
            this.drawRecord();

            this.recorder_state = '录音中...';
          },
          error => {
            // 出错了
            console.log(`${error.name} : ${error.message}`);
          },
      );
    },

    // 结束录音
    stopRecorder() {
      this.recorder.stop();
      this.drawRecordId && cancelAnimationFrame(this.drawRecordId);
      this.drawRecordId = null;
      this.recorder_state = '无';
    },
    // 录音播放
    playRecorder() {
      this.recorder.play();
      this.drawPlay(); // 绘制波浪图
      this.play_state = '录音播放中';
    },

    // 停止录音播放
    stopPlayRecorder() {
      this.recorder.stopPlay();
      this.play_state = '无';
    },
    // 销毁录音
    destroyRecorder() {
      this.recorder.destroy().then(() => {
        console.log(this.drawRecordId, this.drawPlayId);
        this.drawRecordId && cancelAnimationFrame(this.drawRecordId);
        this.drawRecordId = null;

        this.drawPlayId && cancelAnimationFrame(this.drawPlayId);
        this.drawPlayId = null;

        this.init();
      });
    },


    //上传
    upload(){
      var pcmBlob = this.recorder.getPCMBlob();
      // 创建一个formData对象
      var formData = new FormData()
      // 此处获取到blob对象后需要设置fileName满足当前项目上传需求，其它项目可直接传把blob作为file塞入formData
      const newbolb = new Blob([wavBlob], { type: 'audio/pcm' })
      //获取当时时间戳作为文件名
      const fileOfBlob = new File([newbolb], new Date().getTime() + '.pcm')
      formData.append('file', fileOfBlob)
      uploadWavData(formData).then((response) => {
        console.log(response);
      });
    },
    /**
     * 绘制波浪图-录音
     * */
    drawRecord() {
      this.drawRecordId = requestAnimationFrame(this.drawRecord);

      let da=this.recorder.getRecordAnalyseData();
      const v = da[0] / 128;
      const y = (v * this.$refs.record.height) / 2;
      console.log(y);//这个y可代表声音大小

      this.drawWave({
        canvas: this.$refs.record,
        dataArray: this.recorder.getRecordAnalyseData(),
        bgcolor: 'white',
        lineWidth: 1,
        lineColor: 'red',
      });
    },
    /**
     * 绘制波浪图-播放
     * */
    drawPlay() {
      this.drawPlayId = requestAnimationFrame(this.drawPlay);
      this.drawWave({
        canvas: this.$refs.play,
        dataArray: this.recorder.getPlayAnalyseData(),
      });
    },
    drawWave({
               canvas,
               dataArray,
               bgcolor = 'rgb(200, 200, 200)',
               lineWidth = 2,
               lineColor = 'rgb(0, 0, 0)',
             }) {
      if (!canvas) return;

      const ctx = canvas.getContext('2d');
      const bufferLength = dataArray.length;
      // 一个点占多少位置，共有bufferLength个点要绘制
      const sliceWidth = canvas.width / bufferLength;
      // 绘制点的x轴位置
      let x = 0;

      // 填充背景色
      ctx.fillStyle = bgcolor;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // 设定波形绘制颜色
      ctx.lineWidth = lineWidth;
      ctx.strokeStyle = lineColor;

      ctx.beginPath();

      for (let i = 0; i < bufferLength; i++) {
        const v = dataArray[i] / 128;
        const y = (v * canvas.height) / 2;

        if (i === 0) {
          // 第一个点
          ctx.moveTo(x, y);
        } else {
          // 剩余的点
          ctx.lineTo(x, y);
        }
        // 依次平移，绘制所有点
        x += sliceWidth;
      }

      // 最后一个点
      ctx.lineTo(canvas.width, canvas.height / 2);
      ctx.stroke();
    },
  },
};
</script>
<style lang="scss" scoped>
.BaseRecorder {
  text-align: center;
  & > div {
    margin: 20px 0;
  }
  &-wave {
    canvas {
      width: 400px;
      height: 100px;
      border: 1px solid #ccc;
    }
  }
}
.recorder_state {
  font-size: 30px;
  color: red;
}
</style>


