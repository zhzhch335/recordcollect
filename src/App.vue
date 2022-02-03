/* eslint-disable no-debugger */
<template>
  <div id="app">
    <van-notice-bar text="每人的链接唯一，不要分享否则你的数据会被覆盖" />
    <van-notice-bar text="征集时间截至到 2月5号 23：59：59" />
    <van-notice-bar v-if="repeat" text="【警告】你提交过了，再提交会被覆盖" />
    <van-field v-model="name" label="昵称(可修改）" />
    <!-- <van-field v-model="avatorAddress" center clearable label="上传头像">
      <template #right-icon>
        <van-uploader v-if="!avatorAddress" :after-read="afterRead" />
        <van-uploader v-if="avatorAddress" :after-read="afterRead">
          {{ avatorAddress }}
        </van-uploader>
      </template>
      <template #input>
        <input style="display: none" type="text" />
      </template>
    </van-field> -->
    <van-field
      v-model="failSoundAddress"
      center
      clearable
      label="录制被击败语音（啊~之类的 最多3秒）"
    >
      <template #right-icon>
        <van-button
          type="primary"
          :disabled="isRecording"
          @click="record('failSoundAddress')"
          >{{
            countDown == 0 ? "开始录音" : `倒计时${countDown}秒`
          }}</van-button
        >
        <van-button
          style="margin-left: 10px"
          type="info"
          @click="$refs.failSound.play()"
          :disabled="isRecording || !failSoundAddress"
          >回放录音</van-button
        >
        <audio ref="failSound" autoplay="false" :src="failSoundAddress"></audio>
      </template>
      <template #input>
        <input style="display: none" type="text" />
      </template>
    </van-field>
    <van-field
      v-model="birthSoundAddress"
      center
      clearable
      label="录制“生日快乐”这四个字（最多4秒）"
    >
      <template #right-icon>
        <van-button
          type="primary"
          :disabled="isRecording"
          @click="record('birthSoundAddress')"
          >{{
            countDown == 0 ? "开始录音" : `倒计时${countDown}秒`
          }}</van-button
        >
        <van-button
          style="margin-left: 10px"
          type="info"
          @click="$refs.birthSoundAddress.play()"
          :disabled="isRecording || !birthSoundAddress"
          >回放录音</van-button
        >
        <audio
          ref="birthSoundAddress"
          autoplay="false"
          :src="birthSoundAddress"
        ></audio>
      </template>
      <template #input>
        <input style="display: none" type="text" />
      </template>
    </van-field>
    <van-button :disabled="!canSubmit" @click="submit()" type="primary">
      提交
    </van-button>
    <!-- <Progress strokeColor="#FF00AA" :value="(recordProcess / 30) * 100">
      <template v-slot:footer>
        <b>录制限时3秒</b>
      </template>
    </Progress> -->
  </div>
</template>

<script>
import axios from "axios";
// import Recorder from "js-audio-recorder";
// import Progress from "easy-circular-progress";
import { Toast, Dialog } from "vant";
import Recorder from "./recorder";

var audio_context;

