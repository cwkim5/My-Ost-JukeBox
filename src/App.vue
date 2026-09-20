<template>
  <div class="box">
    <h1>{{ playlist[currentIndex].title }}</h1>

    <div class="progress-container">
      <span class="time">{{ formatTime(currentTime) }}</span>
      <input
        type="range"
        min="0"
        :max="duration"
        step="0.1"
        v-model.number="currentTime"
        @input="onSeek"
      />
      <span class="time">{{ formatTime(duration) }}</span>
    </div>

    <button @click="playAudio">{{ isPlaying ? '⏸ PAUSE' : '▶ PLAY' }}</button>

    <div class="playlist-controls">
      <button @click="prevSong">◀ PREV</button>
      <button @click="nextSong">NEXT ▶</button>
    </div>

    <div class="volume-container">
      <label for="volumeRange"
        >🔊 Volume: {{ Math.round(volume * 100) }}%</label
      >
      <input
        id="volumeRange"
        type="range"
        min="0"
        max="1"
        step="0.01"
        v-model.number="volume"
        @input="changeVolume"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const playlist = ref([
  {
    title: 'STAGE THEME 1',
    url: 'https://github.com/cwkim5/My-Ost-JukeBox/raw/main/Text%20project%201.mp3',
  },
  {
    title: 'STAGE THEME 2',
    url: 'https://github.com/cwkim5/My-Ost-JukeBox/raw/main/STAGE%20THEME%202.mp3',
  },
  {
    title: 'STAGE THEME 3',
    url: 'https://github.com/cwkim5/My-Ost-JukeBox/raw/main/Theme%203.mp3',
  },
  {
    title: 'STAGE 1 BOSS THEME',
    url: 'https://github.com/cwkim5/My-Ost-JukeBox/raw/main/STAGE%201%20BOSS.mp3',
  },
]);

const currentIndex = ref(0);

const isPlaying = ref(false);
const volume = ref(1);

const currentTime = ref(0);
const duration = ref(0);

const createAudio = (url) => {
  const audio = new Audio(url);
  audio.volume = volume.value;

  audio.addEventListener('loadedmetadata', () => {
    duration.value = audio.duration;
  });

  audio.addEventListener('timeupdate', () => {
    currentTime.value = audio.currentTime;
  });

  audio.addEventListener('ended', () => {
    nextSong();
  });

  return audio;
};

let backgroundMusic = createAudio(playlist.value[currentIndex.value].url);

const onSeek = (e) => {
  backgroundMusic.currentTime = e.target.value;
};

const formatTime = (secs) => {
  if (isNaN(secs)) return '00:00';
  const minutes = Math.floor(secs / 60);
  const seconds = Math.floor(secs % 60);
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(
    2,
    '0'
  )}`;
};

const changeVolume = () => {
  backgroundMusic.volume = volume.value;
};

const playAudio = () => {
  if (backgroundMusic.paused) {
    backgroundMusic
      .play()
      .then(() => {
        isPlaying.value = true;
        console.log('Now Playing...');
      })
      .catch((error) => {
        console.error('Wait, It has some trouble!');
      });
  } else {
    backgroundMusic.pause();
    isPlaying.value = false;
    console.log('Pause!');
  }
};

const switchSong = (newIndex) => {
  backgroundMusic.pause();
  isPlaying.value = false;

  currentIndex.value = newIndex;

  backgroundMusic = createAudio(playlist.value[currentIndex.value].url);

  currentTime.value = 0;

  playAudio();
};

const prevSong = () => {
  let newIndex = currentIndex.value - 1;
  if (newIndex < 0) {
    newIndex = playlist.value.length - 1;
  }
  switchSong(newIndex);
};

const nextSong = () => {
  let newIndex = currentIndex.value + 1;
  if (newIndex >= playlist.value.length) {
    newIndex = 0;
  }
  switchSong(newIndex);
};
</script>

<style>
.box {
  width: 100vw;
  height: 100vh;
  background: linear-gradient(136deg, #111118 0%, #1a1a2e 100%);
  color: #00f0ff;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 24px;
  font-family: 'Courier New', Courier, monospace;
}

.box h1 {
  color: #00f0ff;
  font-size: 2.2rem;
  text-shadow: 0 0 10px rgba(0, 240, 255, 0.5);
  letter-spacing: 2px;
  margin-bottom: 10px;
}

.progress-container {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 80%;
  max-width: 450px;
  background: rgba(255, 255, 255, 0.05);
  padding: 12px 16px;
  border-radius: 12px;
  border: 1px solid rgba(0, 240, 255, 0.2);
}

input[type='range'] {
  -wepkit-appearance: none;
  width: 100%;
  height: 6px;
  background: #111;
  border-radius: 3px;
  outline: none;
  cursor: pointer;
  border: 1px solid rgba(0, 240, 255, 0.3);
}

input[type='range']::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #00f0ff;
  box-shadow: 0 0 8px #00f0ff, 0 0 20px rgba(0, 240, 255, 0.5);
  cursor: pointer;
  transition: transform 0.1s;
}

input[type='range']::-webkit-slider-thumb:hover {
  transform: scale(1.2);
}

button {
  background: transparent;
  color: #00f0ff;
  border: 1px solid #00f0ff;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  letter-spacing: 1px;
  transition: all 0.2s ease;
  box-shadow: 0 0 5px rgba(0, 240, 255, 0.1);
}

button:hover {
  background: #00f0ff;
  color: #111;
  box-shadow: 0 0 15px rgba(0, 240, 255, 0.6);
}

.playlist-controls {
  display: flex;
  gap: 15px;
}

.volume-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.03);
  padding: 8px 16px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.05);
  width: 60%;
  max-width: 250px;
}

.time {
  font-size: 12px;
  color: #00f0ff;
  opacity: 0.7;
  font-family: monospace;
}

.volume-container label {
  font-size: 0.9rem;
  color: #888;
}
</style>
