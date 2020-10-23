<template lang="pug">
  .audio-block
    .audio-player
      .control(
        @click="togglePlay"
      )
        button(:name="isPlaying ? 'pause' : 'play'" fill="inherit") >
      
      .progress-bar
        input(
          :class="{ 'slider-none': !isPlaying }"
          ref="progress"
          :style="`background: -webkit-linear-gradient(left, ${mainColor} 0%, ${mainColor} ${progress}%, #dddddd ${progress}%, #dddddd 100%);`" 
          min="0" 
          max="100" 
          v-model="progress"
          @input="setAudioTime"
          type="range"
        )
      .time(
        v-if="duration"
        :style="{color: mainColor}"
      ) {{ currentDuration }} / {{ duration }}
      .volume
        button(:name="volumeIcon" @click="rangeOpen = !rangeOpen" width="16" height="16") x
        input(
          :style="`background: -webkit-linear-gradient(left, ${mainColor} 0%, ${mainColor} ${rangeValue}%, #dddddd ${rangeValue}%, #dddddd 100%);`"
          min="0" 
          max="100" 
          v-model="rangeValue" 
          type="range" 
          v-show="rangeOpen"
        )
      audio(
        ref="audio"
        @timeupdate="updateProgress"
        @ended="isPlaying=false"
        @error="errorHandler"
        preload="metadata"
      )
        source(src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3")
      
    | {{ navigator }}
</template>

<script>
export default {
  data() {
    return {
      isPlaying: false,
      progress: 0,
      duration: 100,
      currentDuration: "00:00",
      rangeOpen: false,
      rangeValue: 100,
      navigator: "",
    };
  },
  computed: {
    mainColor() {
      return "#000000";
    },
    volumeIcon() {
      if (this.rangeValue > 70) {
        return "volume/max";
      } else if (this.rangeValue > 30) {
        return "volume/mid";
      } else if (this.rangeValue > 0) {
        return "volume/low";
      } else {
        return "volume/mute";
      }
    },
  },
  watch: {
    rangeValue(value) {
      this.$refs.audio.volume = value / 100;
    },
  },
  mounted() {
    this.$refs.audio.onloadedmetadata = () => {
      this.duration = this.getHumanDuration(
        Math.ceil(this.$refs.audio.duration)
      );
    };

    this.navigator =
      navigator.appCodeName +
      " |2 " +
      navigator.appName +
      " |3 " +
      navigator.platform +
      " |4 " +
      navigator.userAgent;

    console.log(navigator);
  },
  methods: {
    togglePlay() {
      if (!this.isPlaying) {
        this.$refs.audio.play();
      } else {
        this.$refs.audio.pause();
      }
      this.isPlaying = !this.isPlaying;
    },
    updateProgress() {
      // this.duration = this.getHumanDuration(
      //   Math.ceil(this.$refs.audio.duration)
      // );
      this.progress =
        (this.$refs.audio.currentTime / this.$refs.audio.duration) * 100;
      this.currentDuration = this.getHumanDuration(
        Math.ceil(this.$refs.audio.currentTime)
      );
    },
    setAudioTime() {
      this.$refs.audio.currentTime =
        (this.$refs.audio.duration * this.progress) / 100;
    },
    getHumanDuration(duration) {
      const toFormat = (value) => {
        return value <= 9 ? "0" + value : value;
      };

      const minutes = parseInt(duration / 60);
      const seconds = duration - minutes * 60;
      return `${toFormat(minutes)}:${toFormat(seconds)}`;
    },
    errorHandler() {
      alert("AudioBlock: Something wrong!");
    },
  },
};
</script>

<style lang="scss" scoped>
.audio-block {
  margin-top: 30px;

  .audio-title {
    margin-top: 20px;
    margin-bottom: 10px;
    font-size: 18px;
    line-height: 140%;
    color: #747474;

    &:first {
      margin-top: 30px;
    }
  }

  .audio-player {
    background: rgb(228, 111, 228);
    max-width: 500px;
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;

    .control {
      display: flex;
      justify-content: center;
      align-content: center;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      border-style: solid;
      border-width: 2px;
      border-color: var(--main-color);
      fill: var(--main-color);
      transition: 0.3s;
      cursor: pointer;
      user-select: none;

      &:hover {
        fill: black;
        border-color: black;
      }
      svg {
        width: 8px;
        height: 100%;
      }
    }

    .progress-bar {
      display: flex;
      align-items: center;
      flex-grow: 1;
      max-width: 300px;
      input {
        margin: 0 10px;
        width: 100%;
      }
    }

    .volume {
      position: relative;
      display: flex;
      margin-left: 15px;
      cursor: pointer;
      transition: 0.3s;
      fill: var(--main-color);

      svg:hover {
        fill: #000000;
      }
      input {
        position: absolute;
        transform: rotate(-90deg);
        width: 80px;
        left: -33px;
        top: -47px;
      }
    }
  }
}

input[type="range"] {
  background: #dddddd;
  border-radius: 5px;
  height: 4px;
  -webkit-appearance: none;
  outline: none;

  &:focus,
  &:active {
    outline: none;
  }

  &::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 14px;
    height: 14px;
    background: #ffffff;
    border: 2px solid var(--main-color);
    border-radius: 14px;
    cursor: pointer;
  }

  &::-moz-range-thumb {
    width: 14px;
    height: 14px;
    background: #ffffff;
    border: 2px solid var(--main-color);
    border-radius: 14px;
    cursor: pointer;
  }

  &.slider-none {
    &::-webkit-slider-thumb,
    &::-moz-range-thumb {
      display: none;
    }
  }
}
</style>
