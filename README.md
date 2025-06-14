# Vue Polygon Drawer

[![NPM Version](https://img.shields.io/npm/v/vue-srt-video-player.svg)](https://www.npmjs.com/package/vue-srt-video-player)
[![NPM Downloads](https://img.shields.io/npm/dm/vue-srt-video-player.svg)](https://www.npmjs.com/package/vue-srt-video-player)
[![License](https://img.shields.io/npm/l/vue-srt-video-player.svg)](https://github.com/Ali-Arya/vue-srt-video-player/blob/main/LICENSE)

A minimal, dependency-free Vue 3 video player that supports `.srt` subtitles and visual cue markers. Built with the Composition API and styled for plug-and-play use.

---

### Features

- 🎥 **Video Playback** – Play any video file (`.mp4`, `.webm`, etc.)
- 📝 **SRT Subtitle Support** – Add subtitles using standard `.srt` files
- ⏱️ **Cue Points** – Show labeled markers on the video timeline
- 🌈 **Simple Styling** – Comes with a clean, responsive design
- ⚙️ **Lightweight** – No external dependencies, just Vue 3
---

### Installation

Install the package using your favorite package manager.

```bash
npm install vue-srt-video-player
```

or

```bash
yarn add vue-srt-video-player
```

---

### Usage

The component is self-contained. All you need to do is import the component and its stylesheet into your Vue application.

Here is a basic example in a Vue 3 component using `<script setup>`:

```vue
<template>
  <div>
    <VideoPlayer />
  </div>
</template>

<script setup>
// 1. Import the component
import VideoPlayer from 'vue-srt-video-player'

// 2. IMPORTANT: Import the stylesheet
import 'vue-srt-video-player/style.css'
</script>

<style>

```

That's it! The component manages its own state internally.

---

### License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.