<template>
  <div class="hello">
    <h1>Azure Speech Test</h1>
    <div class="container">
      <textarea v-model="msg"/>
      <div class="mic" @click="record">
        <i class="fas fa-microphone" v-bind:class="{ recording }"></i>
      </div>
    </div>
    <h4>Uses <a href="https://github.com/lkennxyz/azure-stt-helper">@lkennxyz/azure-stt-helper</a> to make make the request to azure, and <a href="https://recordrtc.org/">RecordRTC.js</a> for recording.</h4>
  </div>
</template>

<script>
import { RecordRTCPromisesHandler, StereoAudioRecorder } from 'recordrtc';
import { azureSTT, getToken } from '@lkennxyz/azure-stt-helper';

export default {
  name: 'SpeechTest',
  data() {
    return {
      recording: false,
      msg: '',
      speed: '',
      stream: '',
      region: 'westeurope',
      start: '',
      end: '',
      token: '',
    };
  },
  async mounted() {
    this.stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    console.log(process.env);
    const subscriptionKey = process.env.VUE_APP_SUBSCRIPTION_KEY;
    this.token = await getToken({ region: this.region, subscriptionKey });
  },
  methods: {
    async record() {
      this.recording = true;
      const recorder = new RecordRTCPromisesHandler(this.stream, {
        type: 'audio',
        mimeType: 'audio/wav',
        recorderType: StereoAudioRecorder,

      });
      recorder.startRecording();
      const sleep = m => new Promise(r => setTimeout(r, m));
      await sleep(3000);
      await recorder.stopRecording();
      const blob = await recorder.getBlob();
      this.recording = false;
      this.speech(blob);
    },
    async speech(wav) {
      const result = await azureSTT({region: this.region, token: this.token, language: 'en-GB', wav});
      this.msg = result.DisplayText;
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.container {
  display: flex;
  flex-direction: row;
  justify-content: center;
}
.mic {
  margin-left: 16px;
}
.recording {
  color: red;
}
</style>
