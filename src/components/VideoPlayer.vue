<template>
  <div class="video-parent">
    <div class="playing-video-container">
      <video ref="videoPlayer" controls @loadedmetadata="onLoadedMetadata" @timeupdate="onTimeUpdate" @play="onPlay" />
      <span v-show="currentCaption.length > 0" class="video-caption">{{ currentCaption }}</span>
    </div>
    <div class="video-container">
      <input id="input-video-url" v-model="videoUrl" placeholder="Enter Video URL (in .mp4 format)" />
      <button @click="loadVideo">Load</button>
    </div>

    <caption-list :videoStarted="videoStarted" :captions="captions" :videoReady="videoReady" :videoDuration="videoDuration" @add-caption="addCaption"
      @remove-caption="removeCaption" class="caption-list" />
  </div>
</template>

<script>
import CaptionList from './CaptionList.vue';

export default {
  components: {
    CaptionList,
  },
  data() {
    return {
      videoUrl: '',
      videoReady: false,
      videoDuration: 0,
      captions: [],
      videoStarted: false,
      currentCaption: '',
    };
  },
  methods: {
    onPlay() {
      this.videoStarted = true;
      this.captions.sort((a, b) => a.startTime - b.startTime);
    },
    onTimeUpdate(event) {
      const currentTime = event.target.currentTime;
      const caption = this.captions.find(c => c.startTime <= currentTime && c.endTime >= currentTime);
      if (caption) {
        this.currentCaption = caption.captionText;
      }else {
        this.currentCaption = "";
      }
    },
    loadVideo() {
      this.$refs.videoPlayer.src = this.videoUrl;
      this.captions = [];
      this.videoReady = false;
    },
    onLoadedMetadata(event) {
      this.videoReady = true;
      this.videoDuration = event.target.duration;
    },
    addCaption(captionData) {
      this.captions = [...this.captions, captionData];
    },
    removeCaption(index) {
      this.captions.splice(index, 1);
    }
  }
};
</script>

<style>
input {
  padding: 8px;
  margin-right: 10px;
  border: 1px solid #ddd;
}

video {
  width: 100%;
  height: auto;
}

button {
  padding: 5px 10px;
  cursor: pointer;
}

.caption-list {
  width: 100%;
}

.caption {
  position: absolute;
  bottom: 50px;
  left: 20px;
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
}

.video-parent {
  display: flex;
  flex-direction: column;
}

.video-container {
  width: 100%;
  display: flex;
  margin-top: 10px;
}

.playing-video-container {
  position: relative;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.video-caption {
  position: absolute;
  bottom: 80px;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
}

.video-container input {
  flex: 8;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.video-container button {
  flex: 2;
  background-color: #383838;
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
}
</style>
