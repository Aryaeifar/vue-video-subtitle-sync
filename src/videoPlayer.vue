<script lang="ts" setup>
import { ref, onBeforeUnmount, computed } from 'vue';

const videoUrl = ref<string | null>(null);
const subtitleUrl = ref<string | null>(null);
const videoPlayerKey = ref(0);
const videoElement = ref<HTMLVideoElement | null>(null);
const isPlaying = ref(false);
const currentTime = ref(0);
const duration = ref(0);
const subtitleCues = ref<number[]>([]);

const parseSrtTimes = (srtContent: string): number[] => {
  const srtLineRegex = /(\d{2}:\d{2}:\d{2},\d{3})\s-->/g;
  const cues: number[] = [];
  let match;
  while ((match = srtLineRegex.exec(srtContent)) !== null) {
    const [hours, minutes, seconds, milliseconds] = match[1].split(/[:,]/).map(Number);
    const timeInSeconds = hours * 3600 + minutes * 60 + seconds + milliseconds / 1000;
    cues.push(timeInSeconds);
  }
  return cues;
};

const convertSrtToVtt = (srtContent: string): string => {
  return `WEBVTT\n\n${srtContent.replace(/(\d{2}:\d{2}:\d{2}),(\d{3})/g, '$1.$2')}`;
};

const handleVideoUpload = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (videoUrl.value) URL.revokeObjectURL(videoUrl.value);
  videoUrl.value = file ? URL.createObjectURL(file) : null;
  isPlaying.value = false;
  currentTime.value = 0;
  duration.value = 0;
};

const handleSrtUpload = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (subtitleUrl.value) URL.revokeObjectURL(subtitleUrl.value);
  subtitleCues.value = [];

  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      const srtContent = e.target?.result as string;
      subtitleCues.value = parseSrtTimes(srtContent);
      const vttContent = convertSrtToVtt(srtContent);
      const vttBlob = new Blob([vttContent], { type: 'text/vtt' });
      subtitleUrl.value = URL.createObjectURL(vttBlob);
      videoPlayerKey.value++;
    };
    reader.readAsText(file);
  } else {
    subtitleUrl.value = null;
  }
};

const togglePlay = () => {
  if (videoElement.value) {
    if (isPlaying.value) {
      videoElement.value.pause();
    } else {
      videoElement.value.play();
    }
  }
};

const stopVideo = () => {
  if (videoElement.value) {
    videoElement.value.pause();
    videoElement.value.currentTime = 0;
  }
};

const handleSeek = (event: Event) => {
  if (videoElement.value) {
    const target = event.target as HTMLInputElement;
    videoElement.value.currentTime = Number(target.value);
  }
};

const handleLoadedMetadata = () => {
  if (videoElement.value) {
    duration.value = videoElement.value.duration;
  }
};

const handleTimeUpdate = () => {
  if (videoElement.value) {
    currentTime.value = videoElement.value.currentTime;
  }
};

const progressBarStyles = computed(() => {
  const progressPercent = duration.value > 0 ? (currentTime.value / duration.value) * 100 : 0;
  return {
    background: `linear-gradient(to right, #000 ${progressPercent}%, #fff ${progressPercent}%)`
  };
});

onBeforeUnmount(() => {
  if (videoUrl.value) URL.revokeObjectURL(videoUrl.value);
  if (subtitleUrl.value) URL.revokeObjectURL(subtitleUrl.value);
});
</script>

