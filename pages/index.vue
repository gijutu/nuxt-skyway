<template>
<section class="container">
  <div>
    <logo/>
    <video id="their-video" width="300" autoplay playsinline></video>
    <video id="my-video" muted="true" width="300" autoplay playsinline></video>

    <div class="main">
      <h2>manabyのビデオチャットテスト</h2>
      <div class="microphone mb-1">
        マイク:
        <select v-model="selectedAudio" @change="onChange">
          <option disabled value="">Please select one</option>
          <option v-for="(audio, key, index) in audios" v-bind:key="index" :value="audio.value">
            {{ audio.text }}
          </option>
        </select>
      </div>
      <div class="camera mb-1">
        カメラ:
        <select v-model="selectedVideo" @change="onChange">
          <option disabled value="">Please select one</option>
          <option v-for="(video, key, index) in videos" v-bind:key="index" :value="video.value">
            {{ video.text }}
          </option>
        </select>
      </div>
      <div>
        <p>Your id: <span id="my-id">{{peerId}}</span></p>
        <p>他のブラウザでこのIDをコールしましょう。</p>
        <h3>コールする</h3>
        <input v-model="calltoid" placeholder="call id">
        <button @click="makeCall" class="button--green">Call</button>
      </div>
    </div>

  </div>
</section>

</template>

<script>
import Logo from '~/components/Logo.vue'
import Peer from 'skyway-js';

export default {
  components: {
    Logo
  },
  data () {
    return {
      APIKey: process.env.SKYWAY,
      selectedAudio: '',
      selectedVideo: '',
      audios: [],
      videos: [],
      localStream: null,
      peerId: '',
      calltoid: ''
    }
  },

  methods: {
    onChange: function () {
      if(this.selectedAudio != '' && this.selectedVideo != ''){
        this.connectLocalCamera();
      }
    },

    connectLocalCamera: async function(){
      const constraints = {
        audio: this.selectedAudio ? { deviceId: { exact: this.selectedAudio } } : false,
        video: this.selectedVideo ? { deviceId: { facingMode: { exact: this.selectedVideo } } } : false
      }

      const stream = await navigator.mediaDevices.getUserMedia(constraints);
      document.getElementById('my-video').srcObject = stream;
      this.localStream = stream;
    },

    makeCall: function () {
      console.log('makeCall');
      const call = this.peer.call(this.calltoid, this.localStream);
      this.connect(call);
    },

    connect: function (call) {
      call.on('stream', stream => {
        const el = document.getElementById('their-video');
        el.srcObject = stream;
        el.play();
      });
    }
  },

  mounted: function () {
    this.peer = new Peer({
      key:   this.APIKey,
      debug: 3,
    });

    this.peer.on('open', () => {
      this.peerId = this.peer.id
    });

    this.peer.on('call', call => {
      call.answer(this.localStream);
      this.connect(call);
    });

    //デバイスへのアクセス
    navigator.mediaDevices.enumerateDevices()
    .then((deviceInfos) => {
      for (let i = 0; i !== deviceInfos.length; ++i) {
        const deviceInfo = deviceInfos[i]
        if (deviceInfo.kind === 'audioinput') {
          this.audios.push({
            text: deviceInfo.label ||
            `Microphone ${this.audios.length + 1}`,
            value: deviceInfo.deviceId
          })
        } else if (deviceInfo.kind === 'videoinput') {
          this.videos.push({
            text: deviceInfo.label ||
            `Camera  ${this.videos.length - 1}`,
            value: deviceInfo.deviceId
          })
        }
      }
    })
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
