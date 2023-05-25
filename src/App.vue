<template>
  <div class="main">
    <div class="main-card">
      <h1>Тестовое задание CDNvideo</h1>
      <div class="video-player" id="video-player">
        <video id="video" style="width: 100%"></video>
        <div class="video-controls">
          <div id="play-control">
            <img
              src="./assets/controls/pause.svg"
              width="20"
              v-if="videoPlay"
            />
            <img src="./assets/controls/play.svg" width="20" v-else />
          </div>
          <div class="progress-control" style="width: 80%">
            <span>{{ fancyTimeFormat(currentPosition) }}</span>
            <input
              style="width: 100%"
              id="progress-bar"
              type="range"
              name=""
              step="1"
              :value="currentPosition"
              @change="(event) => changeCurrentPosition(event)"
              :max="mediaDuration"
            />
            <span>{{ fancyTimeFormat(mediaDuration) }}</span>
          </div>
          <div id="fullscreen-control">
            <img src="./assets/controls/fullscreen.svg" width="16" />
          </div>
        </div>
      </div>
      <div class="video-statistics">
        <div class="video-statistics-tabs">
          <div
            class="video-statistics-tab"
            :class="activeTab === 1 ? 'video-statistics-tab-active' : ''"
            @click="activeTab = 1"
          >
            <p>Position metrics</p>
          </div>

          <div
            class="video-statistics-tab"
            :class="activeTab === 2 ? 'video-statistics-tab-active' : ''"
            @click="activeTab = 2"
          >
            <p>Buffer metrics</p>
          </div>

          <div
            class="video-statistics-tab"
            :class="activeTab === 3 ? 'video-statistics-tab-active' : ''"
            @click="activeTab = 3"
          >
            <p>Media Info</p>
          </div>
        </div>

        <div class="video-statistics-block" v-if="activeTab === 1">
          <p>
            Current position:
            <span>{{ fancyTimeFormat(currentPosition) }}</span>
          </p>
          <p>
            Media duration: <span>{{ fancyTimeFormat(mediaDuration) }}</span>
          </p>
        </div>

        <div class="video-statistics-block" v-if="activeTab === 2">
          <p style="color: #000000">
            Current Buffer size: <span> {{ currentBuffer + 's' }}</span>
          </p>
        </div>

        <div class="video-statistics-block" v-if="activeTab === 3">
          <p style="color: #000000">
            Fragments amount: <span> {{ fragmentsAmount }}</span>
          </p>
          <p style="color: #000000">
            Current fragment: <span> {{ currentFragment }}</span>
          </p>
          <p style="color: #000000">
            Media url:
            <span>
              <a :href="videoUrl">{{ videoUrl }}</a>
            </span>
          </p>
          <p style="color: #000000">
            Current bitrate: <span> {{ currentBitrate }}</span>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import Hls from 'hls.js';
import { onMounted, ref } from 'vue';

const currentPosition = ref();
const mediaDuration = ref();

const fragmentsAmount = ref();
const currentFragment = ref();

const currentBuffer = ref();
const currentBitrate = ref();

const videoUrl = ref();
const activeTab = ref(0);

const videoPlay = ref(false);
const fullscreen = ref(false);

const fancyTimeFormat = (duration) => {
  const hrs = ~~(duration / 3600);
  const mins = ~~((duration % 3600) / 60);
  const secs = ~~duration % 60;
  let ret = '';
  if (hrs > 0) {
    ret += '' + hrs + ':' + (mins < 10 ? '0' : '');
  }
  ret += '' + mins + ':' + (secs < 10 ? '0' : '');
  ret += '' + secs;

  return ret;
};

onMounted(() => {
  const video = document.getElementById('video');
  const videoSource =
    'https://devstreaming-cdn.apple.com/videos/streaming/examples/img_bipbop_adv_example_ts/master.m3u8';

  if (Hls.isSupported()) {
    const config = {
      startPosition: 70,
    };

    const hls = new Hls(config);
    hls.loadSource(videoSource);
    hls.attachMedia(video);

    hls.on(Hls.Events.FRAG_CHANGED, function (event, data) {
      currentFragment.value = data.frag.sn + 1;
    });

    hls.on(Hls.Events.LEVEL_PTS_UPDATED, function (event, data) {
      if (data.type === 'video') currentBitrate.value = data.level.bitrate;
    });

    hls.on(Hls.Events.BUFFER_APPENDING, function () {
      currentBuffer.value =
        hls.mainForwardBufferInfo.end - hls.mainForwardBufferInfo.start;
    });

    hls.on(Hls.Events.LEVEL_LOADED, function (event, data) {
      mediaDuration.value = data.details.totalduration;
      fragmentsAmount.value = data.details.fragments.length;
      videoUrl.value = data.details.url;
    });
  }

  video.addEventListener(
    'timeupdate',
    () => (currentPosition.value = video.currentTime)
  );

  document.getElementById('play-control').addEventListener('click', () => {
    videoPlay.value = !videoPlay.value;
    videoPlay.value ? video.play() : video.pause();
  });

  document
    .getElementById('fullscreen-control')
    .addEventListener('click', () => {
      fullscreen.value = !fullscreen.value;
      fullscreen.value
        ? document.getElementById('video-player').requestFullscreen()
        : document.exitFullscreen();
    });

  document
    .getElementById('progress-bar')
    .addEventListener('change', (event) => {
      video.currentTime = event.target.value;
    });
});
</script>

<style lang="scss" scoped>
.main {
  display: flex;
  justify-content: center;

  .main-card {
    padding: 24px 4px;
    display: flex;
    flex-direction: column;
    gap: 40px;
    width: 80%;
    align-items: center;

    h1 {
      font-size: 24px;
      color: black;
    }
  }
}

.video-statistics {
  color: black;
  width: 100%;

  .video-statistics-tabs {
    display: flex;
    gap: 40px;
    width: 100%;

    .video-statistics-tab {
      text-align: center;
      cursor: pointer;
      padding: 10px 20px;
      background: #b7b7b7;
      width: calc(100% / 3);
    }

    .video-statistics-tab:hover {
      background: #dadada;
    }

    .video-statistics-tab-active {
      background: #dadada;
    }
  }
  .video-statistics-block {
    margin: 20px 0;

    span {
      font-weight: 600;
    }
  }
}

.video-player {
  position: relative;

  .video-controls {
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: absolute;
    bottom: 0;
    height: 40px;
    padding: 0 20px;
    width: 100%;
    background: rgba($color: #dadada, $alpha: 0.7);

    div {
      cursor: pointer;
    }
  }
}

.video-player-fullscreen {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}

.progress-control{
  display: flex;
  align-items: center;
  gap: 16px;
  width: 80%;
}
</style>