<template>
  <div class="container">
    <div class="card">
      <div class="card-header">
        <h2 class="card-title">Video Player</h2>
        <p class="card-description">
          Upload your own video and .srt subtitle file to play them locally.
        </p>
      </div>
      <div class="card-content">
        <div class="input-grid">
          <div class="input-group">
            <label for="video-upload">1. Upload Video</label>
            <input 
              id="video-upload" 
              type="file" 
              class="input-file"
              accept="video/*" 
              @change="handleVideoUpload" 
            />
          </div>
          <div class="input-group">
            <label for="srt-upload">2. Upload Subtitle (.srt)</label>
            <input 
              id="srt-upload" 
              type="file" 
              class="input-file"
              accept=".srt" 
              @change="handleSrtUpload" 
            />
          </div>
        </div>

        <div class="video-wrapper">
          <video
            v-if="videoUrl"
            ref="videoElement"
            :key="videoPlayerKey"
            :src="videoUrl"
            crossorigin="anonymous"
            class="video-player"
            @loadedmetadata="handleLoadedMetadata"
            @timeupdate="handleTimeUpdate"
            @play="isPlaying = true"
            @pause="isPlaying = false"
            @ended="isPlaying = false"
          >
            <track
              v-if="subtitleUrl"
              :src="subtitleUrl"
              kind="subtitles"
              label="English"
              default
            >
          </video>
          <div v-else class="video-placeholder">
            <p>Your video will appear here</p>
          </div>
        </div>
        
        <div v-if="videoUrl" class="custom-controls">
          <button @click="togglePlay" class="control-button">
            <svg v-if="!isPlaying" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="6" y="4" width="4" height="16"></rect><rect x="14" y="4" width="4" height="16"></rect></svg>
          </button>
          <button @click="stopVideo" class="control-button">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect></svg>
          </button>
          <div class="progress-bar-wrapper">
            <input 
              type="range"
              class="progress-bar"
              :value="currentTime"
              :max="duration"
              :style="progressBarStyles"
              @input="handleSeek"
            >
            <div class="cue-markers" v-if="duration > 0">
              <div 
                v-for="time in subtitleCues" 
                :key="time" 
                class="cue-marker" 
                :style="{ left: `${(time / duration) * 100}%` }"
              ></div>
            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.container {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  padding: 2rem;
  display: flex;
  justify-content: center;
}

.card {
  width: 100%;
  max-width: 800px;
  background-color: hsl(0 0% 100%);
  color: hsl(222.2 84% 4.9%);
  border-radius: 0.5rem;
  border: 1px solid hsl(214.3 31.8% 91.4%);
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
}

.card-header {
  padding: 1.5rem;
  border-bottom: 1px solid hsl(214.3 31.8% 91.4%);
}

.card-title {
  font-size: 1.5rem;
  font-weight: 600;
  letter-spacing: -0.025em;
  margin: 0;
}

.card-description {
  margin-top: 0.5rem;
  font-size: 0.875rem;
  color: hsl(215.4 16.3% 46.9%);
}

.card-content {
  padding: 1.5rem;
}

.input-grid {
  display: grid;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
  
  @media (min-width: 768px) {
    grid-template-columns: repeat(2, 1fr);
  }
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

label {
  font-size: 0.875rem;
  font-weight: 500;
}

.input-file {
  display: flex;
  height: 2.5rem;
  width: 100%;
  border-radius: calc(0.5rem - 2px);
  border: 1px solid hsl(214.3 31.8% 91.4%);
  background-color: hsl(0 0% 100%);
  padding: 0.25rem 0.375rem;
  font-size: 0.875rem;
  color: hsl(215.4 16.3% 46.9%);

  &::file-selector-button {
    border: none;
    padding: 0.25rem 0.75rem;
    height: 100%;
    margin-right: 0.75rem;
    border-radius: calc(0.5rem - 4px);
    background-color: hsl(222.2 84% 4.9%);
    color: hsl(0 0% 100%);
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s;
    
    &:hover {
      background-color: hsl(222.2 84% 4.9% / 0.9);
    }
  }
}

.video-wrapper {
  width: 100%;
  aspect-ratio: 16 / 9;
  background-color: #020817;
  border-radius: calc(0.5rem - 2px);
  overflow: hidden;
  margin-bottom: 1rem;
}

.video-player {
  width: 100%;
  height: 100%;
}

.video-placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #64748b;
  font-size: 0.875rem;
}

.custom-controls {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 0.5rem;
}

.control-button {
  background: none;
  border: none;
  padding: 0.25rem;
  cursor: pointer;
  color: hsl(222.2 84% 4.9%);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background-color 0.2s;

  &:hover {
    background-color: hsl(214.3 31.8% 91.4%);
  }

  svg {
    width: 20px;
    height: 20px;
  }
}

.progress-bar-wrapper {
  position: relative;
  width: 100%;
  display: flex;
  align-items: center;
}

.progress-bar {
  -webkit-appearance: none;
  appearance: none;
  width: 100%;
  height: 15px;
  border-radius: 5px;
  outline: none;
  cursor: pointer;
  border: 1px solid hsl(214.3 31.8% 91.4%);

  &::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #000;
    cursor: pointer;
    border: 2px solid #fff;
    box-shadow: 0 0 2px rgba(0,0,0,0.5);
    margin-top: -5px; 
  }

  &::-moz-range-thumb {
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #000;
    cursor: pointer;
    border: 2px solid #fff;
  }
}

.cue-markers {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; 
  overflow: hidden;
}

.cue-marker {
  position: absolute;
  width: 2px;
  height: 100%;
  background-color: orange;
  transform: translateX(-50%);
}
</style>