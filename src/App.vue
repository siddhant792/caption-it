<template>
  <div id="app">
    <VideoPlayer :captions="captions" class="video-player" />
  </div>
</template>

<script>
import VideoPlayer from './components/VideoPlayer.vue';

export default {
  components: {
    VideoPlayer
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background-color: #f0f0f0;
  margin: 0;
  padding: 30px;
}

.video-player {
  width: 100%;
}

#app {
  width: 60%;
  align-items: center;
  display: flex;
  flex-direction: column;
} 
</style>
