<template>
  <div class="caption-list-container">
    <div class="input-container">
      <div class="input-row">
        <input v-model="newCaption.captionText" placeholder="Enter the Caption" />
      </div>
      <div class="input-row">
        <input v-model="newCaption.startTime" placeholder="Start Time (min:sec)" />
        <input v-model="newCaption.endTime" placeholder="End Time (min:sec)" />
      </div>
      <button class="btn-add" @click="addCaption" :disabled="!videoReady">Add</button>
    </div>
    <table class="caption-table" v-show="this.captions.length > 0">
      <thead>
        <tr>
          <th>Caption</th>
          <th>Start Time</th>
          <th>End Time</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(caption, index) in captions" :key="index">
          <td>{{ caption.captionText }}</td>
          <td>{{ formatTime(caption.startTime) }}</td>
          <td>{{ formatTime(caption.endTime) }}</td>
          <td><button @click="removeCaption(index)">✕</button></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  props: {
    captions: {
      type: Array,
      required: true
    },
    videoReady: {
      type: Boolean,
      required: true
    },
    videoDuration: {
      type: Number,
      required: true
    },
    videoStarted: {
      type: Boolean,
      required: true
    },
  },
  data() {
    return {
      newCaption: {
        captionText: '',
        startTime: '',
        endTime: ''
      }
    };
  },
  methods: {
    addCaption() {
      if (!this.videoReady) {
        alert("Please load the video first.");
        return;
      } 
      if (this.videoStarted) {
        alert("Cannot add caption in between on a playing video.");
        return;
      }
      if (
        this.newCaption.captionText.trim() !== '' &&
        this.newCaption.startTime !== '' &&
        this.newCaption.endTime !== ''
      ) {
        const startTimeInSeconds = this.convertToSeconds(this.newCaption.startTime);
        const endTimeInSeconds = this.convertToSeconds(this.newCaption.endTime);
        if (startTimeInSeconds === null || endTimeInSeconds === null) {
          alert('Invalid time format. Please use min:sec format.');
          return;
        }

        if (startTimeInSeconds > this.videoDuration || endTimeInSeconds > this.videoDuration) {
          alert('Start time or end time is more than the video length.');
          return;
        }

        if (endTimeInSeconds <= startTimeInSeconds) {
          alert("Invalid start/end time for caption.")
        }

        for (let caption of this.captions) {
          if (
            (startTimeInSeconds >= caption.startTime && startTimeInSeconds < caption.endTime) ||
            (endTimeInSeconds > caption.startTime && endTimeInSeconds <= caption.endTime) ||
            (startTimeInSeconds <= caption.startTime && endTimeInSeconds >= caption.endTime)
          ) {
            alert("New caption start/end time overlaps with an existing caption.");
            return;
          }
        }

        const caption = {
          captionText: this.newCaption.captionText,
          startTime: startTimeInSeconds,
          endTime: endTimeInSeconds
        };
        this.$emit('add-caption', caption);
        this.newCaption.captionText = '';
        this.newCaption.startTime = '';
        this.newCaption.endTime = '';
      } 
    },
    removeCaption(index) {
      if (this.videoStarted) {
        alert("Cannot remove caption in between on a playing video.");
        return;
      }
      this.$emit('remove-caption', index);
    },
    formatTime(seconds) {
      const date = new Date(null);
      date.setSeconds(seconds);
      return date.toISOString().substr(14, 5);
    },
    convertToSeconds(time) {
      const parts = time.split(':');
      if (parts.length === 2) {
        const minutes = parseInt(parts[0], 10);
        const seconds = parseInt(parts[1], 10);
        if (!isNaN(minutes) && !isNaN(seconds)) {
          return minutes * 60 + seconds;
        }
      }
      return null;
    }
  }
};
</script>

<style scoped>
.caption-list-container {
  margin-top: 20px;
}

.input-container {
  margin-bottom: 20px;
}

.input-row {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.input-row input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.btn-add {
  border-radius: 10px;
  padding: 7px 75px;
  border: 1px solid black;
  background-color: rgb(56, 56, 56);
  color: white;
}

.caption-table {
  width: 100%;
  border-collapse: collapse;
}

.caption-table th,
.caption-table td {
  border: 1px solid #ddd;
  padding: 8px;
}

.caption-table th {
  background-color: #e1e1e1;
  text-align: left;
}

.caption-table td button {
  background: none;
  border: none;
  color: red;
  cursor: pointer;
  font-size: 16px;
}
</style>
