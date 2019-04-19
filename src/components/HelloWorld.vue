<template>
  <div class="wrap photo-wrap flex-column">
    <!-- 自动抓拍 -->
    <div class="flex-item" v-show="!no_video">
      <div class="out-circle" v-bind:class="[picPath? '' : 'anim']">
        <div class="contentHolder">
          <p v-show="!picPath">
            <video id="video" autoplay></video>
            <canvas style hidden="hidden" id="canvas" width="500" height="500"></canvas>
          </p>
          <img v-show="picPath" :src="picPath">
          <div class="tip-box">{{picPath ? '抓取成功':'正在抓取'}}</div>
        </div>
      </div>
      <p class="text-center text">{{picPath ? '采集成功，请按确认提交':'对准人脸，系统自动抓拍'}}</p>
    </div>

  </div>
</template>
<script type="text/javascript">
export default {
  data() {
    return {
      intervalTimer: null,
      isReplay: false, //防止多次点击重试
      picPath: null, //图片路径
      time: 0, //识别次数
      getauthCodeTip: "", //获取验证码按钮内容
      stream: null,
      no_video: false,

      uploadLoading: {
        loading: false,
        msg: "正在上传"
      }
    };
  },
  created() {},
  mounted() {
    var Media = document.getElementById("video");
    window.addEventListener("DOMContentLoaded", this.videoListener(), false);

    // 微信
    document.addEventListener(
      "WeixinJSBridgeReady",
      function() {
        Media.play();
      },
      false
    );
  },
  watch: {},
  computed: {},
  methods: {
    videoListener() {
      let _this = this;
      //这段代 主要是获取摄像头的视频流并显示在Video 签中
      var canvas = document.getElementById("canvas"),
        context = canvas.getContext("2d"),
        video = document.getElementById("video"),
        videoObj = { video: true },
        errBack = function(error) {
          alert("error: "+error);
          let msg = "";
          if (
            error.name == "NotFoundError" ||
            error.name == "DevicesNotFoundError"
          ) {
            msg = error.msg || "无法找到视频设备";
          } else if (
            error.name == "NotReadableError" ||
            error.name == "TrackStartError"
          ) {
            msg = error.msg || "未能分配视频源";
          } else if (
            error.name == "OverconstrainedError" ||
            error.name == "ConstraintNotSatisfiedErrror"
          ) {
            msg = error.msg || "约束无法满足";
          } else if (
            error.name == "NotAllowedError" ||
            error.name == "PermissionDeniedError"
          ) {
            msg = error.msg || "用户代理或者当前平台不允许该请求";
          } else if (error.name == "TypeError") {
            msg = error.msg || "必须至少请求一个音频和视频";
          } else {
            msg = error.msg || "其他错误";
          }
          console.log("Video capture error: ", error.code);
          return;
        };

      //alert("navigator.mediaDevices.getUserMedia: >>>" + navigator && navigator.mediaDevices && navigator.mediaDevices.getUserMedia);
      //navigator.getUserMedia这个写法在Opera中好像是navigator.getUserMedianow
      //更新兼容火狐浏览器
      if (
        navigator.getUserMedia ||
        navigator.webkitGetUserMedia ||
        navigator.mozGetUserMedia ||
        (navigator.mediaDevices && navigator.mediaDevices.getUserMedia)
      ) {
        this.no_video = false;
        navigator.getUserMedia =
          navigator.getUserMedia ||
          navigator.webkitGetUserMedia ||
          navigator.mozGetUserMedia ||
          navigator.mediaDevices.getUserMedia;
        // alert("if: >>> "+navigator.getUserMedia);
        navigator.getUserMedia(videoObj,function(stream) {
            alert("stream: "+stream);
            video.srcObject = stream;
            video.play();
            _this.stream = stream;
            //拍照每秒一次
            _this.intervalTimer = setInterval(function() {
              context.drawImage(video, 0, 0, 520, 520);
              _this.CatchCode();
            }, 10000);
          },errBack);
      } else {
        // alert("else: >>>" );
        this.no_video = true;
       
      }
    },
    dataURItoBlob(base64Data) {
      var byteString;
      if (base64Data.split(",")[0].indexOf("base64") >= 0)
        byteString = atob(base64Data.split(",")[1]);
      else byteString = unescape(base64Data.split(",")[1]);
      var mimeString = base64Data
        .split(",")[0]
        .split(":")[1]
        .split(";")[0];
      var ia = new Uint8Array(byteString.length);
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }
      return new Blob([ia], { type: mimeString });
    },

    // 上传图片  识别图片
    CatchCode() {
      let _this = this;
      var canvans = document.getElementById("canvas");
      if (!canvans) {
        return;
      }
      //获取浏览器页面的画布对象
      //以下开始 数据
      var imageBase64 = canvans.toDataURL();
      var blob = this.dataURItoBlob(imageBase64); // 上一步中的函数
      console.log(blob)
      // this.picPath = imgPath;
      
    },

    // 上传图片  识别图片
    canvasSelImg(eve) {
      
    },

   
  },
  components: {}
};
</script>
<style rel="stylesheet/css">
.photo-wrap {
  background-image: unset;
  padding-bottom: 40px;
}
.out-circle {
  margin: 50px auto;
  padding: 20px;
  width: 550px;
  height: 550px;
  border-radius: 100%;
  border: 5px solid #3d57ff;
}
.anim {
  animation: circleBorder 2s infinite;
}

@keyframes circleBorder {
  0% {
    border: 5px solid #3d57ff;
  }
  25% {
    border: 5px solid #44abf9;
  }
  50% {
    border: 5px solid #4cfff4;
  }
  75% {
    border: 5px solid #407efc;
  }
  100% {
    border: 5px solid #3d57ff;
  }
}
.contentHolder {
  background: #fff;
  width: 550px;
  height: 550px;
  border-radius: 100%;
  border: 1px solid #999;
  overflow: hidden;
  position: relative;

  
}
img {
    width: 100%;
}

video {
  width: 550px;
  height: 550px;
}
.tip-box{
      background: rgba(0, 0, 0, 0.36);
      position: absolute;
      width: 100%;
      bottom: 0;
      text-align: center;
      padding: 30px 0;
      color: #fff;
    }
  
  
</style>