export default {
  name: "App",
  components: {
    // Progress,
  },
  data() {
    return {
      host: "https://meme.xiaoxideai.ren",

      isRecording: false,
      countDown: 0,
      recordProcess: 0,

      recorder: null,

      repeat: false,
      canSubmit: false,

      id: "",
      name: "",
      failSoundAddress: "",
      birthSoundAddress: "",
    };
  },
  mounted() {
    this.id = location.href.match(/id=\d*/)[0].replace("id=", "");
    if (!this.id) {
      Toast("参数不对，请确认链接完整性或者管琛琛重新要一个");
      return;
    }
    axios({
      method: "get",
      url: `${this.host}/queryId?id=${this.id}`,
    }).then((res) => {
      var data = res.data;
      if (!data.errorCode) {
        this.name = data.data.name;
        if (data.data.failSoundAddress || data.data.birthSoundAddress) {
          this.repeat = true;
        }
        Dialog.confirm({
          title: "权限",
          message: "需要同意录音权限才能开始录音",
        }).then(() => {
          try {
            // webkit shim
            window.AudioContext =
              window.AudioContext || window.webkitAudioContext;
            navigator.getUserMedia =
              navigator.getUserMedia || navigator.webkitGetUserMedia;
            window.URL = window.URL || window.webkitURL;

            audio_context = new AudioContext();
            if (!navigator.getUserMedia) {
              Dialog.alert({
                message:
                  "如果录不上试试浏览器中选择“查看电脑版网页”或者换安卓或电脑来录（苹果有的版本我妹有搞定）",
              });
            }
            console.log("请设置Audio context .");
            console.log(
              "navigator.getUserMedia " +
                (navigator.getUserMedia ? "支持." : "不支持！")
            );
          } catch (e) {
            alert("当前浏览器不支持音频功能");
          }

          navigator.mediaDevices
            .getUserMedia({ audio: true })
            .then(this.startUserMedia)
            .catch(this.errorHandler);

          this.canSubmit = true;
        });
      } else {
        Toast(data.errorMessage);
        this.canSubmit = false;
      }
    });

    // navigator.getUserMedia = navigator.getUserMedia ||
    //   navigator.webkitGetUserMedia ||
    //   navigator.mozGetUserMedia;

    // navigator.mediaDevices.getUserMedia({ audio: true }, startUserMedia, function(e) {
    //   __log('No live audio input: ' + e);
    // });
  },
  methods: {
    // afterRead(e) {
    //   let formdata = new FormData();
    //   formdata.append("file", e.file);
    //   axios({
    //     method: "post",
    //     headers: {
    //       "Content-Type": "multipart/form-data;charset=UTF-8",
    //     },
    //     data: formdata,
    //     url: `${this.host}/uploadAvator`,
    //   })
    //     .then((res) => {
    //       var data = res.data;
    //       this.avatorAddress = data.data.filename;
    //     })
    //     .catch((err) => {
    //       console.log(err);
    //     });
    // },
    submit() {
      if (!this.name) {
        Toast("昵称妹写啊！");
        return;
      }
      if (!this.failSoundAddress) {
        Toast("战败语音妹录啊！");
        return;
      }
      if (!this.birthSoundAddress) {
        Toast("生日语音妹录啊！");
        return;
      }
      axios({
        method: "post",
        url: `${this.host}/addMeMeRen`,
        data: {
          id: this.id,
          name: this.name,
          failSoundAddress: this.failSoundAddress,
          birthSoundAddress: this.birthSoundAddress,
        },
      }).then((res) => {
        var data = res.data;
        if (!data.errorCode) {
          Toast(data.errorMessage);
        } else {
          Toast(`出错了，错误信息是${data.errorMessage},麻烦跟琛琛说一声`);
        }
      });
    },
    record(soundName) {
      this.countDown = soundName == "failSoundAddress" ? 3 : 4;
      var interval = setInterval(() => {
        if (this.countDown == 0) {
          clearInterval(interval);
          return;
        }
        this.countDown -= 1;
      }, 1000);
      this.startRecording();
      setTimeout(
        () => {
          this.stopRecording(soundName);
        },
        soundName == "failSoundAddress" ? 3000 : 4000
      );
    },

    startUserMedia(stream) {
      var input = audio_context.createMediaStreamSource(stream);

      // 如果你想让音频直接反馈
      //input.connect(audio_context.destination);
      //__log('Input connected to audio context destination.');
      var config = {
        sampleRate: 16000,
        bitRate: 16,
        mimeType: "audio/mp3",
      };

      this.recorder = new Recorder(input, config);
    },

    startRecording() {
      this.recorder && this.recorder.record();
      this.isRecording = true;
      // button.disabled = true;
      // button.nextElementSibling.disabled = false;
    },

    stopRecording(soundName) {
      this.recorder && this.recorder.stop();
      // button.disabled = true;
      // button.previousElementSibling.disabled = false;
      this.isRecording = false;
      this.createDownloadLink(soundName);

      this.recorder && this.recorder.clear();
    },

    createDownloadLink(soundName) {
      this.recorder &&
        this.recorder.exportAudio((blob) => {
          // var url = URL.createObjectURL(blob);
          // 使用ajax
          // $.ajax({
          //   url:'http://localhost:8000/v1/file/upload-file',
          //   type:'post',
          //   data:{file:url},
          //   success: function (data) {
          //     console.log(data);
          //   },
          //   error: function(data){
          //     console.log(data);
          //   }
          // })

          var formData = new FormData();
          formData.append("data", blob);
          // var name = $("input").val();
          // formData.append("file", $("#upload")[0].files[0]);
          // formData.append("name", name);
          axios({
            method: "post",
            headers: {
              "Content-Type": "multipart/form-data;charset=UTF-8",
            },
            data: formData,
            url: `${this.host}/uploadSound`,
          })
            .then((res) => {
              console.log(res);
              var data = res.data;
              this[soundName] = this.host + "/sound/" + data.data.filename;
            })
            .catch((err) => {
              console.log(err);
            });
          // $.ajax({
          //   // url: 'http://api.shudong.wang/v1/file/upload-file', // 存储音频接口
          //   url: "http://localhost:8080/uploadSound",
          //   type: "POST",
          //   data: formData,
          //   // 告诉jQuery不要去处理发送的数据
          //   processData: false,
          //   // 告诉jQuery不要去设置Content-Type请求头
          //   contentType: false,
          //   beforeSend: function () {
          //     console.log("正在进行，请稍候");
          //   },
          //   success: function (responseStr) {
          //     if (responseStr.status === 0) {
          //       console.log("成功" + responseStr);
          //     } else {
          //       console.log("失败");
          //     }
          //   },
          //   error: function (responseStr) {
          //     console.log("error");
          //   },
          // });
          // var li = document.createElement("li");
          // var au = document.createElement("audio");
          // var hf = document.createElement("a");

          // au.controls = true;
          // au.src = url;
          // hf.href = url;
          // hf.download = new Date().toISOString() + ".mp3";
          // hf.innerHTML = hf.download;
          // li.appendChild(au);
          // li.appendChild(hf);
          // recordingslist.appendChild(li);
        });
    },

    errorHandler(params) {
      console.log("No live audio input: " + params);
    },
  },
};
</script>

<style>
body {
  background: #b9b9b9;
  display: flex;
  justify-content: center;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

  display: flex;
  flex-direction: column;

  background: white;
}
</style>
